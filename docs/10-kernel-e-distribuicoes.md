# Orange Pi 5 Max: kernel, suporte do RK3588 e distribuições

Data da coleta: 03/11/2026.

Este documento reúne o que foi possível confirmar sobre o kernel Linux e as distribuições disponíveis para a Orange Pi 5 Max (placa com SoC Rockchip RK3588). Ele foi escrito para se sustentar sozinho. Onde uma fonte fala de outra placa, isso está marcado.

## Legenda das marcações

- **[oficial]**: fonte primária. Pode ser o fabricante (manual v1.3, site da Orange Pi e pastas públicas do Google Drive que ele indica) ou o código e a documentação do projeto (DTS e drivers do kernel Linux, configuração e regras do Armbian, repositórios `orangepi-xunlong`).
- **[comunidade/Max]**: relato, catálogo ou matéria de terceiros, referente à Orange Pi 5 Max.
- **[placa irmã: nome]**: informação de outra placa. Não vale automaticamente para a Max.
- **[não confirmado]**: não achei fonte confiável que responda. Onde a busca foi feita, o texto diz onde.

Suporte de kernel muda rápido. Sempre que a data importar, ela está indicada.

---

## 1. Resumo rápido

- **Imagens oficiais.** A pasta de Ubuntu e a de Debian, no Google Drive indicado pela Orange Pi, mostram imagens de versão 1.0.2 com kernel 6.1.99 (Ubuntu 22.04 e Debian 11 e 12) e 6.18.49 (Ubuntu 26.04 "resolute" e Debian 13 "trixie", só desktop GNOME). As datas de modificação exibidas são 15/09/2026 e 29/09/2026. O manual v1.3 (2024) ainda lista kernels 5.10.160 e 6.1.43, que são versões anteriores. [oficial, Google Drive oficial, lido em 03/11/2026]
- **Mainline.** A DTS da Max está no kernel principal desde a v6.14-rc1. O arquivo `rk3588-orangepi-5-max.dts` habilita HDMI (com áudio), NVMe, a porta de 2,5 GbE, a ventoinha PWM, o Bluetooth por UART7, a GPU e o NPU. Não descreve o Wi-Fi do AP6611. [oficial, código-fonte do kernel]
- **Em placa real, no mainline, não achei nenhum relato de 2026 confirmando HDMI, Panthor, NPU, 2,5 GbE ou decodificador na Max.** Os relatos lidos são de 2024 e 2025, e o mais recente sobre o mainline mostra erro de DTB (seção 4.1). [não confirmado]
- **Wi-Fi.** O mantenedor do DietPi disse, em setembro de 2026, que o driver `brcmfmac` do mainline ainda não suporta o AP6611S. O build oficial da Orange Pi também bloqueia `brcmfmac` na configuração da Max para o kernel `next` (6.18). Por isso a Max continua, na prática, no kernel de fornecedor. [comunidade/Max, DietPi, 13/09/2026; oficial, orangepi-build]
- **Armbian.** A Max aparece como **Community** (não "Supported"). A configuração da placa no repositório do Armbian não tem mantenedor nomeado, e as regras do projeto definem esse nível assim. Kernels listados na página de 03/11/2026: vendor 6.1.172 e edge 7.3.1. [oficial, armbian/build e docs.armbian.com; comunidade/Max, catálogo]
- **Servidor sem monitor.** Não há uma escolha sem ressalva. Na seção 7 está a recomendação revisada.

---

## 2. Kernel: BSP da Rockchip (5.10 / 6.1) versus mainline

**BSP (Board Support Package) ou kernel de fornecedor.** É um kernel mantido pela Rockchip e adaptado pelos fabricantes de placa. Tem drivers prontos para a maioria dos periféricos do RK3588, inclusive os que ainda não chegaram ao upstream. O custo é que ele é uma bifurcação antiga: recebe pouca atualização de segurança, não segue o kernel principal e costuma ter um conjunto próprio de device trees.

Os três ramos de kernel que o `orangepi-build` usa para a Max são, na ordem do nome interno do build (`legacy`, `current`, `next`) e na ordem do que cada um é:

- **`legacy` = linha 5.10.** Ramo `orange-pi-5.10-rk35xx`, kernel 5.10.160. Último commit em 07/04/2025. [oficial, orangepi-build e linux-orangepi, lido em 03/11/2026]
- **`current` = linha 6.1 (Rockchip BSP).** Ramo `orange-pi-6.1-rk35xx`, kernel 6.1.99. Último commit em 20/08/2026. É o kernel das imagens oficiais de Ubuntu 22.04 e Debian 11 e 12 de setembro de 2026. [oficial, orangepi-build, linux-orangepi e Google Drive]
- **`next` = linha 6.18 (Orange Pi, não kernel.org).** Ramo `orange-pi-6.18-rk35xx`, kernel 6.18.49. Último commit em 28/09/2026. É o kernel das imagens oficiais de Ubuntu 26.04 e Debian 13 de setembro de 2026. Apesar de ser versão nova, é uma árvore da Orange Pi, não o mainline de kernel.org. [oficial, orangepi-build e linux-orangepi, lido em 03/11/2026]
- Na família RK3588 do build, o `next` usa o driver `hdmirx` do mainline só para **entrada** HDMI (script de pré-visualização). Isso não diz nada sobre a saída HDMI. [oficial, orangepi-build, `rockchip-rk3588.conf`]

**Mainline (kernel principal, kernel.org).** É o kernel que segue o upstream. Tem menos drivers prontos para o RK3588, mas recebe atualização de segurança normalmente e é o caminho que as distribuições querem a longo prazo.

- O Armbian "edge" segue o mainline e lista **edge 7.3.1** na página da Max, em 03/11/2026. [comunidade/Max, catálogo Armbian]. Uma busca também mostrou snapshots com edge 7.2.2 e 7.3.0 em datas anteriores. Esses valores divergem entre páginas do mesmo catálogo, porque são fotos de datas diferentes. Vale a página do dia da leitura.
- Um mantenedor de distribuição (DietPi) escreveu que, em setembro de 2026, placas Orange Pi 5 Plus já migravam para mainline 6.18, mas a Max e a Ultra continuavam no fornecedor por causa do Wi-Fi. [comunidade/Max, DietPi, 13/09/2026]
- Um usuário do fórum Armbian disse que, para RK3588 no mainline, o suporte "estava apenas começando" nas versões antigas e recomendou o edge com 6.18 ou mais novo. [comunidade, fórum Armbian, data não lida]. É opinião, não teste.
- O mantenedor do Armbian, em janeiro de 2025 (tópico 49131), disse que o suporte básico ao kernel da Max deveria estar no vendor e no mainline com o DTB de 6.14, e que faltavam os drivers de Wi-Fi e ajustes de DTS. [comunidade/Max, fórum Armbian, 06/01/2025 e 12/01/2025]

