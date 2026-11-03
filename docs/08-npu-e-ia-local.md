# Orange Pi 5 Max: NPU e IA local

Coleta feita em 03/11/2026 e revisada na mesma data, com o manual oficial v1.3 da placa, os repositórios da Rockchip/Airockchip e o histórico do kernel. Este documento foi escrito para ser lido sozinho. Números de desempenho vêm de fontes com datas e versões diferentes, então confira a data e a versão de cada medição antes de usar.

## Legenda de marcação

- **[oficial]**: documentação ou repositório do fabricante do chip (Rockchip/Airockchip) ou documentação de kernel.
- **[oficial, manual v1.3, p. X]**: manual do usuário da própria Orange Pi 5 Max, versão 1.3, citado pela página impressa.
- **[comunidade/Max]**: medição ou relato feito numa Orange Pi 5 Max por terceiros.
- **[placa irmã: qual]**: medição feita em outra placa com RK3588 (ou RK3588S). Como a NPU é do chip, serve de referência, mas não é garantia para a Max.
- **[não confirmado]**: afirmação que não consegui verificar em fonte que li. A nota diz onde procurei.

Regra geral: a NPU é do SoC RK3588, então a parte de hardware e software de NPU vale para qualquer placa com esse chip. Já o desempenho real depende de RAM, refrigeração, versão do runtime e do modelo. Nada aqui foi testado por mim; tudo vem das fontes listadas no fim.

---

## 1. Resumo rápido

- A Orange Pi 5 Max usa o RK3588: 4 núcleos Cortex-A76 e 4 Cortex-A55, GPU Mali-G610 e NPU de **até 6 TOPS** [oficial, manual v1.3, p. 11]. A memória é LPDDR5, de 4, 8 ou 16 GB [oficial, manual v1.3, p. 11-12].
- Para LLMs, o caminho oficial é o **RKLLM** (repositório `airockchip/rknn-llm`, versão mais recente **v1.3.1**). O SDK dessa versão exige driver NPU **v0.9.8** (seção 5.1). Para visão e modelos genéricos, o caminho é o **RKNN-Toolkit2** (`airockchip/rknn-toolkit2`, versão mais recente **v2.5.0**) com o runtime de placa `rknn-toolkit-lite2` [oficial].
- A conversão de modelos roda num PC Ubuntu x86-64, não na placa [oficial, manual v1.3, p. 360 e 385].
- Medições de LLM numa Max: o manual oficial traz uma tabela de desempenho RKLLM feita na própria placa (seção 4), com a ressalva de que o texto do PDF embaralha as colunas. A review independente mediu TinyLlama 1.1B quantizado em cerca de **20,2 tokens/s** em três rodadas [comunidade/Max]. A mesma família de fontes diz que a faixa esperada da Rockchip é 10–15 tokens/s, mas esse número não foi medido pelo autor [comunidade/Max]. Isso é uma divergência, detalhada na seção 9.
- Rodar Ollama ou llama.cpp usa a **CPU** (A76), não a NPU. Não achei medição de Ollama nem de llama.cpp numa Max, nem de Ollama numa RK3588 de 16 GB. Em outra placa RK3588 (Turing Pi RK1, 32 GB), o llama.cpp fez cerca de 22,6 tokens/s num modelo 1.5B Q4_K_M, com só 4 threads [placa irmã: Turing Pi RK1].
- Para visão, o PhotonVision mediu YOLO no RKNN numa Orange Pi 5 com RK3588: cerca de 15 a 23 ms por inferência, conforme o modelo (YOLOv5 a YOLOv11, int8) [placa irmã: Orange Pi 5, variante não informada]. A documentação do Frigate dá 20–40 ms para yolov8n numa Orange Pi 5 Plus [placa irmã]. A tabela oficial do rknn_model_zoo traz FPS de YOLO no RK3588 com um núcleo da NPU (yolov8n INT8 a 76,3 FPS; yolo11n a 64,0 FPS; yolov5s a 49,1 FPS), sem identificar a placa [oficial, seção 7]. Não achei medição de YOLO numa Max.
- Com 16 GB, a Max cabe confortavelmente modelos de 1B a 4B no NPU, e até 6B nos números da tabela oficial da Rockchip. Modelos de 7B e 8B não aparecem em nenhuma medição de NPU que eu tenha lido para RK3588 [não confirmado: procurei em README, changelog, tabela de benchmark e buscas].

---

## 2. Como a NPU funciona

**Arquitetura [oficial, via docs do Frigate e do Rocket, placa irmã/chip]:** a NPU do RK3588 tem **3 núcleos** independentes. O driver pode mandar trabalho para um, dois ou os três. Na documentação do Frigate, a recomendação é usar os 3 núcleos em RK3588 para ganhar desempenho (a configuração padrão é automática). O manual da Max mostra a carga por núcleo com o comando `sudo cat /sys/kernel/debug/rknpu/load` durante a inferência [oficial, manual v1.3, p. 421].

**Clock e consumo [comunidade/Max]:** na review da Max, a NPU rodou a 1 GHz, com faixa dinâmica de 300 MHz a 1 GHz. Com a carga de LLM, o autor relata 100% de ocupação em 1 GHz.

