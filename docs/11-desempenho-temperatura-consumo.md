# Orange Pi 5 Max: desempenho, temperatura e consumo (dados medidos)

Coleta feita em 03/11/2026. Este documento reúne números publicados por terceiros sobre a Orange Pi 5 Max (SoC Rockchip RK3588), além de dados do manual oficial (manual v1.3) e de arquivos do kernel mainline. Ele não traz medições próprias.

## Como ler este documento

- **[Max]**: o número foi medido na própria Orange Pi 5 Max, segundo a fonte, ou está no manual oficial da Max.
- **[placa irmã: nome]**: o número é de outra placa (Orange Pi 5, Orange Pi 5 Ultra, Raspberry Pi 5, Orange Pi RV2 etc.). Serve só como referência.
- **[RK3588, kernel mainline]**: o valor está no código do kernel Linux para o SoC RK3588. Não é uma medição da Max, e o .dts de uma placa específica pode sobrescrevê-lo.
- **Relato**: o dado vem de um comentário em issue, sem ferramenta, método ou data de teste informados. Não é medição validada.
- **lido de gráfico**: o valor foi lido de uma imagem de gráfico, não de texto ou tabela.
- **Suspeito**: o número existe na fonte, mas o método ou o valor não se sustentam. Não use como resultado válido.
- Quando uma fonte não informa um dado (condição, data, versão), a célula diz "não informado".
- Os números estão como aparecem na fonte, com vírgula decimal convertida para o padrão brasileiro quando o original usa ponto.
- O manual aparece como "manual v1.3, p. X", com a página impressa no rodapé.
- Os sites citados em "Fontes" foram abertos e lidos. Resumos de buscas que não foram abertos aparecem só como ressalva no texto, sem URL.

## Resumo rápido