**Escolha prática.** Use o kernel de fornecedor quando precisar de algum recurso que o mainline ainda não cobre bem na Max (Wi-Fi do AP6611S, sobretudo). Use o mainline quando o foco for segurança e atualização do sistema e os recursos que você usa já funcionam nele. O código mostra que o mainline já descreve quase todo o hardware da Max; o que falta confirmar é o funcionamento real em cada periférico (seção 4).

---

## 3. Suporte ao RK3588 no kernel Linux (geral, não específico da Max)

Estas informações são da placa-geral RK3588. Quando o item foi verificado no código para a Max, a seção 4 diz.

- **PCIe.** O controlador DesignWare é antigo no mainline, mas a variante do RK3588 demorou para ser habilitada. Uma rede PCIe de 2,5 GbE foi habilitada na 6.7, na Radxa ROCK 5B. [placa irmã: Radxa ROCK 5B; Collabora, dezembro de 2024, lido na coleta anterior]
- **HDMI.** A PHY de HDMI entrou na 6.9. O controlador HDMI 2.1 (`dw_hdmi_qp-rockchip`) está no tag v6.13, verificado em 03/11/2026. [Collabora, dezembro de 2024; código-fonte do kernel]
- **GPU Mali-G610 (Panthor).** A aceleração 3D entrou na 6.10. O diretório `drivers/gpu/drm/panthor` está no tag v6.10 e no master. [Collabora, dezembro de 2024; código-fonte do kernel]. Não verifiquei desempenho nem OpenCL atuais.
- **USB.** A primeira controladora USB3 foi habilitada na 6.8. Todas as portas usaram a PHY USBDP na 6.10. [Collabora, dezembro de 2024, lido na coleta anterior]
- **Ethernet nativa.** Foi mergeada antes, mas a fonte diz que fica limitada a 1 Gbit. Placas que têm 2,5 GbE costumam usar um chip ligado por PCIe ou USB. [Collabora, dezembro de 2024, lido na coleta anterior]
- **Decodificação de vídeo.** VDPU121 (VP8, MPEG-2, H.264) entrou na 6.12. A decodificação H.264 e H.265 pelo VDPU381 está no tag **v7.0**: os arquivos `rkvdec-vdpu381-h264.c` e `rkvdec-vdpu381-hevc.c` aparecem no v7.0 e não aparecem no v6.19. [oficial, código-fonte do kernel, verificado em 03/11/2026]. Multi-core, VP9 e AV1 (no RK3576) ainda eram trabalho futuro na data do artigo de fevereiro de 2026. [cnx-software, 27/02/2026]
- **NPU.** O driver aberto `rocket` está no tag **v6.18** (o diretório `drivers/accel/rocket` não existe no v6.17). O commit mais antigo que listei para esse diretório é de 25/07/2025 ("accel/rocket: Add registers header"). [oficial, código-fonte do kernel, verificado em 03/11/2026]. Um usuário do Armbian disse que o NPU funciona com o driver de fornecedor (v0.9.8) e que o mainline precisaria de 6.15 ou 6.18. [opinião de fórum]

---

## 4. Estado dos recursos na Orange Pi 5 Max

Especificações oficiais, segundo o manual v1.3 (p. 12-13), a página de produto e a ficha do site da Orange Pi (lidas diretamente em 03/11/2026):

- SoC RK3588 (8 nm): 4 núcleos Cortex-A76 a 2,4 GHz e 4 Cortex-A55 a 1,8 GHz.
- RAM: 4, 8 ou 16 GB LPDDR5. Armazenamento: socket eMMC (ou eMMC soldado, conforme a versão), microSD, QSPI NOR de 16 MB e um slot M.2 M-Key para NVMe (PCIe 3.0 x4). A ficha também cita SATA SSD pelo mesmo slot. [oficial]
- Rede: 1 porta 2,5 GbE com Realtek RTL8125BG (ligada por PCIe). [oficial]
- Wi-Fi 6E e Bluetooth 5.3/BLE pelo módulo AP6611. Wi-Fi por SDIO 3.0; Bluetooth por UART/PCM. [oficial]
- Vídeo: 2 saídas HDMI 2.1 (até 8K a 60 Hz) e 1 saída MIPI DSI de 4 lanes. Entradas: 2 MIPI CSI de 4 lanes e 1 D-PHY RX de 4 lanes. [oficial]
- USB: 2 USB 3.0 (uma delas pode ser device ou host) e 2 USB 2.0 host. [oficial]
- Áudio: codec ES8388, jack de 3,5 mm com entrada de microfone, HDMI eARC. [oficial]
- NPU: 6 TOPS, com suporte a INT4, INT8, INT16 e FP16. GPU: Mali-G610 (OpenGL ES 3.2, OpenCL 2.2, Vulkan 1.2). [oficial]
- Alimentação: USB-C de 5 V / 5 A. Ventoinha: conector de 2 pinos. [oficial, manual]
- Peso: a ficha oficial dá 62 g e o manual dá 58 g. Não afeta o kernel; anotado só para conferência.
- Codecs: a decodificação de 8K H.265/VP9/AVS2 e de 4K AV1 vem da matéria da CNX Software (01/08/2024), não do manual. [comunidade/Max]
- Sistemas suportados, segundo a ficha oficial: Orange Pi OS (Droid), Orange Pi OS (Arch), Orange Pi OS (OH), Ubuntu, Debian e Android 13. [oficial, ficha do site, lida em 03/11/2026]

Recurso por recurso. A coluna "Kernel e data" diz o que foi verificado e quando.

