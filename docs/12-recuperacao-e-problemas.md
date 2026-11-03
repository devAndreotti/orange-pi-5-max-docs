# Orange Pi 5 Max: diagnóstico e recuperação

Data da coleta: 03/11/2026. Revisão com o manual oficial v1.3 e o esquemático oficial V1.1 em 03/11/2026. Fóruns do Armbian e do DietPi e a wiki da comunidade (bunnyamin/bunnix) foram lidos na mesma data.

Este guia reúne sintomas, causas prováveis, formas de confirmar e ações de recuperação para a Orange Pi 5 Max (SoC Rockchip RK3588). Cada afirmação relevante traz uma marca de origem. Os números de página citam o manual v1.3 ("manual v1.3, p. X"), que é o número impresso no rodapé. Os números de página do esquemático ("esquemático V1.1, p. X") são os do PDF, que tem 29 folhas.

## Como ler as marcas

- **[oficial: manual v1.3, p. X]**: vem do manual oficial da Orange Pi para a Max.
- **[oficial: esquemático V1.1, p. X]**: vem do esquemático oficial da Max, lido pelo texto extraído do PDF. As figuras não foram vistas.
- **[comunidade/Max]**: relato ou guia de usuários com a própria Max. Não é oficial e precisa ser testado. Cada relato traz data e link.
- **[placa irmã: nome]**: procedimento de outra placa com o mesmo SoC ou da mesma família. Pode não valer para a Max.
- **[trecho de busca]**: informação que veio só do resumo de um resultado de busca. A página não foi aberta.
- **[RK3588]**: fato do chip ou da documentação genérica do RK3588, não específico da Max.
- **[não confirmado]**: não achei fonte confiável, as fontes não concordam, ou a informação veio só de trecho de busca. O texto diz onde procurei.

## Limitações desta revisão

- O manual v1.3 responde a maior parte das perguntas sobre hardware, botões, gravação e console serial. As figuras das p. 15, 54, 57, 136, 137, 138 e 223 foram lidas como imagem.
- O esquemático V1.1 foi lido pelo texto extraído do PDF (folhas 6, 11, 14, 27 e 28). Os nomes e valores dos componentes vieram desse texto.
- A wiki da Rockchip (`opensource.rock-chips.com`) não resolveu o nome de domínio na revisão anterior. Nesta revisão não tentei de novo. Por isso a documentação do rkdeveloptool e a ordem de boot do RK3588 continuam sem fonte da Rockchip.
- O manual v1.3 não cita `rkdeveloptool` nem `upgrade_tool`. Procurei o texto inteiro por essas palavras e não achei nenhuma ocorrência.
- O U-Boot de referência (`doc/board/rockchip/rockchip.rst`) não traz a ordem de boot do RK3588 nem o apagamento da SPI.
- Não abri as threads do fórum oficial da Orange Pi (orangepi.org) que a busca encontrou. As threads encontradas são sobre Android, cases e aceleração de vídeo, não sobre os temas deste guia.
- Não abri a matéria da CNX nem a página de releases de projetos de UEFI citados em buscas. Os itens que dependem delas ficam como [trecho de busca].

## Antes de tudo: regras de segurança

- Desligue a alimentação e retire a energia antes de mexer em jumpers, fios de serial, MaskROM ou módulos. O manual pede isso para gravar pelo MaskROM (p. 57).
- Faça backup de tudo que estiver no microSD, eMMC ou NVMe antes de qualquer gravação ou apagamento.
- **Apagar a SPI (Erase All no RKDevTool) remove o bootloader gravado nela. Isso é destrutivo.** Ver seção 13.
- **Gravar uma imagem no NVMe ou na eMMC sobrescreve o que já existe lá.** Confira o destino antes de confirmar.
- **Comandos `dd` e `wl` escrevem no dispositivo de destino e apagam o que estiver lá. Um destino errado, como `/dev/sda` em vez do cartão, apaga o disco do sistema.** Ver seções 10 e 14.
- Use fonte de 5 V fixo. Nunca acima de 5 V (manual v1.3, p. 135).

## 1. Fonte insuficiente ou tensão errada

**Fatos confirmados:**
- A entrada Type-C aceita 5 V/5 A (manual v1.3, p. 13, especificação; p. 18, recomendação).
- A entrada Type-C não negocia PD e aceita só 5 V fixos (manual v1.3, p. 18).
- No esquemático V1.1 (p. 14), os canais CC1 e CC2 têm só resistores de 5,1 kΩ para GND. Não há controlador PD no bloco da entrada.
- O manual diz para não usar fonte com tensão acima de 5 V, porque isso queima a placa (manual v1.3, p. 135).
- Quedas de tensão e fonte instável causam boa parte dos problemas de boot. Se a placa reinicia em loop, troque a fonte ou o cabo Type-C (manual v1.3, p. 135).
- Não alimente a placa pela USB de um computador (manual v1.3, p. 136).

**Divergência registrada:** o manual fala em 5 V/5 A no cap. 1 e no cap. 2.1 (p. 13, 18), mas em 5 V/4 A no cap. 2.15 (p. 135) e no cap. 2.17 (p. 144). O esquemático V1.1 (p. 14) também rotula a entrada como "DC: 5V 4A" (texto do PDF com caracteres ilegíveis no meio). Use pelo menos 5 V/5 A.

**Relato sobre o PD [comunidade/Max, thread de 26/08/2025 a 29/08/2025, sem ano na página]:** um membro explica que o FUSB302 não é implementado e que a placa é alimentada sem negociação PD, como os 5 V/4 A do esquema. Confirma o que o esquema mostra. Não é fonte oficial.

**Sintomas:** reinícios, travamentos sob carga, HDMI sem imagem, NVMe que some, boot que falha de forma aleatória.

**Como confirmar:** troque por uma fonte de 5 V e 5 A conhecida e veja se o problema some. Uma fonte de notebook ou de celular com a tensão certa mas corrente menor pode não servir.