**Quantização [oficial]:** o RKNN-Toolkit2 converte modelos para formatos que o NPU executa, com quantização inteira e ponto flutuante. Para LLM, o README e o changelog do rknn-llm usam esquemas com nomes como:
- **w8a8**: pesos e ativações em 8 bits. É o único esquema da tabela de benchmark do RK3588 e do RK3562.
- **w4a16**: pesos em 4 bits e ativações em 16 bits. Aparece nas tabelas do RK3576 e do RV1126B.
- **w4a16_g128**: variante de w4a16 com grupos de 128. Aparece nas tabelas do RK3576 e do RV1126B.
- **fp16**: aparece nos encoders de visão e áudio dos modelos multimodais, não como quantização de LLM.

O changelog do rknn-llm confirma que o suporte a quantização por grupos existe desde a v1.1.0 (tamanhos de grupo de 32, 64 e 128 para w4a16). A afirmação de que "o RKLLM só suporta W8A8 neste chip" [comunidade/Max] não é confirmada: a tabela do RK3588 só traz w8a8, mas nenhuma fonte oficial diz que w4a16 não funciona no RK3588. Mantido como [não confirmado].

**Formatos [oficial]:** o modelo convertido vira um arquivo `.rknn` para visão e um arquivo `.rkllm` para LLM. No exemplo da review do TinyLlama, o `.rkllm` ficou com cerca de 1,14 GB (1142,9 MB) [comunidade/Max].

---

## 3. RKNN-Toolkit2 e rknn-toolkit-lite2

**O que é [oficial]:** o RKNN-Toolkit2 é um SDK para converter modelos, rodar inferência e medir desempenho em PC e em plataformas Rockchip. O repositório é `github.com/airockchip/rknn-toolkit2`.

**Versão [oficial]:** o README e a aba de releases apontam a **v2.5.0** como a versão marcada como "Latest". A release v2.3.2 aparece como anterior.

**Plataformas [oficial]:** RK3588, RK3576, RK3566/RK3568, RK3562, RV1103/RV1106, RV1103B/RV1106B, RV1126B e RK2118. RK1808, RV1109, RV1126 e RK3399Pro ficam no repositório antigo `rknn-toolkit`, e as duas versões **não são compatíveis**.

**Python [oficial]:** o README lista Python 3.6 a 3.12 para o toolkit. Para o RKLLM, a lista é outra (ver seção 4).

**Runtime na placa [oficial]:** o `rknn-toolkit-lite2` oferece interface Python para rodar os modelos já convertidos na placa. Existe também um runtime C/C++ separado. O manual da Max descreve o `rknn_server` e a `librknnrt.so` na placa, e o uso do NPU da placa a partir do PC via adb, mas com exemplos da versão 1.5.2 do toolkit, mais antiga que a atual [oficial, manual v1.3, p. 360-361]. A review usou `rknn-toolkit-lite2` **2.3.2** e `rknn runtime` **1.6.0** para visão [comunidade/Max].

**Operadores [oficial, changelog v2.5.0]:** o changelog cita que GRU roda em NPU/GPU (com multi-batch), GridSample roda na GPU, e ReduceLogSumExp, Pow e TopK rodam na CPU. O README não traz uma lista completa de operadores sem suporte. Se o seu modelo usar um operador fora dessa lista, a conversão pode falhar ou cair na CPU. Não achei a lista completa no README, no changelog nem no manual [não confirmado].

**Onde converter [oficial, manual v1.3, p. 360 e 385]:** o manual manda instalar o RKNN-Toolkit2 num PC Ubuntu, e o exemplo do RKLLM usa Ubuntu 22.04 x64. A conversão não roda na própria placa.

---

## 4. RKLLM (LLMs na NPU)

**Repositório [oficial]:** `github.com/airockchip/rknn-llm`. A versão mais recente no README e no changelog é **v1.3.1**. O changelog dessa versão cita modelos novos (LFM2.5-VL-450M, Spark-X2.5), entrada de áudio multimodal com Gemma4 e correções de runtime.

**Plataformas [oficial]:** RK3588, RK3576, RK3562 e RV1126B.

**Python [oficial]:** 3.10, 3.11 e 3.12. Para Python 3.12 o README pede `export BUILD_CUDA_EXT=0` antes da instalação. A conversão de RWKV exige Python 3.12.

**Modelos listados no README [oficial]:** LLaMA e TinyLLaMA; Qwen2, Qwen2.5, Qwen3 e Qwen3.5; Phi-2 e Phi-3; ChatGLM3-6B; Gemma2, Gemma3, Gemma3n e Gemma4; InternLM2; MiniCPM3 e MiniCPM4; TeleChat2; DeepSeek-R1-Distill; RWKV7; e modelos multimodais como Qwen2-VL, Qwen3-VL, MiniCPM-V, InternVL, SmolVLM, Janus-Pro e DeepSeekOCR. A lista é do README e não traz notas de compatibilidade por modelo.

**Modelos que já rodaram numa Max [oficial, manual v1.3, p. 423-424; comunidade/Max]:** o manual da Max traz uma tabela própria de desempenho RKLLM com TinyLlama, Qwen2, Phi-3, ChatGLM3, Gemma 2B, InternLM2 e MiniCPM. O texto extraído do PDF embaralha as colunas, então só atribuí valores quando a linha estava clara (ver abaixo). A review independente mediu o TinyLlama 1.1B.