| Recurso | Estado | Kernel e data | Marcação |
|---|---|---|---|
| HDMI | A DT da Max habilita `hdmi0` e `hdmi1`, cada um com seu conector (`hdmi-connector`), e marca `frl-enable-gpios`. O controlador `dw_hdmi_qp` existe desde a v6.13. Imagem em placa com mainline: não confirmada (seção 4.1). | Mainline: DT no master (03/11/2026) e tag v7.3; driver desde v6.13 | [oficial, código-fonte do kernel]; placa: [não confirmado] |
| HDMI áudio | A DT habilita `hdmi0_sound` e `hdmi1_sound` (commit de 12/03/2025). O commit está no tag v6.15 e não está no v6.14. | Mainline: a partir da v6.15 | [oficial, código-fonte do kernel] |
| GPU (Panfrost/Panthor) | O nó `&gpu` está habilitado na DT base do Orange Pi 5, que a Max inclui. O driver para o Mali-G610 é o Panthor. O relato da Bunnix cita "vulkan-panfrost", mas o Panfrost não cobre o G610, então esse nome parece confuso. A imagem Ubuntu da comunidade (seção 5.3) usa panfork e a biblioteca `libmali` de fornecedor, não o Panthor. | DT no mainline (03/11/2026); relato sem versão de kernel | [oficial, código] para a DT; [comunidade/Max] para os relatos; funcionamento com Panthor: [não confirmado] |
| Wi-Fi (AP6611) | A DT mainline da Max não tem nó de Wi-Fi (procurei `wifi` e `sdio` na DT da placa e nos `.dtsi` que ela inclui). A configuração do Armbian usa a extensão `bcmdhd` com interface SDIO, para o kernel de fornecedor. A imagem Ubuntu da comunidade também usa `bcmdhd-sdio-dkms` e blacklista `bcmdhd`. O build oficial bloqueia `brcmfmac` no ramo `next` da Max. O relato da Bunnix diz que o driver padrão falha com "unsupported chip 0xaabf". No fórum DietPi, o Wi-Fi funciona no kernel de fornecedor 6.1.84 (março de 2025). Em setembro de 2026, o mantenedor do DietPi disse que o mainline ainda não suporta o AP6611S. Relato de edge 7.0-rc6 (seção 4.1): nenhuma interface sem fio detectada. | Vendor 6.1.84 (mar/2025); mainline, mantenedor (set/2026); `next` com `brcmfmac` bloqueado (orangepi-build, 03/11/2026) | [oficial, DT, configuração do Armbian e do build] para o que está descrito; [comunidade/Max] para o funcionamento |
| Bluetooth | A DT habilita a UART7 e o nó `brcm,bcm43438-bt` (herdado da DT compacta da Orange Pi 5). Na configuração do Armbian, o kernel de fornecedor usa um serviço `brcm_patchram` na `ttyS7`, com o firmware `SYN43711A0.hcd`; o mainline usa o driver `hci_bcm`. Patch de 27/04/2025 (Jimmy Hon, aplicado por Heiko Stuebner em 01/05/2025, commit `db1fefec`). Relatos de março de 2025 (6.1.84) e de junho de 2025 (6.15) dizem que não funcionou. A Bunnix escreveu "Could not make it work (kernel 6.15)". O serviço `ap6611s-bluetooth` da imagem Ubuntu da comunidade usa `brcm_patchram_plus`. | DT: maio/2025; relatos: 6.1.84 e 6.15 | [oficial, código e configuração]; funcionamento: [comunidade/Max] |
| Ethernet 2,5 GbE (RTL8125BG) | A DT habilita `pcie2x1l1` com o regulador `vpcie3v3_pcie_eth`, então o slot PCIe 2x1 lane 1 é o da RTL8125 (dedução pelo nome do regulador). O código do driver `r8169` no master menciona RTL8125. A Bunnix (06/2025) usou o driver `r8125` compilado à parte (DKMS), "para kernels acima de 6.9". Não testei em placa com `r8169`. | Mainline (DT e driver no master, 03/11/2026) | [oficial, código-fonte]; funcionamento com `r8169` na Max: [não confirmado] |
| NVMe (PCIe 3.0 x4) | A DT habilita `pcie3x4` e `pcie30phy`, e o manual lista o slot M.2 M-Key com PCIe 3.0 x4. Não achei teste de estabilidade na Max. | DT no master (03/11/2026) | [oficial] para o slot; estabilidade: [não confirmado] |
| Áudio | O codec ES8388 e o jack analógico estão descritos na DT base. Relato da Bunnix: a saída pelo jack funcionou com PipeWire. HDMI áudio: ver linha própria. | DT no master | [oficial, código]; [comunidade/Max] para o relato |
| Ventoinha PWM | A DT habilita `&fan` com `pwm9`, período de 50000 ns (20 kHz). | DT no master | [oficial, código-fonte] |
| Decodificação de vídeo | O driver `rkvdec` com H.264 e H.265 (VDPU381) está no v7.0 (seção 3). A DT da Max que li não tem nó específico de decodificador, e não verifiquei a DT base. | Mainline: driver desde v7.0 | Driver: [oficial, código]; habilitação na Max: [não confirmado] |
| NPU | A DT base habilita `rknn_core_0`, `rknn_core_1` e `rknn_core_2`, com seus MMUs. O driver upstream é o `rocket`, que está no v6.18. Não testei em placa. Um relato de fórum diz que o driver de fornecedor funciona. | Mainline: driver desde v6.18 | Driver e DT: [oficial, código]; funcionamento no mainline: [não confirmado]; no fornecedor: [comunidade/Max, opinião] |

### 4.1 O que foi confirmado em placa real, no mainline

Esta subseção responde só ao que foi testado em hardware. Código e DT não entram aqui.