**O que fazer:** este é o passo menos arriscado e geralmente o primeiro a testar.

## 2. O que o LED indica

**Fatos confirmados (manual v1.3, p. 153 a 154 e p. 157):**
- A placa tem um LED RGB tricolor, com vermelho, verde e azul.
- O LED vermelho fica aceso enquanto a placa está com energia. Ele continua aceso mesmo com verde e azul desligados.
- O verde e o azul são controlados pelo sistema operacional. Depois que o kernel sobe, eles podem piscar.
- Verde piscando indica que o Linux subiu normalmente. Só vermelho aceso indica que o sistema nem chegou a iniciar.

**Esquema [oficial: esquemático V1.1, p. 28]:** o LED RGB (1206LED_RGB) é controlado pelas linhas PWM_LED1 e PWM_LED2, junto com o botão.

**Trecho de busca [não confirmado]:** um patch de device tree para a Max (linux-rockchip, out/2024, https://lists.infradead.org/pipermail/linux-rockchip/2024-October/052349.html) diz que o vermelho fica ligado com a energia e que azul e verde usam o gatilho "heartbeat". Não abri o patch. Um patch da Ultra depois sugere que a polaridade pode estar invertida nessa família.

**Relatos [comunidade/Max]:**
- 17/05/2025 (thread 51994, https://forum.armbian.com/topic/51994-orange-pi-5-max-sd-card-booting-issue-system-normally-starts-from-nvme-ssd/): com o cartão microSD inserido, o LED ficou azul, sem sinal no HDMI, e o sistema não subiu. Sem o cartão, a placa subiu pelo NVMe.
- 24/05/2025 (mesma thread): com a imagem Armbian da Max, o LED ficou só verde e não apareceu nada na tela. Com a imagem Ubuntu da Orange Pi, a placa subiu.

**Não confirmado:** o que cada cor significa em cada etapa do boot. Os relatos não dizem a etapa. Procurei nas threads do Armbian, nos patches do linux-rockchip e no fórum da Orange Pi, sem achar tabela de cores.

**Como confirmar:** o LED vermelho acende ao ligar? O verde pisca depois de alguns segundos? O azul aparece com cartão inserido? Anote isso antes de qualquer outra ação.

**O que fazer:** use o LED como pista de energia e de início do Linux. Para saber onde o boot parou, use o console serial (seção 4).

## 3. Placa não liga ou fica em boot loop

**Sintoma:** nenhuma imagem, o sistema reinicia repetidamente, ou o boot para em alguma mensagem.

**Causas prováveis:**
1. Fonte insuficiente ou com queda de tensão (seção 1).
2. Imagem de sistema ou bootloader incompatível com a Max. Um relato de usuário de Armbian diz que a imagem da Orange Pi 5 e da 5 Pro funcionou, mas a da Max não [comunidade/Max, 24/05/2025, thread 51994]. Até novembro de 2024 não havia imagem Armbian oficial para a Max. A thread 44637 foi fechada em 13/11/2024 apontando para o repositório da comunidade [comunidade/Max, https://forum.armbian.com/topic/44637-orange-pi-5-max-debian/].
3. Bootloader antigo ou corrompido na SPI (seção 12).
4. microSD corrompido (seção 14).
5. Um NVMe com problema de inicialização PCIe. Há um relato de "rk-pcie: invalid resource" em uma Max que só bootava cerca de 1 em cada 20 vezes [comunidade/Max, trecho de busca, sem solução confirmada].

**Como confirmar:**
- Teste sem nenhum cartão e sem NVMe, só com o console serial ligado. Veja até onde o boot chega.
- Teste com um microSD de imagem conhecida e gravada de forma limpa.
- Se houver NVMe, remova-o e teste de novo.

**O que fazer, do menos arriscado ao mais arriscado:**
1. Troque a fonte por uma de 5 V e 5 A (seção 1).
2. Remova o NVMe, e tente bootar só do microSD com uma imagem conhecida.
3. Regrave o microSD com uma imagem da Orange Pi para a Max e confira a soma de verificação do download.
4. Só então considere apagar a SPI (seção 13), sabendo que isso apaga o bootloader gravado nela.

## 4. Console serial para ver o erro

**Fatos confirmados (manual v1.3, p. 20 e p. 136 a 143):**
- Use um adaptador USB para TTL de **3,3 V**. O manual diz que o módulo precisa ser de 3,3 V e aceitar 1.500.000 bps.
- O manual recomenda CH340. Não use módulos baseados em CP2102 ou PL2303 (p. 136).
- Ligação (manual v1.3, p. 136, figura do módulo e da placa):
  - GND do módulo no GND da placa.
  - RX do módulo no TX da placa.
  - TX do módulo no RX da placa.
  - O pino 5V do módulo fica desligado. O jumper do módulo precisa estar encaixado.
- Configuração no terminal (manual v1.3, p. 139 e p. 142): velocidade 1500000, 8 bits de dados, 1 bit de parada, paridade None, controle de fluxo None. No Windows, o manual usa MobaXterm, com o tipo de sessão Serial e a porta do adaptador (p. 142).

**Fato da ficha oficial de produto:** a serial de debug está incluída nos 40 pinos ("UART 40PIN, 3,3 V").

**Pinos [oficial: manual v1.3, p. 136; esquemático V1.1, p. 27]:** GND no pino 6, TXD da placa no pino 8 (`UART2_TX_M0_DEBUG`) e RXD da placa no pino 10 (`UART2_RX_M0_DEBUG`). Lembre de cruzar: o RX do adaptador vai no pino 8, e o TX do adaptador vai no pino 10.

**Relatos [comunidade/Max]:**
- Wiki bunnyamin/bunnix, página orangepi_5max, editada em 06/06/2026 (https://github.com/bunnyamin/bunnix/wiki/orangepi_5max): confirma "UART 40PIN 3,3 V USB to TTL (1500000 baud)" e o comando `screen -fnL /dev/ttyUSB0 1500000`. Descreve o GND como "linha 2, terceiro pino contado da direita, primeiro preto depois dos dois vermelhos". RX do adaptador vai no TXD da placa, e TX do adaptador vai no RXD. A wiki diz "Do not use flow control (-fn)", mas `-fn` é a opção do `screen` que desliga o controle de fluxo. O texto é ambíguo. Use o comando como está escrito e confira o log.
- 05/06/2025 (thread 51994): um membro corrige o valor de 150000 para 1500000 (um zero a mais). Lembra que nem todo conector de UART aguenta essa velocidade.
- 06/06/2025 (thread 51994): um relato diz que o console serial funcionou em UART2_TX_M0 e UART2_RX_M0, e traz o log do SPL e do U-Boot. São os mesmos sinais dos pinos 8 e 10 do esquemático.

**Cuidado:** um erro de digitação na velocidade já atrapalhou usuários. Confira o valor 1500000 com todos os zeros.

**O que fazer:** capture o log inteiro. Ele mostra se o SPL e o U-Boot subiram e de qual dispositivo tentaram bootar.

## 5. HDMI sem imagem

**Fatos confirmados:** a placa tem duas saídas HDMI 2.1 (manual v1.3, p. 12 e p. 15). O manual tem um teste de HDMI na seção 3.11 (p. 197 e seguintes), mas não tem seção de solução de problemas para vídeo.

**Sintoma:** o LED acende e o sistema parece bootar, mas o monitor não mostra nada.

**Relatos [comunidade/Max]:**
- 17/05/2025 (thread 51994): com o cartão inserido, o LED azul acendeu e não apareceu nada no HDMI (ver seção 2). Sem cartão, a placa subiu pelo NVMe.
- 18/05/2025 (mesma thread): um moderador do Armbian respondeu que pouca placa tem HDMI na fase de boot, e que o log deve vir pela serial de debug. Ele não diz que a Max tenha o defeito.
- Buscas por "Orange Pi 5 Max HDMI no signal" achei relatos de Max no fórum do Armbian. Não achei solução específica da Max [trecho de busca].

**Causas prováveis:**
1. Monitor, cabo ou adaptador. Um relato de placa irmã mostrou que um monitor de 7 polegadas não dava sinal, mas um monitor de PC funcionava [placa irmã, trecho de busca].
2. Fonte insuficiente (seção 1).
3. Atualização de sistema ou kernel que deixou o HDMI sem reconhecimento. Um relato do DietPi diz que voltar à versão anterior resolveu [placa irmã: Orange Pi 5, trecho de busca].
4. Falha de inicialização de vídeo do boot. Uma nota de Raspberry Pi menciona travamento de vídeo com fonte fraca [outra placa, só como alerta].

**Como confirmar:** teste com outro monitor e outro cabo. Teste com fonte sabidamente boa. Veja no console serial (seção 4) se o boot chega ao sistema.

**O que fazer:** troque cabo e monitor, depois a fonte, e só depois mexa no software.

## 6. NVMe não detectado

**Fatos confirmados:**
- O manual lista um slot M.2 M-KEY, PCIe 3.0 x4, para SSD NVMe (manual v1.3, p. 12; figura de baixo da placa na p. 15).
- O manual pede um SSD NVMe com PCIe 3.0 x4 (manual v1.3, p. 53).

**Divergência registrada:** uma matéria da CNX fala em duas interfaces M.2. O manual v1.3 e a figura da p. 15 mostram um só slot M.2 M-Key, embaixo da placa. A mesma figura mostra, na face de cima, um conector chamado "PCIe expansion interface", que não é um slot M.2.

**Relatos [comunidade/Max]:**
- 17/05/2025 (thread 51994): com o Armbian da Orange Pi 5 Pro instalado no NVMe, a Max subiu pelo NVMe quando o cartão foi retirado. Não é confirmação de instalação nativa da imagem da Max no NVMe.
- Um erro "rk-pcie: invalid resource" em uma Max que subia só às vezes [comunidade/Max, trecho de busca, https://www.reddit.com/r/OrangePI/comments/1ki8f6x/opi_5_max_failed_to_boot_rkpcie_invalid_resource]. O mesmo resumo diz que o NVMe não foi encontrado e que a placa não bootou, e um usuário relatou que uma imagem de outro autor só subiu depois de gravar um firmware SPI de outro repositório. Sem solução confirmada.
- Relatos de travamento da Max com kernels 6.18.10 e 6.19.0 aparecem na seção 8. Não envolvem NVMe.

**Causas prováveis:**
1. O SSD não é NVMe de fato, mas SATA ou outro tipo. Relatado em placas irmãs [placa irmã: Orange Pi 5, não confirmado na Max].
2. O SSD não é compatível com o barramento PCIe da placa. Relatado em placa irmã [placa irmã].
3. Bootloader SPI antigo (seção 12).
4. Problema de inicialização PCIe, como o erro "rk-pcie: invalid resource" de um relato na Max, que bootava ocasionalmente [comunidade/Max, trecho de busca].

**Como confirmar:**
- Confirme no SSD que ele é NVMe, M-Key, e não SATA.
- Teste o SSD em outro computador ou com um SSD conhecido que funcione.
- Veja no console serial se aparece erro de PCIe.

**O que fazer:** primeiro teste um SSD NVMe conhecido e bom. Depois atualize o bootloader pela imagem oficial, se houver.

## 7. Wi-Fi AP6611

**Fatos confirmados:**
- O módulo Wi-Fi 6E e Bluetooth 5.3 é o AP6611 (manual v1.3, p. 12). A interface Wi-Fi é SDIO 3.0 e a interface Bluetooth é UART/PCM. A flash QSPI NOR de 16 MB também está listada na p. 12.
- O identificador do chip é 0xaabf (wiki da comunidade, citada abaixo).
- No esquemático V1.1 (p. 6), o Wi-Fi está ligado ao barramento SDIO (SDIO_D0 a D3, CMD, CLK, com sufixo M0_WIFI) e ao pino WIFI_REG_ON. O Bluetooth vai pela UART7 (UART7_TX/RX_M0_BT, UART7_RTS/CTS) e pelo BT_REG_ON [oficial: esquemático V1.1, p. 6].

**Relatos de Wi-Fi [comunidade/Max, em ordem de data]:**

| Data | Fonte | Kernel ou imagem | O que aconteceu |
|---|---|---|---|
| 05/01/2025 | Thread do DietPi, teste da imagem da Max (https://dietpi.com/forum/t/orange-pi-5-max-test-image-feedback/22083) | DietPi Bookworm, Bullseye e Trixie | Wi-Fi "Not Found" no dietpi-config. Com a imagem Ubuntu Joshua Riek (6.1.0-1025-rockchip), o Wi-Fi funcionava. O pacote `bcmdhd-sdio-dkms` não instalou no kernel 6.1.75-vendor-rk35xx (faltavam os headers). |
| 06/01/2025 | Mesma thread, resposta do mantenedor do DietPi | Kernel novo publicado no APT | O driver vinha compilado só para PCIe, não para SDIO. Foi publicado um kernel próprio para a Max. |
| 08/01/2025 | Mesma thread | Kernel do DietPi, após atualização e em instalação nova | Wi-Fi funcionando. |
| 07/01/2025 e 12/01/2025 | Thread 49131 (https://forum.armbian.com/topic/49131-orange-pi-5-max-builds/) | Armbian 6.12 e Armbian vendor 6.1 | No 6.12, o `bcmdhd-dkms` do Armbian não compilava. Wi-Fi não ativo de fábrica. No vendor 6.1 do Armbian, o Wi-Fi existia, mas o sistema travava no `lsmod` e havia erros PCIe. O Ubuntu Rockchip 6.1 vendor funcionou. |
| 19/04/2025 e 21/09/2025 | Thread 51307 (https://forum.armbian.com/topic/51307-orange-pi-5-ultra/) | **Placa Ultra**, com a DTB da Max e a imagem Armbian Linux 6.1 vendor da Max | Wi-Fi funcionou. Bluetooth funcionou com `rfkill` e `bluez`. Em 21/09/2025, outro usuário confirmou que a mesma imagem subiu na Ultra sem mudanças, mas não testou Wi-Fi. |
| 26/04/2025 (mesma thread) | Mesma thread | Mainline 6.12 e 6.15 na Ultra | Relatado que o chip Wi-Fi não tinha suporte. Relato sobre a Ultra, não sobre a Max. |
| 06/06/2026 (edição da wiki) | Wiki bunnyamin/bunnix (https://github.com/bunnyamin/bunnix/wiki/orangepi_5max) | Arch Linux ARM com DTB copiada da imagem Orange Pi, kernel 6.15 | O texto diz que o sistema sobe com Wi-Fi e Bluetooth funcionando. O Bluetooth aparece como "Could not make it work (kernel 6.15)". |

**Bluetooth:** um patch do linux-rockchip para habilitar o Bluetooth do AP6611s na UART7 da Max e da Ultra aparece em buscas de abril e maio de 2025 [trecho de busca, https://lists.infradead.org/pipermail/linux-rockchip/2025-May/057891.html]. Não abri o patch nem o commit. Ainda não sei em qual kernel ele entrou. A wiki diz que o Bluetooth não funcionou no 6.15.

**Driver antigo:** a wiki diz que o bcmdhd antigo não reconhece o 0xaabf e dá "dhdsdio_probe_attach: unsupported chip: 0xaabf". O texto sugere `CONFIG_BCMDHD=n` [comunidade/Max, wiki, lida em 03/11/2026]. Não testei o efeito dessa opção.

**Passo de Wi-Fi da wiki que não se aplica:** a wiki também copia `rtw88` do repositório lwfinger para o kernel 5.10.110. O rtw88 é o driver de chips Realtek, não do AP6611 (Broadcom). O texto não explica por que o passo aparece. Não use esse passo para o AP6611.

**Firmware:** o arquivo de configuração do Armbian para a Max usa o Bluetooth com o firmware `SYN43711A0.hcd` [placa do Armbian: arquivo de configuração da Max, aberto em https://raw.githubusercontent.com/armbian/build/main/config/boards/orangepi5-max.csc]. Não encontrei a lista oficial de firmwares do AP6611 para a Max. Não abri o repositório `orangepi-xunlong/firmware` [não confirmado].

**Como confirmar:**
- `cat /sys/bus/mmc/devices/mmc*/device` deve mostrar `0xaabf` (wiki).
- Veja no `dmesg` se o driver reclama de chip não suportado.

**O que fazer:** use a imagem que traz o driver para a Max. Os relatos que deram Wi-Fi foram: a imagem do DietPi com o kernel próprio da Max (atualizada em janeiro de 2025), a imagem Armbian vendor 6.1 da Max, e o Arch da wiki com kernel 6.15. Se usar kernel mainline, confirme o Wi-Fi no `dmesg`. Os relatos de mainline são da Ultra, não da Max.

## 8. Travamentos sob carga

**Sintoma:** o sistema reinicia ou trava sob carga pesada de disco ou CPU.

**Relato encontrado, sobre a Max:** três nós de um cluster Kubernetes com Orange Pi 5 Max reiniciavam de forma espontânea sob muita escrita em disco. Os reinícios não deixavam registro em log. Ocorreu em kernels 6.19.0 e 6.18.10, mas não no 6.18.9. O armazenamento era btrfs sobre volumes Longhorn com iSCSI. **O relato não menciona NVMe nem travamento, só reinício.** Um nó registrou um erro no btrfs sem reiniciar. Uma pessoa da lista do btrfs sugeriu um bit de memória travado, e o teste em espaço de usuário não achou erro [comunidade/Max, lista linux-rockchip, fevereiro de 2026].

**Divergência:** o usuário acha pouco provável falha de hardware. Um colaborador respondeu que um sintoma parecido apareceu em uma máquina virtual x86. **Não há conclusão.**

**Relato do Armbian sobre o vendor 6.1 (49131, 12/01/2025):** no vendor 6.1 do Armbian, o sistema trava ao rodar `lsmod` e aparecem erros PCIe no log. O autor supôs um problema no arquivo DTS. Um mantenedor respondeu que o DTS pode estar certo e que a diferença pode ser do RK3588 SDK (rkr4.1). Sem conclusão. [comunidade/Max]

**Causas prováveis:**
1. Um kernel específico com bug (o relato sugere 6.18.10 e 6.19.0 como instáveis).
2. Fonte instável sob carga (seção 1).
3. Problema de memória (o teste em espaço de usuário não é conclusivo).
4. Um NVMe ou um cabo com problema de conexão (não confirmado para a Max).
5. Erros PCIe no kernel vendor 6.1 do Armbian (relato de 12/01/2025, sem solução).

**O que fazer, do menos arriscado:** troque a fonte e confira a temperatura; depois teste um kernel anterior citado como estável no relato; por último, troque o NVMe ou a memória. Não encontrei relato de travamento sob carga ligado ao NVMe da Max.

## 9. Modo MaskROM e botões

**Fatos confirmados (manual v1.3, p. 12, p. 13, p. 15, p. 57):**
- A placa tem um botão **MaskROM** e um botão **POWER** separados. No manual, o botão MaskROM aparece como "MaskROM Key" (p. 13 e p. 15).
- Na serigrafia da própria placa, ao lado do botão MaskROM, está escrito **BOOT**. Ou seja, BOOT e MaskROM são o mesmo botão físico. Veja a figura da p. 15 e a figura do conector de 40 pinos (p. 223).
- O texto do manual chama o botão de MaskROM. Quando alguém disser "botão BOOT", é o mesmo botão.

**Esquema [oficial: esquemático V1.1, p. 28]:** o botão MASKROM/BOOT liga o pino BOOT_SARADC_IN0 ao GND por um resistor de 100 Ω (R114). O esquema também tem o botão POWER_Key.

**Tabela de boot [oficial: esquemático V1.1, p. 6]:** a tabela BOOT MODE CONFIG diz que, com 0 V no pino, o modo é "USB (Maskrom mode)". Isso explica o procedimento do manual: com o botão pressionado, o pino vai a 0 V e a placa entra em MaskROM.

**Procedimento (manual v1.3, p. 56 a 57, cap. 2.6.1, passos 11a a 11e):**
1. Placa desligada, sem fonte e sem cartão TF.
2. Conecte o computador à placa com um cabo **USB-A macho-macho**. A porta de gravação da placa é a USB-A azul de cima, na pilha USB 3.0 (figura da p. 57). O texto da p. 57 diz "male to female", mas as p. 20 e 54 dizem "male to male", e só o macho-macho encaixa em duas portas USB-A fêmea.
3. Segure o botão MaskROM.
4. Conecte a alimentação Type-C e ligue a placa. Solte o MaskROM.
5. O RKDevTool mostra "Found a MASKROM device" (p. 57, passo e). Se não aparecer, o sistema não entrou em modo MaskROM.

**Relato de comunidade [trecho de busca]:** a página de releases do projeto fiebrandt/edk2-rk3588 (https://github.com/fiebrandt/edk2-rk3588/releases) diz para segurar o MaskROM ao ligar, e que `lsusb` deve mostrar o ID 2207:350b. É um projeto de RK3588 em geral, não confirmei se cobre a Max. Não abri a página.

**Ponto de cautela:** o manual não fala em apertar o botão brevemente só com o cabo USB conectado. Um relato de comunidade diz que isso funciona [comunidade/Max, trecho de busca, não confirmado]. Não use esse atalho sem confirmação.

**Cuidado com o relato de boot pelo botão:** um relato de 30/11/2025 (thread 51994) diz que segurar o botão "boot" por 5 s ao ligar, com o cartão inserido, faz a placa subir pelo cartão. Isso não bate com a tabela (0 V = MaskROM). Não consegui reconciliar os dois. Veja a seção 12.

**Como confirmar:** use o RKDevTool para ver se aparece "Found a MASKROM device". Não encontrei comando de `ld` equivalente que o manual cite para a Max.

## 10. Ferramentas de gravação: RKDevTool, rkdeveloptool e upgrade_tool

**Fatos confirmados (manual v1.3, p. 54 a 62 e p. 130 a 134):**
- O manual usa o RKDevTool v3.15 e o driver DriverAssitant v5.12 (Windows). Os dois são baixados da pasta de ferramentas oficiais da Orange Pi.
- O RKDevTool carrega o loader e grava por endereço.
- O manual não menciona `rkdeveloptool` nem `upgrade_tool`. **Não confirmado** que essas ferramentas funcionem na Max.

**Arquivos de loader (manual v1.3, pasta MiniLoader, p. 54 a 62 e p. 132):**
- `MiniLoaderAll.bin`: carregado no RKDevTool em p. 60 (passo l), na gravação pelo SPI+NVMe, e em p. 132 (passo h), no apagamento da SPI.
- `rkspi_loader.img`: selecionado em p. 60 (passo n), na gravação pelo SPI+NVMe.
- `rk3588_linux_pcie.cfg`: arquivo de configuração importado em p. 59 (passo i).
- A gravação exige marcar "force write by address" (p. 62, passo q).

**Comandos de terminal (`rkdeveloptool`):** o manual v1.3 não cita nenhum deles. Há relatos, todos de trecho de busca:
- Projeto edk2-rk3588 (https://github.com/fiebrandt/edk2-rk3588/releases): carregar o loader com `db`, escolher a SPI NOR com `cs 9`, gravar com `wl 0`, conferir lendo com `rl` e comparando SHA-256, e reiniciar com `rd`. Não é relato da Max.
- README de imagem HardenedBSD para `orange_pi_5_max` (https://dk-01.installer.hardenedbsd.org/irradium/images/orange_pi_5_max/README.TXT): entrar em MaskROM, `db` com um loader, `wl 64` com um idbloader e `rd`. Usa outro loader e outro offset.
- Relato no Reddit (https://www.reddit.com/r/OrangePI/): um dono de Max disse que o rkDevTool no Windows e o rkdeveloptool no Ubuntu deixaram de reconhecer o dispositivo em MaskROM depois de um problema com a placa. Sem solução.
- Relatos de placa irmã (Orange Pi 5, trecho de busca): erro "Creating Comm Object failed" no rkdeveloptool depois de entrar em MaskROM. Um usuário suspeitou de chip SPI ruim.

**Divergência:** o offset de gravação não coincide. Uma fonte grava em setor 0 e a outra em `wl 64`. Depende do loader e da imagem.

**Status:** `rkdeveloptool` na Max tem relato, mas nenhum relato confirmado de sucesso. A documentação da Rockchip não abriu nesta revisão.

**Atenção, em negrito: os comandos `wl` escrevem na flash e apagam o conteúdo daquele endereço. Confira o loader, o offset e o tamanho antes de rodar.**

## 11. Gravação de imagem SPI+NVMe (resumo)

O manual descreve uma gravação em que o bootloader vai para a SPI e o sistema vai para o NVMe. O passo a passo está em p. 53 a 62:
- Use o NVMe com PCIe 3.0 x4 (p. 53).
- Instale o driver DriverAssitant v5.12 e o RKDevTool v3.15 (p. 54 a 56).
- Entre em MaskROM como na seção 9.
- Importe `rk3588_linux_pcie.cfg`, carregue `MiniLoaderAll.bin` e `rkspi_loader.img`, e escolha a imagem Linux (p. 59 a 61).
- Marque "force write by address" e execute (p. 62).

Se a gravação der erro, o manual manda apagar a SPI antes e tentar de novo (p. 62). Veja a seção 13.

## 12. Ordem de boot e como a SPI interfere

**Sintoma:** a placa parece ignorar o cartão ou o NVMe, ou sempre sobe a mesma imagem antiga.

**Ordem da BootROM (primeiro estágio, dentro do chip) [oficial: esquemático V1.1, p. 6, tabela BOOT MODE CONFIG]:** a tabela lista a ordem FSPI M2, FSPI M1, FSPI M0, eMMC, SD Card e USB. O NVMe não aparece na lista. Por isso, o NVMe só sobe com o bootloader gravado na SPI.

**Pino de configuração [oficial: esquemático V1.1, p. 6]:** o resistor superior R170 é de 100 kΩ, e o inferior R171 não é montado (DNP), então o pino fica em 1,8 V. Na tabela, cada linha combina Rup, Rdown, valor do ADC e tensão: 682 equivale a 0,3 V, e 4095 a 1,8 V, o nível 7, que é a ordem acima. A leitura vem do texto do PDF.

**O que vem depois da BootROM (U-Boot e SPL):** esta é outra etapa. Não está no esquema nem no manual v1.3. Os relatos abaixo tratam dela:
- 17/05/2025 (thread 51994): com o cartão inserido, a placa não subia. Sem cartão, subia pelo NVMe.
- 26/05/2025 e 06/06/2025 (mesma thread): logs do SPL com "Trying to boot from MMC1" e "Trying to boot from MMC2". Não há mapeamento confirmado de MMC1 e MMC2 para cartão, eMMC ou outro dispositivo.
- 30/11/2025 (mesma thread): o autor relatou que segurar o botão "boot" por 5 s ao ligar, com o cartão inserido, fez a placa subir pelo cartão. Demorou até 60 s. Pelo mesmo relato, o boot por USB também funcionou, mas só numa porta USB 2.0 [comunidade/Max, não confirmado; não bate com a tabela da BootROM, ver seção 9].
- 09/12/2025 (mesma thread, placa irmã Orange Pi 5 Plus): o SPL não detectou o cartão em boot frio, e o log mostra "MMC1", "MMC2" e "MTD2". Não é a Max [placa irmã].
- Um relato de Rockchip genérico descreve SPI NOR, depois SPI NAND, eMMC e SD [RK3588, trecho de busca]. É coerente com a ordem do esquema da Max.
- A wiki da comunidade (https://github.com/bunnyamin/bunnix/wiki/orangepi_5max) diz que o U-Boot espera uma tabela GPT no cartão. Ver seção 14 para os comandos de gravação.

**SPI na Max:** a flash QSPI NOR de 16 MB é confirmada pelo manual (p. 12 e p. 15).

**Como a SPI interfere:** um relato em trecho de busca diz que a placa pode ficar presa num U-Boot e num Linux pequenos gravados na SPI, até a SPI ser apagada [comunidade, não confirmado na Max]. Um relato de fórum DietPi (placa irmã) diz que um bootloader SPI desatualizado impediu o boot por NVMe, e que atualizar resolveu [placa irmã: Orange Pi 5, não confirmado na Max].

**Como confirmar:**
- O console serial mostra o U-Boot ou o SPL sendo carregado a partir de qual dispositivo? Procure a linha de tentativa de MMC ou de NVMe.
- Se a placa sobe sempre a mesma imagem independentemente do cartão, a SPI é suspeita.

**O que fazer:** atualize o bootloader pela imagem oficial da Orange Pi, se houver. Apagar a SPI é o último recurso (seção 13).

## 13. Como apagar a SPI

**Atenção, em negrito: apagar a SPI remove o bootloader gravado nela. Sem um bootloader bom para regravar, a placa pode deixar de bootar pelo cartão. Isso é destrutivo.**

**Quando considerar:** só depois de tentar as seções 1 a 3, e quando você confirmou que o bootloader da SPI é o problema.

**Procedimento do manual (cap. 2.14, manual v1.3, p. 128 a 134):**
1. Localize a SPI Flash na figura da p. 128.
2. Instale o driver DriverAssitant v5.12 e abra o RKDevTool v3.15 (p. 129 a 130). O cabo do passo 7 é descrito como USB 2.0 macho-macho.
3. Conecte o computador à placa com o cabo USB-A macho-macho, na mesma porta da seção 9. O texto do passo 8a (p. 131) diz "Type-C cable", mas a figura da mesma página é a da porta USB-A (igual à da p. 57), e a p. 130 pede cabo macho-macho.
4. Sem cartão TF e sem alimentação (p. 131, passo 8b).
5. Segure o botão MaskROM, conecte a alimentação Type-C e solte (p. 131 a 132, passos 8c a 8e).
6. Em "Advanced Features", selecione `MiniLoaderAll.bin` e clique em Download (p. 132 a 133, passos 8f a 8i).
7. Escolha o armazenamento como SPINOR, clique em Switch Storage e em **Erase All**, que apaga a SPI (p. 133 a 134, passos 8k a 8m).
8. Depois do apagamento, reinicie a placa e confira o boot pelo console serial.

Após apagar, o manual não diz como regravar um bootloader bom na SPI. Não encontrei esse procedimento para a Max [não confirmado].

**Relatos de UEFI na Max [comunidade/Max, thread 61755, https://forum.armbian.com/topic/61755-seeking-latest-uefi-for-orange-pi-5-max/, sem ano visível na página]:**
- Um usuário com UEFI antigo, herdado de uma distro abandonada, pediu uma versão mantida (27/08/2025).
- Um moderador do Armbian respondeu que não há imagem que combine device tree do Armbian com UEFI, e que o UEFI deve vir do projeto edk2 de RK3588 (28/08/2025).
- Um membro disse que o projeto edk2-porting/edk2-rockchip não tem a Max nativamente (28/08/2025).
- Outro usuário achou o projeto amazingfate/edk2-rk35xx, que tem imagem para a Max, mas sem atualização desde 2025 (29/08/2025). Não reabri a página do GitHub.
- Um membro afirmou que o Armbian 26.8 pode gravar o bootloader na SPI pelo armbian-config. Não confirmado; é comentário de membro.

**Atenção, em negrito: gravar UEFI ou bootloader na SPI sobrescreve o que já está lá. Se a gravação falhar, a placa pode ficar sem boot pela SPI.**

## 14. microSD corrompido

**Sintoma:** boot que para no U-Boot ou no kernel, mensagens de erro de leitura do cartão, ou sistema que sobe e depois apresenta erros de disco.

**Causa provável:** cartão com setores danificados ou gravação incompleta. Esse é um problema geral de cartões. O manual pede cartão TF de pelo menos 16 GB, classe 10 ou superior, e recomenda 32 GB (p. 17 e p. 21).

**Layout de partições descrito na wiki da comunidade [comunidade/Max, https://github.com/bunnyamin/bunnix/wiki/orangepi_5max; a própria wiki diz que o texto foi escrito em 2025-06-07, sem dizer se foi testado; não testei]:** a tabela GPT tem as partições loader1 (início em 0x40), loader2, trust, boot e rootfs. O U-Boot é gravado com `dd` no início do cartão: `idbloader.img` com `seek=64` e `u-boot.itb` com `seek=16384`. A wiki manda copiar esses arquivos de uma imagem Orange Pi que funcione.

**Atenção, em negrito: `dd if=/dev/zero of=/dev/sd?` apaga os primeiros 20 MB do dispositivo, e `dd ... of=/dev/sd?` sobrescreve o início dele. Troque `sd?` pelo cartão certo antes de rodar. Um destino errado apaga o disco do sistema.**

**Relato de placa irmã:** no Orange Pi 5 Plus, o SPL não detectou alguns cartões novos em boot frio. O autor associou isso a um atraso de 1 ms entre alimentar o cartão e o primeiro comando, citando um artigo de outro SoC (NXP i.MX 93) [placa irmã, thread 51994, 09/12/2025 a 11/12/2025]. Pode explicar falhas de cartão específico na Max, mas não está confirmado.

**Como confirmar:**
- Regrave a imagem com um leitor e uma ferramenta de gravação conhecidos, e confira a soma de verificação do arquivo.
- Teste o cartão em outro computador e veja se há erros de leitura.
- Use outro cartão de marca conhecida e de capacidade adequada.

**O que fazer:** regrave a imagem em outro cartão. Se o problema seguir, o cartão está ruim ou a placa não lê aquele modelo.

## 15. Ventoinha PWM

**Fatos confirmados [oficial: esquemático V1.1, p. 11]:** o conector da ventoinha é J3, de 2 pinos de 1,25 mm. O pino 1 é + (5 V) e o pino 2 é − (chaveado por MOSFET, com FAN_CTL_H).

**Relatos [comunidade/Max]:**
- 11/03/2025 (thread 50442, https://forum.armbian.com/topic/50442-pwm-fan-connector-not-working-on-orange-pi-5-max): com a imagem Armbian community 25.5.0-trunk.185 (bookworm, kernel 6.12.17), a ventoinha no conector de 1,25 mm não girou em nenhuma carga. Com a imagem Orange Pi Ubuntu jammy (kernel 5.10.160), a mesma ventoinha girou sob carga.
- Data relativa na página, por volta de 30/10/2026 (mesma thread): com o kernel 6.1.115-vendor-rk35xx a ventoinha gira, mas esse kernel não tem aceleração OpenGL. Com a imagem Debian Trixie da Orange Pi, a ventoinha gira no boot e para.
- 4 de junho, sem ano visível (thread 60076, https://forum.armbian.com/topic/60076-pwm-fan-configuration-on-orange-pi-5-max-running-armbian-debian-minimal/): uma ventoinha 40x40x10 de 5 V, 4 fios, foi ligada ao cabeçalho de 40 pinos, não ao conector de 1,25 mm. Fios: vermelho em +5 V (pino 4), preto em GND (pino 6), azul em PWM (pino 7, GPIO PWM3_IR_M3), amarelo do tacômetro sem ligação. O relato ativou o overlay `rk3588-pwm3-m3` com o `armbian-config` e usou o opifancontrol. Com vendor 6.1.115 e Debian Trixie minimal, a temperatura caiu de cerca de 70 °C para 48 °C.

**Não confirmado:** por que o conector de 1,25 mm não gira no kernel 6.12 do Armbian. Os relatos mostram que o kernel e a imagem mudam o resultado.

**O que fazer:** se a ventoinha do conector de 1,25 mm não girar, teste com o kernel vendor 6.1 ou com a imagem Orange Pi. Se for usar o cabeçalho de 40 pinos, use o overlay `rk3588-pwm3-m3`, e confira antes os pinos no seu conector (seção 4).

## 16. Divergências e lacunas registradas

- **Botão BOOT:** resolvido. BOOT é a serigrafia do mesmo botão MaskROM. Fontes: manual v1.3, p. 15 e figura do conector (p. 223); esquemático V1.1, p. 28.
- **Cabo para gravação:** resolvido. É USB-A macho-macho, na porta USB-A azul de cima (figura da p. 57). O "male to female" da p. 57 não encaixa nos dois lados.
- **Cabo para apagar a SPI:** resolvido. É o mesmo cabo e a mesma porta. O "Type-C" da p. 131 contradiz a figura da própria página.
- **Fonte:** 5 V/5 A (manual p. 13 e p. 18; ficha oficial) contra 5 V/4 A (manual p. 135 e p. 144) e "DC: 5V 4A" no esquema (p. 14). Use pelo menos 5 V/5 A.
- **NVMe:** um slot M-KEY segundo o manual (p. 12 e p. 15), contra duas interfaces M.2 segundo a CNX (comunidade). Não abri a CNX.
- **Serial:** resolvido. Pinos 6 (GND), 8 (TXD) e 10 (RXD), segundo o manual (p. 136) e o esquemático V1.1 (p. 27).
- **Ordem de boot:** a BootROM tem ordem oficial pelo esquema V1.1 (p. 6). O NVMe não está na lista. A ordem depois da BootROM (U-Boot) não tem fonte oficial, e o relato do botão "boot" (30/11/2025) não bate com a tabela (0 V = MaskROM).
- **Ferramentas de linha de comando:** `rkdeveloptool` e `upgrade_tool` não estão no manual v1.3. Há relatos de trecho de busca, com offsets diferentes entre si. Não confirmados para a Max.
- **Crash sob carga:** o relato da lista linux-rockchip é de reinício, não de travamento, e não envolve NVMe. A causa não ficou clara. Há um relato do Armbian sobre travamento no `lsmod` com erros PCIe (12/01/2025), sem conclusão.
- **Wi-Fi:** resolvido em parte. Funcionou com o kernel próprio do DietPi (jan/2025), com a imagem Armbian vendor 6.1 da Max, e com o Arch da wiki (kernel 6.15). O relato de mainline sem Wi-Fi é da Ultra. Bluetooth com kernel 6.15 não funcionou segundo a wiki, e o status do patch da UART7 não foi confirmado.
- **Ventoinha:** o conector de 1,25 mm não girou na imagem Armbian 6.12 (março de 2025). A causa não ficou clara.
- **Onde procurei sem resposta:** significado de cada cor do LED em cada etapa (threads do Armbian, patches do linux-rockchip, fórum da Orange Pi); causa de HDMI sem imagem (threads do Armbian e do DietPi, buscas gerais); NVMe (busca e espelho do Reddit). Nenhuma fonte deu resposta oficial.
- **Fontes que não consegui abrir:** wiki da Rockchip (DNS não resolveu na revisão anterior), Web Archive (recusado), CNX, página de releases do edk2-rk3588, README da HardenedBSD, espelho do Reddit e patches do linux-rockchip (só trechos de busca).

## Fontes

- https://raw.githubusercontent.com/u-boot/u-boot/master/doc/board/rockchip/rockchip.rst
- https://raw.githubusercontent.com/armbian/build/main/config/boards/orangepi5-max.csc
- https://forum.armbian.com/topic/49131-orange-pi-5-max-builds/
- https://forum.armbian.com/topic/51994-orange-pi-5-max-sd-card-booting-issue-system-normally-starts-from-nvme-ssd/
- https://forum.armbian.com/topic/44637-orange-pi-5-max-debian/
- https://forum.armbian.com/topic/50442-pwm-fan-connector-not-working-on-orange-pi-5-max/
- https://forum.armbian.com/topic/60076-pwm-fan-configuration-on-orange-pi-5-max-running-armbian-debian-minimal/
- https://forum.armbian.com/topic/51307-orange-pi-5-ultra/
- https://forum.armbian.com/topic/61755-seeking-latest-uefi-for-orange-pi-5-max/
- https://github.com/bunnyamin/bunnix/wiki/orangepi_5max
- https://dietpi.com/forum/t/orange-pi-5-max-test-image-feedback/22083
- https://drive.google.com/file/d/1cAKUBHF2yXtmmLJaVtK17nOKoMlI02wA/view (esquemático V1.1 da Max; lido pelo texto extraído do PDF, folhas 6, 11, 14, 27 e 28)