**Tabela oficial de benchmark, RK3588 [oficial, com lacuna]:** o arquivo `benchmark.md` do rknn-llm traz TTFT (tempo até o primeiro token), tokens/s e memória, com entrada de 128 tokens e geração de 64 tokens. Todas as linhas do RK3588 são w8a8. O arquivo diz que os dados foram coletados nas frequências máximas de CPU e NPU de cada plataforma, mas não diz qual placa RK3588 foi usada, nem a versão do runtime RKLLM nem a do driver. Valores:

| Modelo | Tokens/s | TTFT (ms) | Memória (MB) |
|---|---|---|---|
| Qwen2 0.5B | 41,58 | 145,90 | 669,56 |
| MiniCPM4 0.5B | 45,34 | 135,29 | 534,82 |
| Qwen3 0.6B | 32,91 | 199,09 | 791,11 |
| TinyLLaMA 1.1B | 24,43 | 243,93 | 1093,66 |
| RWKV7 1.5B | 13,82 | 634,34 | 1486,19 |
| Qwen2.5 1.5B | 16,69 | 378,31 | 1689,21 |
| InternLM2 1.8B | 15,45 | 380,14 | 1775,56 |
| Gemma2 2B | 10,37 | 598,41 | 2779,22 |
| Gemma3n 2B | 9,91 | 1124,51 | 2702,59 |
| TeleChat2 3B | 10,18 | 618,48 | 2788,39 |
| Phi3 3.8B | 7,45 | 1017,28 | 3758,34 |
| MiniCPM3 4B | 5,94 | 1408,49 | 4384,02 |
| ChatGLM3 6B | 4,98 | 1352,94 | 5985,99 |
| Qwen3.5 0.8B | 27,05 | 587,74 | 1039,66 |
| Qwen3.5 2B | 13,59 | 776,95 | 2121,91 |
| Qwen3.5 4B | 6,22 | 2068,88 | 4575,78 |
| Qwen3-VL 2B | 14,98 | 383,62 | 1868,98 |
| DeepSeekOCR 3B (A570M) | 31,60 | 701,21 | 3066,66 |
| Gemma4 E2B | 11,12 | 599,06 | 2498,70 |
| LFM2.5-VL 450M | 62,16 | 102,74 | 436,94 |
| Spark-X2.5 1.7B | 15,61 | 386,23 | 1830,32 |

Os números vêm da própria Rockchip, mas a placa não é identificada. Por isso marco como [oficial] com a ressalva de que a placa e a versão não são informadas. Para RK3576 e RV1126B a tabela tem w4a16 e w4a16_g128; para RK3562 só w8a8.

**Como a memória entra [não confirmado]:** a tabela reporta memória por modelo, mas nenhuma fonte que li diz se a NPU usa a RAM do sistema. Procurei no README, no changelog e no manual; não encontrei a arquitetura de memória documentada.

**Medido numa Max, manual oficial [oficial, manual v1.3, p. 423-424]:** o manual descreve o método: prompt de cerca de 256 tokens gerado por GPT, medição da velocidade de geração ("reasoning") e de preenchimento ("prefill"), e leitura da carga da NPU. O próprio manual avisa que a contagem de tokens varia entre modelos, então o resultado tem erro. Os valores que consegui atribuir com clareza no PDF são:
- Phi-3 3,8B (W8A8): **17,45 tokens/s** de geração, com memória de 4,29 GB e carga da NPU de 3×62%.
- Gemma 2B (W8A8): **4,73 tokens/s** de geração, com memória de 3,90 GB.

Os demais valores da tabela (TinyLlama, Qwen, ChatGLM3, InternLM2, MiniCPM) estão embaralhados no texto extraído: a linha de geração e a de preenchimento não se separam de forma confiável. Não atribuí nenhum valor a eles.

**Medido numa Max, review [comunidade/Max]:** TinyLlama 1.1B quantizado, RKLLM, 3 rodadas: **20,27**, **20,04** e **20,40** tokens/s (média de cerca de 20,2). Tempos de 1617 a 1646 ms por rodada. A review não informa a versão do RKLLM nem a contagem exata de tokens por rodada.

**Divergência para o TinyLlama e outros modelos:** o mesmo modelo aparece com 15 tokens/s (busca: release de 2024 da CNX Software, placa não nomeada), 20,2 tokens/s (review da Max) e 24,4 tokens/s (tabela oficial RK3588). Phi-3 e Gemma 2B também divergem entre a tabela da Rockchip e a do manual da Max (seção 9). Os números vêm de versões de runtime e de medição diferentes, então não dá para dizer qual é o "certo".

**Conversão [comunidade/Max]:** o TinyLlama levou 79,5 s para converter no host (carga 0,4 s, build 22,7 s, export 56,4 s), num AMD Ryzen AI Max+ 395 com Ubuntu 24.04. Isso é o host de conversão, não a Max.

---

## 5. Driver rknpu (kernel BSP) e driver "rocket" (mainline)