- **Relatos lidos sobre mainline na Max.** A Bunnix (wiki, escrito em 07/06/2025) diz que kernels 5.10.160 e 6.15 compilaram, mas que o 6.15 "starts but does not complete" por erro de DTB. Para contornar, copiou o DTB da imagem oficial. A própria página diz que o sistema "should boot, with working Wifi and Bluetooth", mas não deixa claro qual kernel fez o Wi-Fi funcionar. No kernel 6.15, o Bluetooth não funcionou ("Could not make it work"). [comunidade/Max, github.com/bunnyamin/bunnix/wiki/orangepi_5max, lido em 03/11/2026]
- **Armbian, tópico 59179** (edge 7.0-rc6, imagem Noble, KDE Neon). O autor relata que o Wi-Fi não detecta nenhuma interface sem fio e que o Bluetooth "está ativo, mas não acha dispositivo". Ele já tinha instalado os pacotes `armbian-firmware` e `full-mirror-armbian-firmware`. Não li resposta. A data não aparece na página lida; a listagem do fórum mostra abril. [comunidade/Max, forum.armbian.com/topic/59179, lido em 03/11/2026]
- **Armbian, tópico 49131** (janeiro de 2025). O autor relata que o vendor 6.1 da build dele ficava instável (`lsmod` e `modprobe` travavam, erros de PCIe), mas que a imagem Ubuntu rockchip, com kernel 6.1 de fornecedor, funcionava bem. O mantenedor disse que a placa é "unsupported" e não recebe build estável. [comunidade/Max, forum.armbian.com/topic/49131, lido em 03/11/2026]
- **Título, sem leitura.** A listagem de tópicos relacionados mostra "First boot on kernel 6.1.172 doesn't kick off provisioning script, but kernel 7.2.6 does provision correctly (Orange Pi 5 Max)", com data de 28 de setembro (ano não visível). Não abri o tópico. Se for verdade, o edge 7.2.6 chegou ao provisionamento, mas isso não cobre HDMI, GPU, NPU, 2,5 GbE nem decodificador. [comunidade/Max, título em listagem, não lido]

**Resultado por recurso no mainline, em placa real:**

| Recurso | Situação | Marcação |
|---|---|---|
| Boot | Mainline 6.15 deu erro de DTB em 2025. Edge 7.2.6 aparece no título como provisionando, sem leitura. | [comunidade/Max] |
| HDMI (saída) | Nenhum relato lido de imagem em mainline. | [não confirmado] |
| Panthor (GPU) | Nenhum relato lido. A comunidade usa panfork e `libmali`. | [não confirmado] |
| NPU (`rocket`) | Nenhum relato lido no mainline. | [não confirmado] |
| 2,5 GbE | Relato de 2025 usou `r8125` DKMS. Nenhum relato lido com `r8169`. | [não confirmado] |
| Decodificador | Nenhum relato lido. | [não confirmado] |
| Wi-Fi e Bluetooth | Bluetooth falhou no 6.15 (Bunnix, 2025). Wi-Fi não detectado no edge 7.0-rc6 (tópico 59179, sem resposta lida). | [comunidade/Max] |

**Conclusão.** Nenhuma fonte lida em 2026 confirma HDMI, Panthor, NPU, 2,5 GbE ou decodificador funcionando na Max em kernel mainline com hardware real. O que existe é código (seções 3 e 4) e relatos antigos ou de fornecedor. Teste na sua placa antes de depender de qualquer um desses recursos.

---

## 5. Distribuições e imagens

### 5.1 Imagem oficial Orange Pi

Lista de imagens oficiais lida na página de downloads da Max (03/11/2026) e nas pastas públicas do Google Drive que ela indica:

| Imagem | Kernel | Data no Drive | Marcação |
|---|---|---|---|
| Ubuntu 22.04 (jammy) desktop XFCE, versão 1.0.2 | 6.1.99 | 15/09/2026 | [oficial, Google Drive] |
| Ubuntu 22.04 (jammy) server, versão 1.0.2 | 6.1.99 | 15/09/2026 | [oficial, Google Drive] |
| Ubuntu 26.04 (resolute) desktop GNOME, versão 1.0.2 | 6.18.49 | 29/09/2026 | [oficial, Google Drive] |
| Debian 11 (bullseye) server, versão 1.0.2 | 6.1.99 | 15/09/2026 | [oficial, Google Drive] |
| Debian 12 (bookworm) server, versão 1.0.2 | 6.1.99 | 15/09/2026 | [oficial, Google Drive] |
| Debian 13 (trixie) desktop GNOME, versão 1.0.2 | 6.18.49 | 29/09/2026 | [oficial, Google Drive] |

- As datas são as de modificação exibidas pelo Drive. O ano 2026 vem dos metadados dos arquivos.
- O Drive lido não lista servidor para Ubuntu 26.04 nem para Debian 13. Também não lista as imagens de 2024 com kernel 5.10.160 que o manual cita. Essas imagens antigas não aparecem nas pastas lidas. [oficial]
- As pastas de Orange Pi OS (Droid), Orange Pi OS (Arch), Android e OpenWRT existem na página de downloads, mas não as abri. Por isso não tenho data nem versão de kernel delas. [não confirmado]
- **Manual v1.3** (24/09/2024, p. 558-559). Lista imagens de 2024 com kernel 5.10.160 (Ubuntu 20.04 e 22.04, Debian 11 e 12) e 6.1.43 (Ubuntu 22.04 e Debian 12). O que o Drive mostra é mais novo que isso. [oficial, manual v1.3]
- **OpenWRT** oficial: `openwrt-rockchip-armv8-xunlong_orangepi-5max-ext4-sysupgrade_v1.0.img.gz`, de 19/07/2024, na lista do manual. [oficial, manual v1.3, p. 559]
- **Site oficial.** A ficha e a página de downloads da Max estão no site da Orange Pi e foram lidas diretamente em 03/11/2026. A ficha lista Orange Pi OS (Droid), (Arch), (OH), Ubuntu, Debian e Android 13. [oficial, orangepi.org]
- **Android** oficial: arquivos `OrangePi5Max_RK3588_Android13_v1.0.0.tar.gz` e a variante `_spi-nvme_`, ambos de 2024, citados no manual. A página de downloads também tem código-fonte do Android. [oficial, manual v1.3, p. 558]. Não verifiquei se há imagem Android posterior a 2024.

### 5.2 Armbian