- **CPU**: no Geekbench 6 em configuração de fábrica, a placa fez 731 pontos em núcleo único e 2.831 em multinúcleo (SkatterBencher #89). Outra execução, na issue #49 do sbc-reviews, deu 743 e 2.792. São referências, não média.
- **Memória**: a RAM é LPDDR5, conforme o manual v1.3 (p. 11, 12 e 15). A citação de LPDDR4X na review do TinyComputers é erro da review. Com tinymembench (issue #49, modelo de 8 GB), a cópia de memória ficou perto de 12 GB/s e o preenchimento perto de 28 GB/s. Não há STREAM publicado.
- **Consumo ocioso**: 1,4 W medido na tomada (issue #49) e 2,4 W com a tela desligada (gráfico do LinuxLinks, com Wi-Fi e Bluetooth ligados). As condições diferem, e as fontes não permitem separar a causa. Sob carga de CPU, a faixa observada vai de cerca de 10 W a 21 W, e um teste de overclock desligou a placa com a fonte da Pi 5 (a fonte foi apontada como causa). A fonte oficial é de 5 V / 5 A (manual v1.3, p. 13).
- **Consumo com SSD**: um comentário de 18/04/2026 na issue #49 relata cerca de 1 W em ociosidade sem SSD e cerca de 5 W com SSD NVMe ligado. É um relato, sem ferramenta nem método informados.
- **Temperatura**: com refrigeração ativa, o SoC ficou perto de 66 a 68 °C em carga moderada. O manual mostra sensores e o comando para lê-los (p. 221–222). Os trips do kernel para o RK3588 são alerta passivo em 85 °C e crítico em 115 °C [RK3588, kernel mainline]. Não há medição da Max sem refrigeração.
- **Rede**: iperf3 em 2,5 GbE deu 2,35 Gbps de envio, mas só 773 Mbps de recebimento na configuração padrão (issue #49). Wi-Fi deu 297 Mbps de envio e 176 Mbps de recebimento (issue #49). Detalhes na seção 6.
- **Armazenamento**: NVMe com iozone deu 1,04 GB/s de leitura sequencial e 2,26 GB/s de escrita (issue #49). O número de leitura de 5,7 GB/s continua suspeito. microSD deu 60 MB/s de leitura sequencial (issue #49). eMMC não tem medição.
- **GPU**: glmark2-es2 deu 770 pontos com o Mali-G610 (issue #49). Só há um teste, sem outra placa no mesmo sistema para comparar.

## 1. Desempenho de CPU

| Valor | Condição do teste | Placa | Fonte | Data |
|---|---|---|---|---|
| Geekbench 6 (Arm Preview): 731 pts em núcleo único, 2.831 pts em multinúcleo | Configuração de fábrica, Debian Bullseye Desktop, kernel 6.1.43 | [Max] | SkatterBencher #89 | Publicado em 23/08/2025, atualizado em 20/09/2025 |
| Geekbench 6: 743 pts em núcleo único, 2.792 pts em multinúcleo | Debian 11, kernel 6.1.43-rockchip, modelo de 8 GB | [Max] | GitHub, geerlingguy/sbc-reviews, issue #49 | Issue aberta em 17/08/2024; data do teste não informada |
| HPL (top500-benchmark): 52,924 Gflops e 4,13 Gflops/W | Mesmo relatório da issue #49; potência de 12,8 W durante o HPL (seção 4) | [Max] | GitHub, geerlingguy/sbc-reviews, issue #49 | Data do teste não informada |
| Geekbench 6: 805 pts em núcleo único, 3.070 em multinúcleo | Armbian 23.8.1, kernel 5.10.160-legacy-rk35xx; data da publicação não informada | [placa irmã: Orange Pi 5 4 GB] | geerlingguy/sbc-reviews, issue #5 | Não informada na página |
| 7-Zip: 15.230 MIPS | Configuração de fábrica, mesmo sistema | [Max] | SkatterBencher #89 | 23/08/2025 |
| AI Benchmark: 510 pts | Configuração de fábrica, mesmo sistema | [Max] | SkatterBencher #89 | 23/08/2025 |
| 7-Zip (sbc-bench): 15.960 multinúcleo, 2.994 núcleo único | Kernel 6.1, Bullseye, clocks 2.280/1.800 MHz; data não informada | [Max] | sbc-bench, Results.md | Data não informada na página |
| 7-Zip (sbc-bench): 16.780 multinúcleo, 2.689 núcleo único | Kernel 5.10, Jammy, clocks 2.400/1.800 MHz | [placa irmã: Orange Pi 5] | sbc-bench, Results.md | Data não informada na página |
| Sysbench CPU: 13.688,80 eventos/s (10 s), 136.916 eventos no total | Resfriamento ativo; número de threads não informado no resumo | [Max] | TinyComputers, review | 20/09/2025 |
| Geekbench 6 com overclock: ganho de cerca de 3,7% (só governors), 8,3% (memória OC), 27,2% (PVTPLL com tensão aumentada), 22,4% (overclock manual) | Ganhos em relação ao padrão, média geométrica; overclock não é a configuração padrão | [Max] | SkatterBencher #89 | 23/08/2025 |

Observações:

- A sbc-bench lista a Max numa linha própria, mas diz que as linhas RK35xx são "preliminares" e cita a Orange Pi 5 pelo nome, sem mencionar a Max. Não fica claro se a nota vale para a Max.
- As pontuações de 7-Zip divergem entre as duas fontes: 15.230 MIPS (SkatterBencher) contra 15.960 (sbc-bench). São saídas diferentes do mesmo programa, e a fonte não explica a diferença. Não é possível comparar as duas diretamente.
- O TinyComputers cita Geekbench 5 no texto, mas não publica o valor. O review também não traz 7-Zip nem sbc-bench.
- Há outras execuções públicas do Geekbench 6 para a Max. Só abri a issue #49, que cita uma delas.

Comparação com outras placas (mesma fonte, qualitativo):

- O LinuxLinks compara a Max com Raspberry Pi 5, Orange Pi RV2 e um mini PC Intel N100, usando Phoronix Test Suite (smallpt, compress-pbzip2, Stockfish, Coremark, RAMspeed). Os valores estão só em gráficos. Segundo o texto, a Max (versão de 8 GB) fica perto do N100 em Coremark e à frente da Pi 5 em smallpt, compress-pbzip2, Stockfish e Coremark. A matéria diz que as diferenças entre a Max e a Ultra são variação normal de benchmark. Fonte: LinuxLinks, "Benchmarking the Orange Pi 5 Ultra, Orange Pi 5 Max and Orange Pi RV2", publicado em 20/06/2025. Marcar como [Max] comparado com [placa irmã: Raspberry Pi 5] e [placa irmã: N100].

## 2. Desempenho de GPU e NPU

| Valor | Condição do teste | Placa | Fonte | Data |
|---|---|---|---|---|
| glmark2-es2: 770 pontos, janela de 800x600. Renderizador: Mali-G610, OpenGL ES 3.2 | Driver ARM g13p0; glmark2 2023.01; Debian 11 com Xfce. O relatório não deixa claro se é a variante es2 ou es2-wayland | [Max] | GitHub, geerlingguy/sbc-reviews, issue #49 | Issue aberta em 17/08/2024; data do teste não informada |
| glmark2 de cerca de 4.800 pontos | Ubuntu 22.04, governors em performance e PAN_MESA_DEBUG=gofaster. Sistema e driver diferentes do teste acima, então não é comparável | [placa irmã: Orange Pi 5] | Comentário na issue #49 do sbc-reviews | 07/09/2024 |
| NPU (RKLLM, TinyLlama 1,1B quantizado): cerca de 20,2 tokens/s (três rodadas: 20,27; 20,04; 20,40) | NPU em 1 GHz; quantização não detalhada no resumo | [Max] | TinyComputers, review | 20/09/2025 |

Observações:

- O TinyComputers afirma que a GPU não foi testada a fundo. O LinuxLinks excluiu a GPU dos seus testes de consumo com jogos e IA.
- O mesmo relatório da issue #49 traz um screenfetch que mostra a GPU como "llvmpipe" (renderização por software). O relatório não diz em que momento esse screenfetch foi rodado, então não dá para ligá-lo ao teste glmark2, que mostra o driver da Arm.
- São só um resultado sem repetições e um resultado de outro sistema. A diferença entre 770 e cerca de 4.800 mostra que sistema e driver pesam muito nesse teste.

## 3. Memória RAM

### Tipo de memória

| Afirmação | Placa | Fonte | Data |
|---|---|---|---|
| LPDDR5, de 4, 8 ou 16 GB | [Max] (especificação) | Manual v1.3, p. 11, 12 e 15 | Manual v1.3 |
| Modelo de 16 GB com LPDDR5; clock do controlador de memória (DMC) em 1.968 MHz no padrão e 3.192 MHz com overclock (o título fala em LPDDR5-6400, mas o valor medido é 3.192 MHz) | [Max] | SkatterBencher #89 | 23/08/2025 |
| Anúncio: 4, 8 ou 16 GB de LPDDR5 | [Max] (especificação) | CNX Software, lançamento | Publicado em 01/08/2024, atualizado em 03/12/2024 |
| 16 GB LPDDR4X-2133, 64 bits; cerca de 14,4 GB disponíveis para o sistema. **Erro da review:** o manual e as demais fontes dizem LPDDR5 | [Max] (unidade testada) | TinyComputers, review | 20/09/2025 |

A citação de LPDDR4X na review do TinyComputers foi tratada como erro da review, não como dúvida aberta. Ela contradiz o manual v1.3 (p. 11, 12 e 15) e as outras duas fontes.

### Largura de banda

| Valor | Condição do teste | Placa | Fonte | Data |
|---|---|---|---|---|
| Cópia (memcpy): 12.093,2 MB/s; preenchimento (memset): 28.018,2 MB/s; cópia NEON LDP/STP: 12.069,4 MB/s | tinymembench v0.4.10; modelo de 8 GB; Debian 11, kernel 6.1.43-rockchip | [Max] | GitHub, geerlingguy/sbc-reviews, issue #49 | Data do teste não informada |
| Latência de leitura aleatória: 249,7 ns com 64 MiB (278,7 ns com duas leituras simultâneas) | Mesmo teste (tinymembench, issue #49) | [Max] | GitHub, geerlingguy/sbc-reviews, issue #49 | Data do teste não informada |
| 0,10 GB/s | "Teste simples de largura de banda". A ferramenta e os parâmetros não são citados. | [Max] | TinyComputers, review | 20/09/2025 |
| Copy (memcpy): 12.420,1 MB/s; memset: 29.405,7 MB/s; latência de leitura aleatória de 236,4 ns com 64 MiB | tinymembench v0.4.10; Armbian 23.8.1, kernel 5.10.160 | [placa irmã: Orange Pi 5 4 GB] | geerlingguy/sbc-reviews, issue #5 | Não informada na página |

Suspeito: 0,10 GB/s é muito baixo para memória LPDDR de 16 GB, e o método não é descrito. O tinymembench da issue #49, com a mesma placa (modelo de 8 GB), mede cerca de 12 GB/s de cópia. Por isso o 0,10 GB/s não serve como largura de banda real. O próprio review diz que não há referência de base para interpretá-lo.

A linha da Orange Pi 5 4 GB é de outra placa e outro sistema, mas usa a mesma ferramenta e mostra a mesma ordem de grandeza (cerca de 12 GB/s de cópia).

Sem medição de STREAM ou lmbench para a Max. O LinuxLinks mediu RAMspeed com Phoronix, mas os gráficos mostram só a posição relativa da Max (à frente dos concorrentes), sem valores no texto.

Relato único em resultado de busca, não aberto e não verificado: uma avaliação de compra descreve travamentos (erro SError no console serial) com muita leitura e escrita em volume btrfs. Trate como um caso isolado, não como fato.

## 4. Consumo de energia

| Valor | Condição do teste | Placa | Fonte | Data |
|---|---|---|---|---|
| Ocioso, medido na tomada: 1,4 W | "Idle power draw (at wall)". Debian 11, kernel 6.1.43. Estado do Wi-Fi, do Bluetooth e do NVMe não informado | [Max] | GitHub, geerlingguy/sbc-reviews, issue #49 | Data do teste não informada |
| Ocioso, tela desligada: 2,4 W (lido de gráfico; a tabela em Wh mostra 2,4 Wh) | Uso contínuo sem tela, Ubuntu 22.04, Wi-Fi e Bluetooth ligados, sem underclock. CPU em 408 MHz no ocioso | [Max] | LinuxLinks, Power, página 1 (gráfico, lido de gráfico) e página 4 (tabela) | 09/07/2025 |
| Ocioso com SSD NVMe ligado: cerca de 5 W (sem SSD, cerca de 1 W) | Relato de comentário, sem ferramenta, medida ou data do teste. O autor atribui o aumento à falta de ASPM L1.2 (CLKREQ# não ligado) | [Max] (relato) | GitHub, geerlingguy/sbc-reviews, issue #49, comentário | 18/04/2026 |
| Uso leve: 3,9 Wh (média de 3,9 W) | Cerca de 8 horas por dia de uso geral (navegação, e-mail, compilação, vídeo, música), sem jogos ou IA local, Ubuntu 22.04.2 | [Max] | LinuxLinks, Power, páginas 2 e 4 | 09/07/2025 |
| Carga máxima simulada (`stress-ng --matrix 0`): 10,9 W | Medido na tomada | [Max] | GitHub, geerlingguy/sbc-reviews, issue #49 | Data do teste não informada |
| Geekbench 6 multinúcleo: 9,9 W | Medido na tomada | [Max] | GitHub, geerlingguy/sbc-reviews, issue #49 | Data do teste não informada |
| HPL (top500-benchmark): 12,8 W | Medido na tomada | [Max] | GitHub, geerlingguy/sbc-reviews, issue #49 | Data do teste não informada |
| Ociosidade: 1,0 W; carga de stress-ng (matrix): 10,0 W; Geekbench multinúcleo: 8,9 W; HPL: 11,5 W | Medido na tomada; Armbian 23.8.1, kernel 5.10.160 | [placa irmã: Orange Pi 5 4 GB] | geerlingguy/sbc-reviews, issue #5 | Não informada na página |
| Ociosidade headless: 2,5 Wh | Mesmas condições da tabela do LinuxLinks | [placa irmã: Orange Pi RV2] | LinuxLinks, Power, página 4 | 09/07/2025 |
| Ociosidade headless: 3,5 Wh | Mesmas condições | [placa irmã: Raspberry Pi 5] | LinuxLinks, Power, página 4 | 09/07/2025 |
| Uso leve: 3,7 Wh (RV2) e 4,9 Wh (Pi 5) | Mesmas condições | [placa irmã: Orange Pi RV2] e [placa irmã: Raspberry Pi 5] | LinuxLinks, Power, página 4 | 09/07/2025 |
| Carga de CPU (s-tui, stress): cerca de 10 W em padrão, cerca de 12 W com governors ajustados, cerca de 21 W com PVTPLL ou overclock manual | Medido na tomada, carga só de CPU, sem informação de duração ou resfriamento | [Max] | SkatterBencher #89 | 23/08/2025 |
| Carga típica: 15 a 25 W | Sem ferramenta e duração informadas; o review diz ter usado resfriamento ativo nas medições de temperatura | [Max] | TinyComputers, review | 20/09/2025 |
| Teste de estresse de High Performance Linpack: a placa desligou em cerca de 33 W com a fonte da Pi 5; com outra fonte USB-C chegou a quase 40 W | Overclock; a queda foi atribuída à fonte de alimentação | [Max] | SkatterBencher #89 | 23/08/2025 |
| Carga de CPU: "um pouco mais" que a RV2 e a Pi 5 | Gráfico, sem valores no texto | [Max] comparada com [placa irmã: Orange Pi RV2] e [placa irmã: Raspberry Pi 5] | LinuxLinks, Power, página 3 | 09/07/2025 |

Fonte de alimentação oficial: USB Type-C de 5 V / 5 A (manual v1.3, p. 13).

Observações:

- O gráfico do LinuxLinks está em watts e confere com a tabela em Wh (2,4 Wh com média de 2,4 W). A fonte mede o sistema inteiro, sem o monitor.
- Os dois valores de ociosidade da Max (1,4 W e 2,4 W) vêm de condições diferentes: sistema operacional, Wi-Fi e Bluetooth ligados ou não, e estado do NVMe. As fontes não permitem separar a causa.
- A faixa de 15 a 25 W do TinyComputers é maior que a medida pelo SkatterBencher em carga de CPU (cerca de 10 a 12 W em padrão) e pela issue #49 (de 9,9 a 12,8 W). As condições não são iguais, então a diferença não pode ser atribuída a uma causa.
- As linhas do Orange Pi 5 4 GB (issue #5) não são comparáveis diretamente com a Max: o sistema operacional e a versão do kernel são diferentes, e a carga de stress-ng usa outra medição. Mesmo assim, a ociosidade de 1,0 W contra 1,4 W e 2,4 W da Max sugere que as condições de ociosidade (Wi-Fi, Bluetooth, sistema e armazenamento) pesam no resultado.
- Consumo com NVMe: o único dado é o relato de comentário (cerca de 1 W sem SSD e cerca de 5 W com SSD). Ele não tem ferramenta nem método. O mesmo comentário pergunta se o 1,4 W da issue #49 foi medido com NVMe instalado, e nenhum comentário lido responde.
- Consumo com microSD ou com 2,5 GbE sob carga: sem medição publicada para a Max. Procurei no artigo de consumo do LinuxLinks (quatro páginas), na issue #49 e em buscas. O artigo de benchmark exclui o disco porque a placa não vem com armazenamento.
- Um resultado de busca cita 5 a 15 W em uso sustentado, dependendo do resfriamento. A fonte não foi aberta e não tem URL aqui.

## 5. Temperatura e throttling

### Limiares do kernel para o RK3588 [RK3588, kernel mainline]

Fonte: `rk3588-base.dtsi`, do kernel mainline (acessado em 03/11/2026, sem data de publicação no arquivo).

| Zona térmica | Alerta "passive" | Crítico "critical" | Histerese do alerta |
|---|---|---|---|
| bigcore0 (cpu_b0, cpu_b1) | 85 °C | 115 °C | 2 °C |
| bigcore2 (cpu_b2, cpu_b3) | 85 °C | 115 °C | 2 °C |
| littlecore (cpu_l0 a cpu_l3) | 85 °C | 115 °C | 2 °C |
| gpu | 85 °C | 115 °C | 2 °C |
| package, center, npu | não tem alerta passivo | 115 °C | 0 °C |

- Os trips "passive" têm mapa de resfriamento (frequência da CPU e da GPU). É aqui que o kernel reduz o clock. Esse é o limiar de throttling do SoC, segundo o código genérico do RK3588: 85 °C.
- O trip "critical" de 115 °C é o limite para o kernel desligar ou parar a placa por software. O desligamento por hardware (tsadc, `hw-tshut-temp`) fica em 120 °C no mesmo arquivo.
- O manual mostra `soc_thermal` com crit = 115,0 °C (manual v1.3, p. 222). Isso bate com o trip crítico do mainline.
- Não localizei o device tree do BSP da Orange Pi nesta coleta, então não comparei os valores do BSP com os do mainline. O .dts da Max pode sobrescrever esses valores, e isso não foi verificado.

### Medições

| Valor | Condição do teste | Placa | Fonte | Data |
|---|---|---|---|---|
| Sensores do kernel (`sensors`): 47,2 °C em todas as zonas (gpu, littlecore, bigcore0/1, npu, center, soc) | Leitura única no manual; carga e resfriamento não informados | [Max] | Manual v1.3, p. 221–222 | Manual v1.3 |
| Temperatura do NVMe (`smartctl -a /dev/nvme0`): 40 °C | Exemplo do manual; carga e modelo do SSD não informados | [Max] | Manual v1.3, p. 222 | Manual v1.3 |
| SoC 66,5 °C; clusters 66,5 °C e 67,5 °C; centro, GPU e NPU 65,6 °C | Carga moderada, resfriamento ativo; temperatura ambiente e duração não informadas | [Max] | TinyComputers, review | 20/09/2025 |
| Throttling: não reportado | Mesma condição; o autor avisa que resfriamento passivo pode não bastar em cargas longas | [Max] | TinyComputers, review | 20/09/2025 |
| SoC em média 37,0 °C (padrão); 38,6 °C (governors); 38,7 °C (memória OC); 52,0 °C (PVTPLL); 52,7 °C (overclock manual) | Carga de CPU com s-tui; resfriamento não informado | [Max] | SkatterBencher #89 | 23/08/2025 |
| Pico de cerca de 70 °C no High Performance Linpack, com tensões de 1,3 V e 1,25 V | Overclock; resfriamento não informado | [Max] | SkatterBencher #89 | 23/08/2025 |
| Acima de 85 °C sob carga sustentada sem dissipador; throttling em 30 a 60 s de carga total | Sem dissipador, carga sustentada | [placa irmã: Orange Pi 5, RK3588S] | Armbian, "Stay cool" | 23/07/2025 |

Como medir: manual v1.3, seção 3.15 ("Temperature sensor", p. 221–222). O comando `sensors` lista as zonas do kernel, e `sudo smartctl -a /dev/nvme0 | grep "Temperature:"` lê o SSD NVMe.

Observações:

- A diferença entre 37 °C (SkatterBencher, carga de CPU) e 66 °C (TinyComputers, carga moderada) pode vir da carga, da duração e do resfriamento. Não é possível separar essas causas com as informações das fontes.
- A faixa de 70 a 95 °C do Armbian, citada antes, é genérica. Foi substituída pelos limiares do mainline para o RK3588 (85 °C passivo e 115 °C crítico).
- Temperatura da Max sem refrigeração: sem medição publicada. Os dados sem refrigeração que existem são de Orange Pi 5 (RK3588S), não da Max. Procurei na issue #49 do sbc-reviews (sem temperatura), no Hackster (só a matéria de lançamento) e em busca web. Resultados de busca sobre o Phoronix (Orange Pi 5 sem dissipador, média de 71 °C e pico de 85 °C) e sobre um cooler de 52Pi (queda de cerca de 71–76 °C para 45–50 °C com ventilador) são de Orange Pi 5. Não foram abertos (o Phoronix responde 403), então não os uso.
- Relatos de fórum sobre a Max com ventilador em caixa metálica (cerca de 48 °C) existem em resultados de busca, mas não os usei porque não abri a fonte.

## 6. Rede

| Valor | Condição do teste | Placa | Fonte | Data |
|---|---|---|---|---|
| Ethernet 2,5 GbE, controlador Realtek RTL8125BG | Especificação de fábrica (manual v1.3, p. 12) | [Max] | Manual v1.3; CNX Software, lançamento (01/08/2024) | Manual v1.3 |
| Wi-Fi 6E e Bluetooth 5.3 com módulo AP6611, Wi-Fi via SDIO 3.0 | Especificação de fábrica (manual v1.3, p. 12) | [Max] | Manual v1.3; CNX Software, lançamento | Manual v1.3 |
| Wi-Fi descrito como "dual-band" e confiável | Descrição qualitativa, sem número | [Max] | TinyComputers, review | 20/09/2025 |
| iperf3 em 2,5 GbE: 2,35 Gbps de envio; 773 Mbps de recebimento (`--reverse`); bidirecional: 2,33 Gbps de envio e 362 Mbps de recebimento | Kernel 6.1.43, Debian 11 (conforme o relatório). Configuração de ajustes, servidor, roteador e cabo não informados | [Max] | GitHub, geerlingguy/sbc-reviews, issue #49 | Data do teste não informada |
| Recebimento de 2,35 Gbps depois de mover a interrupção do RTL8125 para os núcleos A76 grandes | Relato de comentário na issue #49: o autor diz que a recepção era lenta por padrão. O comentário não diz o modelo da placa | [Max] (relato) | GitHub, geerlingguy/sbc-reviews, issue #49, comentário | 13/09/2024 |
| Wi-Fi (chip Synaptics AP6611S, como escrito na issue): 297 Mbps de envio; 176 Mbps de recebimento; bidirecional: 237 Mbps de envio e 38 Mbps de recebimento | Kernel 6.1.43, Debian 11 (conforme o relatório). Banda, distância, roteador e servidor não informados | [Max] | GitHub, geerlingguy/sbc-reviews, issue #49 | Data do teste não informada |
| iperf3 em 1 GbE: 942 Mbps (ida), 906 Mbps (volta), 931 Mbps subida / 340 Mbps descida em bidirecional | Ethernet de 1 GbE, Armbian 23.8.1, kernel 5.10.160 | [placa irmã: Orange Pi 5 4 GB] | geerlingguy/sbc-reviews, issue #5 | Não informada na página |

Observações:

- O TinyComputers afirma que o Ethernet e o Wi-Fi funcionaram bem, mas não publica nenhuma velocidade.
- O recebimento de 773 Mbps na issue #49 é o gargalo da placa nesse teste. O relato de 13/09/2024 diz que a velocidade cheia voltou depois de mover a interrupção da rede para os núcleos A76. Isso não foi reproduzido por outra fonte.
- Não há medição de Wi-Fi 6E com banda informada. A única medição de Wi-Fi da Max (issue #49) não diz a banda, a distância nem o roteador.

## 7. Armazenamento

| Valor | Condição do teste | Placa | Fonte | Data |
|---|---|---|---|---|
| NVMe leitura sequencial de "até 5,7 GB/s" | SSD de 1 TB em M.2 PCIe 3.0 x4; ferramenta, tamanho de bloco, profundidade de fila e estado (frio ou quente) não informados | [Max] | TinyComputers, review | 20/09/2025 |
| NVMe escrita sequencial de 2,1 GB/s | Mesma condição, sem ferramenta informada | [Max] | TinyComputers, review | 20/09/2025 |
| NVMe com iozone (MakerDisk NVMe 2280, 512 GB): leitura sequencial de 1M: 1.038,04 MB/s; escrita sequencial de 1M: 2.256,93 MB/s; leitura aleatória de 1M: 1.028,30 MB/s; escrita aleatória de 1M: 1.460,79 MB/s; leitura aleatória de 4K: 47,50 MB/s; escrita aleatória de 4K: 135,29 MB/s | disk-benchmark.sh (geerlingguy), TEST_SIZE=1g, em `/`. Largura do link PCIe não informada | [Max] | GitHub, geerlingguy/sbc-reviews, issue #49 | Data do teste não informada |
| NVMe com iozone (SSD da Orange Pi 5, comentário): leitura sequencial de 1M: 368,94 MB/s; escrita sequencial de 1M: 372,99 MB/s; leitura aleatória de 4K: 50,70 MB/s; escrita aleatória de 4K: 123,35 MB/s | Mesmo script; o comentário diz que a Max tem mais pistas PCIe que a Orange Pi 5 | [placa irmã: Orange Pi 5] | GitHub, geerlingguy/sbc-reviews, issue #49, comentário | 07/09/2024 |
| NVMe leitura sequencial de 1M (fio): 428 MB/s; escrita aleatória de 4K (iozone): 80,85 MB/s | SSD KIOXIA XG6 de 1 TB; a fonte não informa a largura do link PCIe | [placa irmã: Orange Pi 5 4 GB] | geerlingguy/sbc-reviews, issue #5 | Não informada na página |
| microSD (SanDisk Ultra 32 GB A1): leitura sequencial de 1M: 60,18 MB/s; escrita sequencial de 1M: 20,51 MB/s; leitura aleatória de 4K: 10,01 MB/s; escrita aleatória de 4K: 2,20 MB/s | Mesmo script (iozone), TEST_SIZE=1g | [Max] | GitHub, geerlingguy/sbc-reviews, issue #49 | Data do teste não informada |
| microSD: cartão de 64 GB (58 GB utilizáveis), UHS-I suportado; velocidade não informada | Não há medição | [Max] | TinyComputers, review | 20/09/2025 |
| microSD leitura sequencial de 1M (fio): 68,1 MB/s; escrita aleatória de 4K (iozone): 3,36 MB/s | Cartão SanDisk Extreme de 128 GB | [placa irmã: Orange Pi 5 4 GB] | geerlingguy/sbc-reviews, issue #5 | Não informada na página |
| Slot M.2 M-Key PCIe 3.0 x4 e slot de microSD (TF) | Especificação de fábrica (manual v1.3, p. 12–13) | [Max] | Manual v1.3 | Manual v1.3 |
| eMMC: soquete para módulos de 32 a 256 GB ou eMMC soldada na placa | Especificação de fábrica; sem velocidade | [Max] | CNX Software, lançamento | 01/08/2024 |

**Valor suspeito: NVMe leitura de 5,7 GB/s.** O manual confirma PCIe 3.0 x4 no slot M.2 (p. 12–13). O teto teórico desse link é de cerca de 3,94 GB/s (8 GT/s por pista, 4 pistas, codificação 128b/130b). Um valor acima disso só seria possível com erro de medição, cache do sistema ou outra falha de método. A fonte não diz como mediu. A medição de iozone da issue #49, na mesma placa, deu cerca de 1,04 GB/s de leitura sequencial, e não reproduz o 5,7 GB/s. Por isso, o número não deve ser usado como resultado válido. A escrita de 2,1 GB/s é plausível e está dentro do limite do barramento.

Armazenamento da Max sem medição: eMMC. O guia do linux.org sobre eMMC e NVMe na Max tem uma seção de velocidades, mas a página respondeu com um desafio de verificação (Cloudflare) nesta coleta. Não foi aberta nem contornada. Fichas de vendedores de módulos eMMC (com "até 400 MB/s") são números de marketing, sem teste na Max, e não foram usadas.

Para confirmar o link PCIe negociado com um NVMe na Max, rode `lspci -vv` e leia o campo LnkSta. Para repetir o teste de leitura, use `fio --rw=read --bs=1M --direct=1`.

## 8. Divergências registradas

1. **Tipo de RAM**: LPDDR4X (TinyComputers) contra LPDDR5 (manual v1.3, SkatterBencher e anúncio). Erro da review: o manual confirma LPDDR5.
2. **Largura de banda de RAM**: 0,10 GB/s do TinyComputers, com método não descrito, contra cerca de 12 GB/s de cópia do tinymembench na issue #49 (mesma placa, modelo de 8 GB). Considerar suspeito.
3. **NVMe leitura**: 5,7 GB/s do TinyComputers, acima do teto de PCIe 3.0 x4, contra 1,04 GB/s de leitura sequencial do iozone na issue #49. Considerar suspeito.
4. **7-Zip**: 15.230 MIPS (SkatterBencher) contra 15.960 (sbc-bench). Métricas ou configurações diferentes.
5. **Consumo sob carga**: 10 a 21 W (SkatterBencher, CPU) e 9,9 a 12,8 W (issue #49) contra 15 a 25 W (TinyComputers). Condições diferentes.
6. **Temperatura**: cerca de 37 °C (SkatterBencher, carga de CPU) contra cerca de 66 °C (TinyComputers, carga moderada com ventilador). Condições diferentes.
7. **Consumo sustentado**: uma fonte de busca cita 5 a 15 W, o que não bate com a faixa do TinyComputers. Fonte não aberta.
8. **Consumo ocioso**: 1,4 W (issue #49, medido na tomada) contra 2,4 W (LinuxLinks, tela desligada, com Wi-Fi e Bluetooth ligados). Condições diferentes; não é possível separar a causa.

## 9. Lacunas

Sem resposta nas fontes abertas. Onde procurei:

- **STREAM e lmbench**: busca web e o artigo de benchmark do LinuxLinks (RAMspeed só em gráfico).
- **Wi-Fi 6E da Max com banda informada**: a issue #49 é a única medição, sem banda, distância ou roteador. Busca web por iperf3 e Wi-Fi da Max não achou outra medição.
- **Largura do link PCIe do SSD usado na issue #49**: não informada. Por isso o resultado de NVMe não diz se é PCIe 2 ou 3.
- **fio no NVMe da Max**: sem medição com fio. A busca trouxe resultados de Orange Pi 5 Plus e Pi 5, que não abri.
- **eMMC da Max**: sem medição. O guia do linux.org está protegido por um desafio de verificação e não foi aberto. Fichas de vendedores não servem como medição.
- **Temperatura da Max sem refrigeração**: Hackster (só lançamento), busca web, issue #49 (sem temperatura) e o Phoronix (403). A issue #100 do sbc-bench, que pode ter resultados de sbc-bench da Max, só aponta para um arquivo externo, que não foi baixado.
- **Consumo com NVMe confirmado**: só o relato de comentário de 18/04/2026, sem método e sem resposta na issue. O 1,4 W da issue #49 não diz se havia NVMe.
- **Consumo com microSD e com 2,5 GbE sob carga**: sem medição publicada.
- **Device tree do BSP da Orange Pi**: não localizado nesta coleta.
- **PiBenchmarks**: a página inicial lida não traz link para a Max, e a busca não achou página de disco da Max.
- **Armbian**: a busca não achou thread com dados de Max sobre iperf, temperatura ou NVMe que eu pudesse abrir.
- **Notebookcheck**: as buscas retornaram só notícias de lançamento, sem medições. Não abri nenhuma página do Notebookcheck com dados.
- **CNX Software**: a matéria de lançamento traz especificações, não medições de desempenho.
- **Gráficos do LinuxLinks**: só o gráfico de ociosidade foi lido (ver seção 4). Os de carga e de Phoronix só aparecem como imagem sem valores no texto, e não foram extraídos.

## Fontes

Todas foram abertas nesta coleta, em 03/11/2026, salvo quando indicado.

- Manual oficial da Orange Pi 5 Max, v1.3 (citado como "manual v1.3, p. X"; páginas 11, 12, 13, 15, 221 e 222). Versão local, sem URL pública neste documento.
- Arquivo `rk3588-base.dtsi`, kernel Linux mainline: https://raw.githubusercontent.com/torvalds/linux/master/arch/arm64/boot/dts/rockchip/rk3588-base.dtsi
- LinuxLinks, "Orange Pi 5 Max Single Board Computer: Power Consumption", página 1 (Steve Emms, 09/07/2025): https://www.linuxlinks.com/orangepi5-max-single-board-computer-power-consumption/
- LinuxLinks, consumo, página 2 (09/07/2025): https://www.linuxlinks.com/orangepi5-max-single-board-computer-power-consumption/2/
- LinuxLinks, consumo, página 3 (09/07/2025): https://www.linuxlinks.com/orangepi5-max-single-board-computer-power-consumption/3/
- LinuxLinks, consumo, página 4 (09/07/2025): https://www.linuxlinks.com/orangepi5-max-single-board-computer-power-consumption/4/
- LinuxLinks, gráfico de ociosidade da Max, "Power Consumption: Idle" (lido em 03/11/2026, valores lidos de gráfico): https://www.linuxlinks.com/wp-content/uploads/2025/07/OrangePi5-Max-Power-Consumption-Idle-Chart-1536x1072.png
- LinuxLinks, "Benchmarking the Orange Pi 5 Ultra, Orange Pi 5 Max and Orange Pi RV2" (20/06/2025): https://www.linuxlinks.com/benchmarking-the-orangepi-5-ultra-orange-pi-5-max-and-orangepi-rv2/
- geerlingguy/sbc-reviews, issue #49, "Orange Pi 5 Max" (aberta em 17/08/2024, atualizada em 20/04/2026; corpo e comentários lidos com a CLI do GitHub): https://github.com/geerlingguy/sbc-reviews/issues/49
- geerlingguy/sbc-reviews, issue #5, sobre Orange Pi 5 4 GB (data não informada na página): https://github.com/geerlingguy/sbc-reviews/issues/5
- ThomasKaiser/sbc-bench, issue #100, "Results for Orange Pi 5 Max" (aberta em 24/08/2024; corpo só aponta para um arquivo externo, não baixado): https://github.com/ThomasKaiser/sbc-bench/issues/100
- PiBenchmarks, página inicial (lida; sem dado da Max): https://pibenchmarks.com/
- TinyComputers, A.C. Jokela, "RK3588 Orange Pi 5 Max Review" (20/09/2025): https://tinycomputers.io/posts/rk3588-orange-pi-5-max-review.html
- SkatterBencher #89, "Orange Pi 5 Max Overclocked to 2650 MHz" (publicado em 23/08/2025, atualizado em 20/09/2025): https://skatterbencher.com/2025/08/23/skatterbencher-89-orange-pi-5-max-overclocked-to-2650-mhz/
- sbc-bench, Results.md (Thomas Kaiser; data não informada na página): https://github.com/ThomasKaiser/sbc-bench/blob/master/Results.md
- Armbian, "Stay cool" (23/07/2025): https://blog.armbian.com/stay-cool/
- CNX Software, lançamento da Orange Pi 5 Max (01/08/2024, atualizado em 03/12/2024): https://www.cnx-software.com/2024/08/01/rockchip-rk3588-powered-orange-pi-5-max-sbc-features-up-to-16gb-lpddr5-2-5gbe-onboard-wifi-6e-and-bluetooth-5-3/