Hoje há dois caminhos de driver NPU no Linux para RK3588. Eles não são intercambiáveis.

### 5.1 rknpu (vendor, kernel BSP)

- **Origem:** é o driver mantido pela Rockchip, que vive no kernel de fornecedor (BSP). Segundo o README do rknn-llm, o driver kernel "é de código aberto e está no código do kernel Rockchip" [oficial], mas o README não dá versão.
- **Versão exigida pelo manual [oficial, manual v1.3, p. 385-386]:** o capítulo de RKLLM do manual manda atualizar o driver NPU do kernel para a **v0.9.6** e traz o pacote `rknpu_driver_0.9.6_20240322`. Isso vale para a versão de RKLLM daquele capítulo, que o manual não nomeia. O RKLLM 1.3.1 pede outra versão, ver a seguir.
- **Versão exigida pelo RKLLM 1.3.1 [oficial, SDK RKLLM 1.3.1, seção 2.4]:** o SDK manda confirmar que o driver na placa é a **v0.9.8** antes de rodar o runtime. Comando da própria Rockchip: `cat /sys/kernel/debug/rknpu/version`, com saída `RKNPU driver: v0.9.8`. Se a versão for menor, o SDK manda baixar o firmware oficial mais novo. Com firmware não oficial, o pacote `rknpu_driver_0.9.8_20241009.tar.bz2` é copiado para `kernel/drivers/rknpu` e o kernel é recompilado. O SDK cita os kernels 5.10 (recomendado 5.10.209) e 6.1 (exemplo 6.1.84). O mesmo pacote está em `rknpu-driver/` no repositório `airockchip/rknn-llm`.
- **Versão na Max [comunidade/Max]:** o relato da review aponta **RKNPU driver 0.9.8** (com RKLLM 1.2.2 e rknn runtime 1.6.0). Um relato de outra placa (Orange Pi 5 Ultra, primo da Max, com imagem vendor) também mostra `rknpu 0.9.8`. O log de um kernel vendor 6.1.115 também mostra `rknpu 0.9.8 20240828` (busca, placa não identificada). Um log de RKLLM 1.1.4 numa Rock 5B+ (RK3588) mostra `rknpu driver version: 0.9.8` [placa irmã: Rock 5B+, issue #154 do rknn-llm].
- **Kernel na Max [comunidade/Max]:** Armbian 25.11, base Debian 12, kernel `6.1.115-vendor-rk35xx`.
- **Quem usa:** o RKLLM e o RKNN runtime dependem desse driver para rodar modelos.

### 5.2 rocket (mainline, subsistema accel)

- **O que é [oficial]:** o driver `accel/rocket` está na documentação do kernel (docs.kernel.org). Ele suporta as NPUs de alguns SoCs Rockchip, citando o RK3588 como exemplo e o capítulo 36 do TRM da Rockchip como fonte do hardware. O código fica em `drivers/accel/rocket` e tem a opção `CONFIG_DRM_ACCEL_ROCKET`, que depende de ARM64 e de IOMMU da Rockchip.
- **Escopo [oficial]:** a parte de kernel é fina. Ele liga e desliga a energia, aloca e mapeia buffers e envia jobs ao frontend do NPU. O resto fica para o userspace.
- **Userspace [oficial, parte]:** o lado de usuário é um driver Gallium chamado "rocket", que a documentação do kernel descreve como parte do projeto Mesa3D. A ligação com o framework Teflon e a versão do Mesa não confirmei: a documentação do kernel não cita Teflon, e o único texto que encontrei sobre o Mesa 25.3 foi um título de busca, sem o artigo aberto [não confirmado].
- **Autoria [não confirmado]:** a documentação do kernel e o Kconfig não citam nome de autor. A atribuição a uma pessoa específica que eu tinha antes não aparece nas fontes que abri, então foi retirada.
- **Data de entrada no mainline [oficial, repositório]:** a árvore do Linux na tag **v6.18** contém `drivers/accel/rocket`, com Kconfig, código de dispositivo, de jobs e de buffers. A tag **v6.17** não tem essa pasta (a página deu 404). A base de configuração do kernel (lkddb) também lista `CONFIG_DRM_ACCEL_ROCKET` a partir de 6.18. Logo, o driver entrou no **Linux 6.18**. O relato do Armbian sobre 6.17 está errado. A página da Phoronix sobre o envio não pôde ser lida: o Web Archive não abriu pela ferramenta, e o site dá erro 403.
- **Status em stable [não confirmado]:** uma referência de configuração diz que o driver existe desde a 6.12.57, o que sugeriria backport em stable. Não verifiquei no changelog de stable.

### 5.3 O que muda entre eles

| Aspecto | rknpu (vendor) | rocket (mainline) |
|---|---|---|
| Origem | Rockchip, kernel BSP | comunidade, kernel mainline (accel) |
| Kernel típico | 6.1 vendor (Armbian/Radxa/Orange Pi) | 6.18 ou superior |
| Versão de driver citada | 0.9.8 (SDK RKLLM 1.3.1) e 0.9.6 (manual, RKLLM antigo) | não se aplica (driver do mainline) |
| Userspace que usa | RKNN e RKLLM (runtime da Rockchip) | Mesa (Gallium "rocket") |
| Suporte a LLM via RKLLM | sim [oficial] | não encontrei ligação com RKLLM [não confirmado] |
| Desempenho comparado | números oficiais na seção 4 | early test (2024) diz "comparável" em MobileNetV1; fonte não aberta [não confirmado] |

Leitura prática: para rodar RKLLM hoje, o caminho é o driver vendor. O rocket é a rota para quem quer NPU com kernel mainline e usa Mesa. Não achei medição de LLM com o rocket.

---

## 6. Ollama e llama.cpp na CPU (núcleos A76)

**Como funciona [oficial/comunidade]:** o Ollama e o llama.cpp padrão rodam no CPU do RK3588. Eles não usam o NPU. O CPU da Max tem 4 núcleos Cortex-A76 (até 2,256 GHz) e 4 Cortex-A55 (1,8 GHz) [comunidade/Max].

**Medição na Max [comunidade/Max]:** não achei nenhuma medição de Ollama nem de llama.cpp numa Orange Pi 5 Max. A review só traz sysbench (13.688 eventos/s, em 10 s) e não testa LLM por CPU. A busca por "llama.cpp RK3588 Orange Pi 5 Max" também não achou resultado da Max. Nas issues do repositório llama.cpp, a busca por "orange pi 5 max" e "RK3588" levou à issue #722 ("Rockchip RK3588 perf", fechada), com relatos de 2023. Não são da Max: um modelo 7B Q5_1 a cerca de 1 token/s, com 7 threads, em estimativa sem benchmark formal [placa irmã: Orange Pi 5 de 16 GB], e cerca de 500 ms por token com 4 threads, sem placa informada no comentário [placa irmã: não informada]. Não trazem versão de build, então servem só como ordem de grandeza. Fonte: https://github.com/ggml-org/llama.cpp/issues/722

**Medição em placa irmã [placa irmã: Turing Pi RK1, RK3588, 32 GB, Ubuntu 22.04 ARM64]:** llama.cpp (build 10134, commit `42fc24306`), 4 threads (`-t 4`), prompt de 1024 tokens e geração de 256 tokens, 5 repetições. Os valores de geração em tokens/s:

| Modelo | Quantização | Tokens/s (geração) | Tempo de prompt de 1024 tokens (s) |
|---|---|---|---|
| Qwen2.5-1.5B-Instruct | Q4_K_M | 22,57 | 18,77 |
| Qwen2.5-1.5B-Instruct | Q8_0 | 15,26 | 12,39 |
| DeepSeek-R1-Distill-Qwen-1.5B | Q4_K_M | 22,78 | 18,80 |
| Llama-3.2-3B-Instruct | Q4_K_M | 11,25 | 40,09 |
| Llama-3.2-3B-Instruct | Q8_0 | 7,36 | 26,96 |
| Qwen2.5-7B-Instruct | Q4_K_M | 5,44 | 89,98 |
| Qwen2.5-7B-Instruct | Q8_0 | 3,43 | 68,09 |

Observações: o artigo diz que a medição é inferência GGUF no CPU do RK3588, e que a inferência com NPU é tratada em outro artigo. Só 4 threads foram usadas, então o resultado não é o teto do chip. O tempo de prompt cresce bastante com o tamanho do modelo, e quantizações maiores caem em velocidade de geração. Os valores são sintéticos (pp1024/tg256) e não representam chat real.

**Outra fonte de CPU, fraca [comunidade, placa "Orange Pi 5" sem variante]:** um post de visão geral cita 11,3 tokens/s para TinyLlama com llama.cpp em CPU, e 6–7 tokens/s para Phi-3 Mini. Não diz variante da placa, quantização, threads nem versão do software, então não usei como medição.

**Ollama [não confirmado]:** procurei por "Ollama RK3588 tokens/s" e por Orange Pi com Llama 3.2 3B. Não achei número de Ollama em nenhuma RK3588, nem nas buscas gerais por Orange Pi 5 Max, nem no artigo da Turing Pi (que cita Ollama só na instalação). O Ollama empacota o llama.cpp, então o número do RK1 serve só como ordem de grandeza. Nas issues do ollama/ollama, a busca por "orange pi 5" e "rk3588" achou pedidos de suporte ao NPU da RK3588, como a issue #5001 (de 12/06/2024, que só pede o suporte), e nenhuma medição de tokens/s na placa. Fonte: https://github.com/ollama/ollama/issues/5001

**Consequência para a Max:** com 16 GB, o CPU consegue modelos de 7B em Q4 a cerca de 5 tokens/s (medição de placa irmã), o que é lento para uso interativo. Para 1B a 3B, o CPU chega a 11 a 23 tokens/s (placa irmã), faixa parecida com a do NPU para TinyLlama. A Turing Pi também testou 7B em Q6_K e Q8_0 na RK1 de 32 GB, com 3,72 e 3,43 tokens/s. Ela afirma que a versão de 16 GB roda o 7B em Q6_K com folga, mas isso é inferência pelo pico de memória (8,68 GiB em Q4_K_M; 14,78 GiB em Q8_0), não medição numa placa de 16 GB [placa irmã: Turing Pi RK1].

---

## 7. Visão computacional (YOLO no RKNN, Frigate)

**Suporte no Frigate [fonte da busca, docs espelhados em git]:** o detector RKNN funciona em RK3588/RK3588S, RK3568, RK3566 e RK3562. O Frigate não traz modelos YOLO prontos; é preciso fornecer o modelo convertido.

**Latências do PhotonVision [placa irmã: Orange Pi 5 com RK3588, variante não informada]:** RKNN Toolkit 2, int8, COCO 2017, tempo total por inferência (CPU e NPU somados): YOLOv5 15,0 ms; YOLOv8 17,3 ms; YOLOv5u 20,3 ms; YOLOv11 23,0 ms. O desvio-padrão fica entre 1,7 e 2,4 ms. A página não traz data de medição, e a quantização usou 20 imagens de calibração.

**Latências [placa irmã: Orange Pi 5 Plus, RK3588]:**
- yolov8n: **20–25 ms** numa revisão mais nova da documentação, e **25–40 ms** numa revisão mais antiga. A divergência é registrada; use como ordem de grandeza.
- yolo-nas s (3 núcleos): cerca de **25–30 ms**, segundo a documentação do Frigate.

**Tabela oficial do rknn_model_zoo [oficial, placa não identificada]:** o README do `airockchip/rknn_model_zoo` traz FPS de YOLO em INT8, entrada 640×640, na coluna "RK3588 @single_core" (um núcleo da NPU). A tabela não diz data, versão de runtime nem se o tempo inclui pré e pós-processamento. A coluna de milissegundos é conversão minha (1000 ÷ FPS), não medição:

| Modelo | FPS (um núcleo) | ms por inferência (conversão) |
|---|---|---|
| yolov5s | 49,1 | 20,4 |
| yolov5n | 84,6 | 11,8 |
| yolov8n | 76,3 | 13,1 |
| yolov8s | 38,6 | 25,9 |
| yolo11n | 64,0 | 15,6 |
| yolo11s | 34,6 | 28,9 |
| yolov10n | 63,9 | 15,6 |
| yolo26n | 55,0 | 18,2 |

Os valores ficam perto dos do PhotonVision, mas as duas fontes não dizem se medem o mesmo intervalo de tempo. Fonte: https://raw.githubusercontent.com/airockchip/rknn_model_zoo/main/README.md

Jeff Geerling não tem entrada de Orange Pi nem de RK3588 no README do ai-benchmarks, lido em 03/11/2026 (https://github.com/geerlingguy/ai-benchmarks). Na issue da Max no sbc-reviews (https://github.com/geerlingguy/sbc-reviews/issues/49) não há número de LLM nem de NPU. O único comentário sobre a NPU, de 09/05/2025, pede esse teste.

**Na Max:** não achei latência de YOLO medida na própria Max. O autor da review mediu ResNet18 INT8 (não YOLO) em **4,09 ms** de média (mínimo 4,02, máximo 4,43; 10 rodadas após 2 de aquecimento), o que dá cerca de **244 FPS** [comunidade/Max]. Esse número é de ResNet18, então não vale como estimativa de YOLO.

**Conversão de visão [comunidade/Max]:** ResNet18 converteu em 1,37 s, com saída de 11,4 MB, no mesmo host x86.

**Ressalva:** os números de Frigate vêm de commits de docs espelhados em um git público, não do site oficial. Tratar como referência de ordem de grandeza.

---

## 8. O que a RAM de 16 GB permite

Fatos:
- A Max vem com 4, 8 ou 16 GB de LPDDR5 [oficial, manual v1.3, p. 11-12]. A review foi feita na variante de 16 GB, e o sistema reportou **14,4 GB** disponíveis depois do kernel e da GPU [comunidade/Max].
- A tabela oficial do RK3588 mostra memória de **670 MB** (Qwen2 0,5B) a **5,99 GB** (ChatGLM3 6B) em w8a8.
- Modelos da tabela de 4B (Phi3 3,8B, MiniCPM3 4B, Qwen3.5 4B) ocupam 3,8 a 4,6 GB.

Leitura (não confirmada):
- Com 16 GB, cabem modelos de até 6B em w8a8 no NPU, com folga para o sistema e o Frigate.
- Modelos de 7B e 8B não aparecem em nenhuma tabela de NPU que li para RK3588. Pelo tamanho, provavelmente cabem em RAM, mas não tenho medição de desempenho.
- Quem quer rodar o NPU e um servidor com Docker ao mesmo tempo deve somar a memória do modelo com a do sistema. A review mostra o sistema com Docker rodando [comunidade/Max].

Limitação de armazenamento [comunidade/Max]: o modelo de TinyLlama convertido ocupa 1,14 GB, e a review usou 1 TB de NVMe em `/opt` e microSD de 64 GB para o sistema.

---

## 9. Divergências entre fontes

1. **Tokens/s do TinyLlama na Max:** a review mediu cerca de 20,2 tokens/s em 20/09/2025 [comunidade/Max]. O artigo de benchmark publicado no mesmo site diz que 10–15 tokens/s é a faixa "esperada" pela Rockchip para a Max e que não mediu esse valor [comunidade/Max]. O manual oficial da Max não traz o TinyLlama de forma legível no PDF (ver seção 4). Não é possível dizer qual é o número real sem uma medição nova com versões declaradas.
2. **TinyLlama no RK3588:** 15 tokens/s (CNX Software, 2024, placa não nomeada), 20,2 (Max) e 24,4 (tabela oficial RK3588, placa não nomeada). Os três usam versões diferentes.
3. **Phi-3 3,8B e Gemma 2B:** o manual da Max dá 17,45 e 4,73 tokens/s de geração [oficial, manual v1.3, p. 424]. A tabela da Rockchip dá 7,45 (Phi3 3.8B) e 10,37 (Gemma2 2B) [oficial]. As direções são opostas nos dois casos, e as versões de RKLLM e de driver não são informadas em nenhuma das duas. Não dá para escolher um valor.
4. **Memória do TinyLlama:** o manual da Max mostra 1,376 GB na linha de TinyLlama, e a tabela da Rockchip mostra 1093,66 MB. Diferença provável de versão de runtime, não confirmada.
5. **Versão do rknn-toolkit2:** resolvida. As releases do GitHub marcam a v2.5.0 como "Latest" e a v2.3.2 como anterior.
6. **RKLLM só W8A8?:** resolvido em parte. A tabela de benchmark do RK3576 e do RV1126B tem w4a16 e w4a16_g128, e a do RK3562 só w8a8. A tabela do RK3588 só tem w8a8, mas isso não prova que o chip não aceite outros esquemas. Continua [não confirmado] para o RK3588.
7. **LPDDR4X ou LPDDR5 na Max:** resolvido. O manual oficial diz LPDDR5 em 4, 8 e 16 GB [oficial, manual v1.3, p. 11-12]. A review que disse LPDDR4X-2133 está errada nesse ponto.
8. **Driver rocket no mainline:** resolvido. O 6.18 é a versão em que a pasta existe e a 6.17 não tem a pasta (seção 5.2). O relato do Armbian sobre 6.17 não bate com o repositório.
9. **Versão do driver rknpu:** resolvida para o RKLLM 1.3.1, que exige **v0.9.8** [oficial, SDK RKLLM 1.3.1, seção 2.4]. O manual pede 0.9.6 para o capítulo de RKLLM dele, e a review usou RKLLM 1.2.2 com driver 0.9.8. A versão de RKLLM do manual não é nomeada, então a diferença não prova conflito entre versões.
10. **YOLO no RK3588:** o PhotonVision dá 15 a 23 ms (placa irmã) e a tabela do rknn_model_zoo dá 11,8 a 28,9 ms (conversão de FPS, um núcleo, sem placa identificada). As fontes não dizem se medem o mesmo intervalo, e nenhuma é da Max. Use como faixa, não como valor.

---

## 10. Limitações reais

- **Conversão de modelo só em PC.** O manual manda converter num PC Ubuntu x86-64 [oficial, manual v1.3, p. 360 e 385]. Na prática, a conversão não roda na Max. Para o runtime na placa, use `rknn-toolkit-lite2` ou o runtime RKLLM.
- **Versão casada entre runtime e driver.** O RKLLM 1.3.1 exige driver v0.9.8 [oficial, SDK]. Antes de atualizar o runtime ou o kernel, confira a versão com `cat /sys/kernel/debug/rknpu/version` e trave uma combinação que já funcionou.
- **Operadores não suportados.** O changelog do v2.5.0 lista operadores que vão para a CPU (ReduceLogSumExp, Pow, TopK) e para a GPU (GridSample). Não achei lista completa de operadores sem suporte. Modelos com operadores novos podem cair em CPU ou falhar na conversão.
- **Kernel vendor.** A própria review aponta dependência do kernel vendor 6.1, com pouca compatibilidade com mainline. Parte do software não tem binário ARM64 e precisa ser compilado.
- **Libomp.** O README do rknn-llm avisa que em algumas plataformas pode faltar `libomp.so`. A solução proposta é copiá-la do toolchain de cross-compilação para o diretório de bibliotecas da placa, ao lado de `librkllmrt.so`.
- **Refrigeração.** Na review, o SoC chegou a 66–67 °C sob carga moderada com refrigeração ativa; o autor avisa que passivo pode não bastar para cargas longas. O consumo ficou entre 15 e 25 W sob carga.
- **Medição de energia.** A review não mediu potência diretamente nas fontes de NPU; a estimativa de 5–6 W que aparece no artigo de benchmark não é medida, segundo o próprio texto.

---

## 11. O que não consegui confirmar

- Número de tokens/s de **qualquer LLM medido numa Max** além do TinyLlama 1.1B (review) e de Phi-3 3,8B e Gemma 2B (manual, p. 424). Para os demais modelos da tabela do manual, o PDF não permite atribuir os valores.
- Medição de **Ollama** em RK3588 (nem na Max). Procurei em buscas gerais, em Orange Pi 5 Max, no artigo da Turing Pi e nas issues do ollama/ollama, sem resultado (as issues só pedem suporte ao NPU).
- Medição de **llama.cpp** numa Max (só placas irmãs: RK1 com 4 threads, e a issue #722 de 2023, com Orange Pi 5 de 16 GB ou placa não informada).
- Latência de **YOLO** na própria Max (só placa irmã: Orange Pi 5 no PhotonVision e Orange Pi 5 Plus no Frigate, com revisões de docs que divergem; a tabela do rknn_model_zoo não identifica a placa).
- Lista completa de **operadores sem suporte** no RKNN-Toolkit2. Procurei no README e no changelog do v2.5.0.
- Se o **rocket** roda modelos RKLLM ou RKNN (não encontrei ligação na documentação do kernel nem no Mesa).
- Texto do artigo da Phoronix sobre o **rocket** e a versão do Mesa (site com 403; Web Archive não abriu pela ferramenta).
- Arquitetura de memória da NPU (se usa a RAM do sistema).
- Se o **rocket** entrou também em stable antes de 6.18 (backport da 6.12.57 não verificado).

---

## Fontes

Fontes abertas diretamente nesta revisão (03/11/2026):

- Manual do usuário Orange Pi 5 Max, versão 1.3 (PDF oficial da fabricante; citado por página impressa: p. 11-12, 360-361, 385-386, 421-424).
- https://github.com/airockchip/rknn-llm (README: versão v1.3.1, plataformas, modelos, Python e nota de libomp)
- https://github.com/airockchip/rknn-llm/blob/main/CHANGELOG.md (changelog v1.0.0 a v1.3.1: quantização, RK3588, sem menção a driver)
- https://github.com/airockchip/rknn-llm/blob/main/benchmark.md (tabelas de benchmark RK3588, RK3576, RK3562 e RV1126B)
- https://github.com/airockchip/rknn-toolkit2 (README: plataformas, Python, changelog com operadores, versão v2.5.0)
- https://github.com/airockchip/rknn-toolkit2/releases (v2.5.0 marcada como "Latest")
- https://raw.githubusercontent.com/torvalds/linux/master/drivers/accel/rocket/Kconfig (CONFIG_DRM_ACCEL_ROCKET e dependências)
- https://github.com/torvalds/linux/tree/v6.18/drivers/accel/rocket (pasta do driver na tag 6.18)
- https://cateee.net/lkddb/web-lkddb/DRM_ACCEL_ROCKET.html (versões do kernel em que a opção aparece: 6.18 em diante)
- https://docs.kernel.org/accel/rocket/index.html (documentação do driver rocket no kernel)
- https://woliveiras.com/posts/running-llms-on-single-board-computers-raspberry-pi-orange-pi-and-radxa (visão geral de CPU e NPU em SBCs; dados sem configuração)
- https://tinycomputers.io/posts/rk3588-orange-pi-5-max-review.md (review da Orange Pi 5 Max, 20/09/2025)
- https://tinycomputers.io/posts/rockchip-rk3588-npu-benchmarks.md (benchmark de RKNN e RKLLM na Max, com versões declaradas)
- https://turingpi.com/llm-inference-benchmarks-rk3588-gguf-quantization/ (llama.cpp em CPU no RK1, RK3588; inclui o pico de memória e a afirmação sobre 16 GB)
- https://github.com/airockchip/rknn-llm/issues/154 (log do RKLLM 1.1.4 com rknpu 0.9.8 numa Rock 5B+)
- https://raw.githubusercontent.com/airockchip/rknn-llm/main/doc/Rockchip_RKLLM_SDK_EN_1.3.1.pdf (SDK RKLLM 1.3.1, seção 2.4: driver v0.9.8 e comando de verificação)
- https://github.com/airockchip/rknn-llm/tree/main/rknpu-driver (listagem do pacote rknpu_driver_0.9.8_20241009.tar.bz2, sem extrair)
- https://docs.photonvision.org/en/latest/docs/benchmarks/rknn-model-benchmarks.html (YOLO no RKNN, Orange Pi 5 com RK3588)
- https://raw.githubusercontent.com/airockchip/rknn_model_zoo/main/README.md (tabela de desempenho de YOLO em FPS, RK3588 com um núcleo)
- https://github.com/geerlingguy/ai-benchmarks (README: sem entradas de Orange Pi ou RK3588)
- https://github.com/geerlingguy/sbc-reviews/issues/49 (issue da Orange Pi 5 Max, com comentários)
- https://github.com/ggml-org/llama.cpp/issues/722 (RK3588 perf: relatos em Orange Pi 5 e RK3588S, sem Max)
- https://github.com/ollama/ollama/issues/5001 (pedido de suporte ao NPU da RK3588, sem medição)

Fontes vistas só em resultados de busca (não abertas; citadas pelo nome, sem URL): CNX Software (release do RKLLM, 15/07/2024, e lançamento da Orange Pi 5 Max, 01/08/2024); documentação do Frigate espelhada em git (latências de YOLO); DietPi e Armbian (versões de driver e rocket); Phoronix (artigo sobre o rocket, só título); Hugging Face (conversão RKLLM de Llama 3.2 3B, sem número de tokens/s); TechSpot (6 TOPS e até 16 GB).

Não abertas por falha: matérias da Phoronix (HTTP 403; Web Archive não abriu); tag v6.17 de torvalds/linux (HTTP 404 para a pasta do rocket, usada como evidência negativa).