- **Página da Max no catálogo.** Armbian Community, "Rolling Release", build de 03/11/2026. Kernels: edge 7.3.1 e vendor 6.1.172. Imagens: Ubuntu 26.04 "resolute" (GNOME e KDE Plasma, edge e vendor; também CLI) e Debian 13 "trixie" (CLI, edge e vendor). Tamanhos de 405 MB a 1,4 GB. [comunidade/Max, armbian.com, 03/11/2026]
- **Configuração da placa.** O arquivo `orangepi5-max.csc` do repositório `armbian/build` (branch main, lido em 03/11/2026) tem `BOARD_MAINTAINER=""` (vazio), `INTRODUCED="2024"`, `KERNEL_TARGET="vendor,current,edge"` e `KERNEL_TEST_TARGET="vendor,edge"`. Usa U-Boot mainline no kernel edge e a extensão `bcmdhd` com interface SDIO para o Wi-Fi. [oficial, armbian/build]
- **O que "Community" quer dizer.** As regras do projeto, na página de regras de suporte de placas, definem três níveis. "Standard Support" exige um mantenedor nomeado com acesso físico à placa. "Staging" (trabalho em andamento) exige mantenedor comprometido, desenvolvimento ativo e compilação quase sempre bem-sucedida. "Community maintained" descreve placas "não sob supervisão ou desenvolvimento ativo", que a comunidade mantém, e que podem sair da base de código. A Max cai no último nível: não tem mantenedor nomeado no `.csc` e a página do catálogo a rotula como Community. [oficial, docs.armbian.com/contribute/board-support-rules/; sem data de atualização visível na página]
- **Divergência de 2024.** Uma matéria de 2024 disse que a Max entrou no Armbian 24.11 como SoC suportado. O `.csc` mostra `INTRODUCED="2024"`, o que confirma a entrada em 2024, mas não o nível. Não confirmei o nível de 2024 em fonte primária. [não confirmado para o nível original]
- Os rótulos da página são o que o site mostra hoje. Não atribua a eles garantias que não foram lidas.

### 5.3 Ubuntu Rockchip (Joshua Riek)

- Projeto que gerava imagens Ubuntu para placas Rockchip, com kernel de fornecedor 6.1 e patches próprios.
- O repositório foi **arquivado pelo dono em 29/04/2026** e está só para leitura. [oficial do projeto, página de releases do GitHub, lida em 03/11/2026]
- A última versão é a v2.4.0 (23/10/2024), com Ubuntu 22.04, 24.04 e 24.10. Lido em 03/11/2026, o texto da v2.4.0 **não cita a Max**.
- A Max aparece nas notas da **v2.3.0** (02/08/2024): "Added support for the Orange Pi 5 Max". [oficial do projeto, notas de versão]
- **Divergência resolvida.** Uma busca disse que a v2.4.0 adicionou a Max. As notas da v2.4.0 não citam a placa. Vale a v2.3.0.
- **Script da Max no repositório** (`config/boards/orangepi-5-max.sh`, lido em 03/11/2026). Suítes jammy e noble, com flavors server e desktop. Instala `panfork` (PPA `jjriek/panfork-mesa`), `mali-g610-firmware`, `libmali-g610-x11`, `camera-engine-rkaiq-rk3588`, `bcmdhd-sdio-dkms` e o serviço `ap6611s-bluetooth`, e blacklista `bcmdhd`. Mostra como a imagem comunitária da Max chegou a GPU e Wi-Fi: por blob e driver de fornecedor, não pelo mainline. [comunidade/Max, código do projeto]
- Como o projeto está arquivado, não recomendo para servidor novo. Serve de referência do que já foi feito.

### 5.4 DietPi

- A Max aparece na tabela de placas Orange Pi do DietPi, com imagem ARMv8 para Debian Trixie. A página não dá nível de suporte para a Max. [comunidade/Max, dietpi.com/docs/hardware, lida em 03/11/2026]
- O mantenedor do DietPi, em 13/09/2026, disse que Max e Ultra continuavam no kernel de fornecedor, por causa do Wi-Fi AP6611S. Novas imagens da Orange Pi 5 Plus já vinham com mainline 6.18. [comunidade/Max, fórum DietPi]
- Um fórum de março de 2025 mostrou Wi-Fi funcionando e Bluetooth não funcionando na Max, em DietPi 9.11 com kernel 6.1.84-vendor-rk35xx. Não houve resposta oficial do DietPi nesse tópico, que foi fechado automaticamente em setembro de 2025.

### 5.5 Debian

- **Oficial, em setembro de 2026:** Debian 11 (bullseye) e 12 (bookworm), server, com kernel 6.1.99. Debian 13 (trixie), desktop GNOME, com kernel 6.18.49. [oficial, Google Drive, lido em 03/11/2026]
- **Oficial, em 2024 (manual v1.3):** Debian 11 e 12, server e desktop XFCE, com kernel 5.10.160 ou 6.1.43. Essas imagens antigas não aparecem no Drive lido. [oficial, manual v1.3, p. 558]
- **Não oficial:** o DietPi tem Debian Trixie para a Max, e o Armbian tem Debian 13 (seção 5.2). [comunidade/Max]

### 5.6 NixOS

- **Página da wiki** (`wiki.nixos.org/wiki/NixOS_on_ARM/Orange_Pi_5`, lida em 03/11/2026). A página trata a Orange Pi 5 como placa com SoC RK3588s, não RK3588. Diz que "None of the basic components (Kernel, HW acceleration drivers) are available from upstream NixOS". Lista dois flakes da comunidade, de ryan4yin e fb87. Não cita a Max. [oficial da wiki do NixOS]
- **Flake de ryan4yin** (`github.com/ryan4yin/nixos-rk3588`, lido em 03/11/2026). O repositório está arquivado e teve último push em 04/10/2026. A tabela de placas do README lista só **Orange Pi 5 e Orange Pi 5 Plus**. A Max não aparece. [comunidade, código e README]
- **Conclusão.** Não achei suporte NixOS para a Max em fonte lida. As placas listadas pelos flakes são irmãs. Para a Max, NixOS é trabalho de adaptação, sem suporte confirmado. [não confirmado]

### 5.7 Android

- **Oficial:** Android 13 para a Max, com arquivos `OrangePi5Max_RK3588_Android13_v1.0.0.tar.gz` e a variante `_spi-nvme_`, ambos de 2024. Também há código-fonte do Android no site. [oficial, manual v1.3, p. 558, e página de downloads]
- Não verifiquei se há imagem Android posterior a 2024. A pasta do Drive "Android Image" existe, mas não a abri.

### 5.8 OpenWrt

- **Oficial:** imagem `openwrt-rockchip-armv8-xunlong_orangepi-5max-ext4-sysupgrade_v1.0.img.gz`, de 19/07/2024, na lista do manual. [oficial, manual v1.3, p. 559]
- Um fórum sobre a 5 Plus diz que a imagem oficial dessa placa não funciona bem e que o OpenWrt do projeto já suporta a 5 Plus. [placa irmã: 5 Plus] Não testei a imagem da Max.

---

## 6. Ferramenta orangepi-build e repositórios orangepi-xunlong

Dados lidos em 03/11/2026, via API de leitura do GitHub.

- **Sistema hospedeiro.** O manual v1.3 diz que o orangepi-build roda em Ubuntu 22.04 instalado na própria placa ou em computador x64 com Ubuntu 22.04. Outros sistemas não foram testados. [oficial, manual v1.3, p. 459]
- **Repositório.** `orangepi-xunlong/orangepi-build`. Descrição: build para H2+, H3, H5, H6, H616, RK3328, RK3399 e RK3588(s). Branch padrão `next`, com branch extra `main`. Último push em 28/09/2026. [oficial do repositório]
- **Raiz do repositório.** Tem `.gitignore`, `LICENSE`, `README.md`, `build.sh`, `external/` e `scripts/`. Não há pasta `config/` na raiz (a API devolveu 404 para ela). A configuração fica dentro de `external/`. [oficial, GitHub]
- **Conteúdo de `external/`.** Pastas `cache/` (com `debs/` e `sources/`), `config/`, `extensions/` (scripts shell de apoio, como `flash-kernel.sh`, `grub.sh` e `rkbin-tools.sh`), `packages/` (com `blobs/`, `bsp/`, `orangepi/`, `pack-uboot/`, `plymouth-theme-orangepi/`, `raspi/` e `extras-buildpkgs/`) e `patch/` (com `atf/` e `misc/`). [oficial, GitHub]
- **Configuração da Max** (`external/config/boards/orangepi5max.conf`, lido em 03/11/2026). Valores principais:
  - `BOARDFAMILY="rockchip-rk3588"`, `BOOTCONFIG="orangepi_5_max_defconfig"`, `BOOT_FDT_FILE="rockchip/rk3588-orangepi-5-max.dtb"`.
  - `KERNEL_TARGET="legacy,current,next"`.
  - `DISTRIB_TYPE_LEGACY="bullseye bookworm bionic focal jammy"`, `DISTRIB_TYPE_CURRENT="bullseye bookworm jammy"`, `DISTRIB_TYPE_NEXT="resolute trixie"`.
  - `MODULES_BLACKLIST_NEXT="brcmfmac"`.
  - `REVISION="1.0.2"`.
- **Família RK3588** (`external/config/sources/families/rockchip-rk3588.conf`, lido em 03/11/2026). Liga os nomes internos aos ramos de kernel:
  - `legacy` = `branch:orange-pi-5.10-rk35xx`. Para Max usa a configuração `linux-rockchip-rk3588-legacy-opi5max`.
  - `current` = `branch:orange-pi-6.1-rk35xx`. Para Max usa `linux-rockchip-rk3588-current-opi5pro`.
  - `next` = `branch:orange-pi-6.18-rk35xx`. Usa `linux-rockchip-rk3588-next`.
- **Ramos e versões do kernel** (repositório `orangepi-xunlong/linux-orangepi`, lido em 03/11/2026, pelo `Makefile` e pelo último commit de cada ramo):
  - `orange-pi-5.10-rk35xx`: 5.10.160, último commit 07/04/2025.
  - `orange-pi-6.1-rk35xx`: 6.1.99, último commit 20/08/2026.
  - `orange-pi-6.18-rk35xx`: 6.18.49, último commit 28/09/2026.
  Os três existem nesse repositório. [oficial]
- **Nomes do Armbian são outros.** "vendor", "current" e "edge" são nomes do Armbian. Os nomes `legacy`, `current` e `next` são do `orangepi-build`. Não são a mesma coisa, mesmo quando o número de kernel coincide. [oficial, armbian/build e orangepi-build]
- **Repositório separado `orangepi-xunlong/external`.** Existe, com branch padrão `master` e último push em 25/03/2020. Não é a pasta `external/` que o build usa. Não abri o conteúdo desse repositório separado. [não confirmado quanto ao conteúdo]
- **Repositório `orangepi-xunlong/firmware`.** A listagem tem `nvram_ap6611s.txt` e também `nvram_ap6611s.txt-orangepi5ultra`. Não confirmei qual dos dois a Max usa. [oficial, listagem do repositório, lida em 03/11/2026]
- Não repito comandos de build, porque não li o README completo. Antes de rodar qualquer comando, leia o README da versão que você for usar.
- **Aviso de risco.** Em ferramentas de imagem, comandos como `dd` ou `sfdisk` gravam diretamente em dispositivos e apagam dados do disco escolhido. Antes de rodar, confirme o dispositivo com `lsblk` e faça backup. Essa advertência vale para qualquer instalação manual de imagem.

---

## 7. Qual escolher para servidor sem monitor

Critérios: servidor headless, rede por cabo, atualização de segurança, pouca manutenção.

Recomendação, com justificativa:

1. **Imagem oficial Debian 12 (bookworm) server, kernel 6.1.99 (versão 1.0.2, 15/09/2026).** É a opção oficial mais recente para servidor. Vem da Orange Pi, com kernel de fornecedor de linha 6.1, que é o caminho de Wi-Fi e Bluetooth mais conhecido na Max. Não confirmei o funcionamento de cada periférico com essa imagem. [oficial, Google Drive; comunidade/Max para o histórico de Wi-Fi]
2. **Imagem oficial Ubuntu 22.04 (jammy) server, kernel 6.1.99 (15/09/2026).** Alternativa oficial, com o mesmo kernel de linha 6.1. [oficial, Google Drive]
3. **Armbian, variante minimal (CLI), Debian 13 ou Ubuntu 26.04, vendor 6.1.172.** É a opção com kernels atualizados na Max, com atualização pelo projeto. O nível é **Community**: o projeto não tem mantenedor nomeado para a placa e pode retirá-la do código. Quem escolher esta opção assume a manutenção. [oficial, configuração e regras do Armbian; comunidade/Max, catálogo]
4. **Kernel `next` (6.18.49) da Orange Pi, ou edge (mainline 7.x), só para teste.** Não é recomendação de servidor. O `next` bloqueia `brcmfmac`, e o mainline não tem relato lido de HDMI, GPU, NPU, 2,5 GbE ou decodificador funcionando na Max (seção 4.1). Se o servidor usar só cabo e não precisar de Wi-Fi, valide o 2,5 GbE e o NVMe antes de adotar. [oficial, orangepi-build; não confirmado para funcionamento]
5. **DietPi.** Também é opção para servidor, com Debian Trixie, mas a Max não tem nível de suporte declarado na página lida, e o mantenedor disse em setembro de 2026 que a Max ainda estava no kernel de fornecedor. [comunidade/Max]
6. **Ubuntu Rockchip.** Não recomendo para novo servidor, porque o repositório está arquivado desde 29/04/2026.
7. **OpenWrt oficial.** Bom para roteamento, mas a imagem é de 2024. [oficial, manual v1.3, p. 559]. **Android e NixOS.** Não são a escolha para servidor headless de uso geral com a Max, pelas fontes lidas. NixOS exige trabalho extra, sem suporte Max confirmado (seção 5.6).

Ressalva: a escolha entre vendor e edge é uma decisão de risco. Vendor tem menos atualização de segurança do kernel. Edge tem menos driver pronto e a Max não tem nível de suporte do Armbian. Esta recomendação não é consenso das fontes, é um julgamento a partir delas.

---

## 8. Como atualizar o kernel com segurança

Estas são práticas gerais. A documentação do Armbian sobre atualização não foi lida nesta coleta; confirme-a antes de aplicar.

- Faça **backup completo** do cartão ou do eMMC antes de qualquer troca de kernel. Para isso, use uma imagem do disco inteiro guardada fora da placa.
- **Não troque de linha de kernel (vendor para edge, ou o inverso) sem ler as notas da distribuição.** Essa troca muda device tree, driver de Wi-Fi e firmware. O DietPi, por exemplo, fez a migração da 5 Plus com aviso de teste de 1 a 2 ciclos antes de oferecer aos usuários antigos.
- Atualize o kernel pelo gerenciador de pacotes da distribuição, sem compilar à mão, e mantenha a entrada anterior no carregador de boot para voltar ao kernel antigo se o novo não subir.
- Depois de reiniciar, confira `uname -r`, o log do kernel (`dmesg`) e os recursos que você usa (rede, NVMe) antes de dar o sistema por pronto.
- Em servidor sem monitor, tenha um acesso de recuperação (console serial ou acesso físico) antes de reiniciar. Se o kernel novo não subir, o SSH some junto.
- Para migrar de distribuição, não há comando confiável nas fontes lidas para a Max. Não informe nem rode nada sem ler a documentação oficial da distribuição escolhida.

---

## 9. Divergências entre fontes

- **Imagens oficiais: manual v1.3 versus Google Drive.** O manual (2024) lista kernels 5.10.160 e 6.1.43. O Drive (setembro de 2026) lista 6.1.99 e 6.18.49. Não é contradição: o Drive é mais novo. Vale o Drive para imagens atuais. [oficial]
- **Versões do Armbian.** Página da Max (03/11/2026): edge 7.3.1, vendor 6.1.172. Uma busca retornou snapshot de 02/09/2026 com edge 7.2.2 e vendor 6.1.115, e outra página do mesmo catálogo de 23/10/2026 com edge 7.3.0. Adotei a página de 03/11/2026, que foi aberta.
- **Nível Armbian em 2024.** Matéria de 2024 disse que a Max foi incluída no 24.11 como SoC suportado. A configuração atual tem `INTRODUCED="2024"` e o nível atual é Community. Não confirmei o nível original em fonte primária.
- **Ubuntu Rockchip v2.4.0.** Resolvida: a Max aparece nas notas da v2.3.0, não nas da v2.4.0.
- **Bluetooth.** Relato de 6.1.84 (março de 2025) não funcionava. Patch de DT de 05/2025 habilitou a UART7. Relato de 6.15 (junho de 2025) também não funcionou, e a Bunnix confirma "Could not make it work (kernel 6.15)". Não há relato posterior confirmado de funcionamento completo.
- **Wi-Fi.** Relato Bunnix diz que o driver padrão falha. Relato DietPi diz que o Wi-Fi funciona no vendor 6.1.84. Não são contraditórios: falha no driver padrão e funciona no kernel de fornecedor com firmware certo. O mantenedor de setembro de 2026 diz que o mainline ainda não suporta o AP6611S. O código do mainline não tem nó de Wi-Fi na DT da Max. O relato do tópico 59179 (edge 7.0-rc6) diz que nenhuma interface sem fio aparece, sem resposta lida.
- **Ethernet 2,5 GbE.** A Bunnix diz que o `r8125` era necessário "até o kernel 6.9" e que acima disso é preciso compilar o módulo. Isso indica o uso de driver externo, não que o `r8169` do mainline falhe na Max. Não testei o `r8169`.
- **Decodificação H.264/H.265 do RK3588.** Resolvida pelo código: o driver VDPU381 está no v7.0 e não no v6.19. A matéria de fevereiro de 2026 também diz 7.0.
- **Entre placas.** O Armbian e o DietPi já migraram placas irmãs (5 Plus) para mainline, enquanto a Max continua no vendor. Não generalize para a Max.
- **Especificações.** O manual (p. 13) dá 58 g de peso e a ficha oficial dá 62 g. Não afeta o kernel; anotado só para conferência.

---

## 10. O que não foi possível confirmar

- **HDMI, Panthor, NPU, 2,5 GbE e decodificador funcionando no mainline, em placa real.** Não achei relato de 2026 lido. Onde procurei: fóruns Armbian (tópicos 49131, 59179 e 44637, lidos em 03/11/2026), wiki da Bunnix (escrita em 2025), busca na web por Armbian, Panthor e rocket em 2026, e o mantenedor do DietPi (setembro de 2026). O tópico de Armbian "First boot on kernel 6.1.172 ... 7.2.6" foi visto só pelo título.
- **Funcionamento na placa em 2026.** O código mostra que o mainline descreve HDMI, NVMe, 2,5 GbE, NPU e GPU da Max, mas não testei em hardware.
- **Wi-Fi AP6611S e Bluetooth no mainline depois de setembro de 2026.** Não achei relato posterior.
- **Decodificador de vídeo na DT da Max.** O driver existe no v7.0; não confirmei se a DT da Max o habilita.
- **Nível "Supported" de 2024 para a Max.** Não há fonte primária que mostre o nível original.
- **Imagens Orange Pi OS (Droid), Orange Pi OS (Arch), Android e OpenWRT atuais.** A página de downloads as lista, mas não abri as pastas do Drive.
- **Qual nvram do AP6611S a Max usa.** A listagem do firmware tem o arquivo genérico e o da Ultra.
- **NixOS para a Max.** Wiki e flake listam só placas irmãs (Orange Pi 5 e 5 Plus; a wiki menciona RK3588s).
- **Notas de status do RK3588 da Collabora (`mainline-status.md`).** Na coleta anterior, o acesso retornou verificação anti-robô. Não retentei nesta coleta, conforme a regra de não contornar bloqueio.

---

## Fontes

Lidas com sucesso em 03/11/2026 (ou na coleta anterior, onde marcado):

- http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/service-and-support/Orange-Pi-5-Max.html (página de downloads, lida diretamente)
- http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/details/Orange-Pi-5-Max.html (ficha oficial, lida diretamente)
- http://www.orangepi.org/orangepiwiki/index.php/Third_Party_Images (wiki, sem a Max na lista)
- https://drive.google.com/drive/folders/1qSA5yr_UOll3LKMzdn74i_SCYygnIcJ8?usp=sharing (pasta oficial de Ubuntu)
- https://drive.google.com/drive/folders/1b6hqA6zdgiScWvohsUdopBrtmytF4-ma?usp=sharing (pasta oficial de Debian)
- https://github.com/torvalds/linux/blob/master/arch/arm64/boot/dts/rockchip/rk3588-orangepi-5-max.dts (arquivo lido via API do GitHub; também presente no tag v7.3 e v6.14)
- https://github.com/torvalds/linux/blob/master/arch/arm64/boot/dts/rockchip/rk3588-orangepi-5-compact.dtsi
- https://github.com/torvalds/linux/blob/master/arch/arm64/boot/dts/rockchip/rk3588-orangepi-5.dtsi
- https://github.com/torvalds/linux/commits/master/arch/arm64/boot/dts/rockchip/rk3588-orangepi-5-max.dts (histórico: primeiro commit de 09/01/2025, c600d252dc52)
- https://github.com/torvalds/linux/tree/v6.18/drivers/accel/rocket (presença do driver por tag)
- https://github.com/torvalds/linux/tree/v7.0/drivers/media/platform/rockchip/rkvdec (presença de VDPU381 por tag)
- https://github.com/armbian/build/blob/main/config/boards/orangepi5-max.csc
- https://docs.armbian.com/contribute/board-support-rules/
- https://www.armbian.com/orange-pi-5-max/
- https://www.armbian.com/boards/orangepi5-max
- https://forum.armbian.com/topic/49131-orange-pi-5-max-builds/ (lido em 03/11/2026)
- https://forum.armbian.com/topic/59179-wifi-and-bluetooth-not-working-on-orange-pi-5-max-armbian-noble-edge-70-rc6/ (lido em 03/11/2026)
- https://forum.armbian.com/topic/44637-orange-pi-5-max-debian/ (lido em 03/11/2026)
- https://github.com/orangepi-xunlong/orangepi-build (repositório, branches e raiz)
- https://github.com/orangepi-xunlong/orangepi-build/blob/next/external/config/boards/orangepi5max.conf
- https://github.com/orangepi-xunlong/orangepi-build/blob/next/external/config/sources/families/rockchip-rk3588.conf
- https://github.com/orangepi-xunlong/linux-orangepi (ramos `orange-pi-5.10-rk35xx`, `orange-pi-6.1-rk35xx` e `orange-pi-6.18-rk35xx`, com `Makefile` e último commit)
- https://github.com/Joshua-Riek/ubuntu-rockchip (repositório arquivado; script `config/boards/orangepi-5-max.sh`)
- https://github.com/Joshua-Riek/ubuntu-rockchip/releases (notas das versões v2.3.0 e v2.4.0)
- https://github.com/bunnyamin/bunnix/wiki/orangepi_5max (lido em 03/11/2026)
- https://wiki.nixos.org/wiki/NixOS_on_ARM/Orange_Pi_5 (lido em 03/11/2026)
- https://github.com/ryan4yin/nixos-rk3588 (README e tabela de placas, lidos em 03/11/2026)
- https://github.com/orangepi-xunlong/firmware (listagem)
- https://dietpi.com/docs/hardware/
- https://dietpi.com/forum/t/orange-pi-5-max-help-with-bluetooth/23116.md
- https://dietpi.com/forum/t/should-orange-pi-5-too-be-moved-to-mainline-linux/25486.md
- https://www.cnx-software.com/2024/08/01/rockchip-rk3588-powered-orange-pi-5-max-sbc-features-up-to-16gb-lpddr5-2-5gbe-onboard-wifi-6e-and-bluetooth-5-3/
- https://www.cnx-software.com/2026/02/27/rockchip-rk3588-rk3576-h-264-and-h-265-video-decoders-mainline-linux/
- https://www.collabora.com/news-and-blog/news-and-events/rockchip-rk3588-upstream-support-progress-future-plans.html (lida na coleta anterior, de dezembro de 2024)
- https://lists.infradead.org/pipermail/linux-rockchip/2025-May/057891.html (lida na coleta anterior)
- https://web.archive.org/web/2026/http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/details/Orange-Pi-5-Max.html (cópia arquivada em 17/10/2026, usada para comparação)

Fontes de documentação não lidas nesta coleta, ou bloqueadas:

- Manual do usuário Orange Pi 5 Max, versão 1.3 (24/09/2024), citado por página: caps. 1 (p. 12-13), 5 (p. 459) e 11 (p. 557-559). Cópia de terceiros, sem URL.
- Notas de status do RK3588 da Collabora (`mainline-status.md`): na coleta anterior o acesso retornou verificação anti-robô. Não retentada nesta coleta.
- Fórum Armbian, tópico 51307 (Orange Pi 5 Ultra): não lido nesta coleta. Kernelsources.org: timeout na coleta anterior. Página de níveis do Armbian antiga (só resultado de busca, substituída pela página de regras lida).
- Tópico do fórum Armbian "First boot on kernel 6.1.172 ... 7.2.6": visto apenas pelo título na listagem. Não aberto.
- Pastas do Google Drive de Orange Pi OS (Droid), Orange Pi OS (Arch), Android e OpenWRT: não abertas.
