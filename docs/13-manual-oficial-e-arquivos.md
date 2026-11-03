# Orange Pi 5 Max: índice de fontes oficiais e resumo do manual

Data da coleta: 03/11/2026.

Este arquivo reúne onde estão as fontes da Orange Pi 5 Max, um resumo do manual do usuário e as respostas às perguntas em aberto do repositório, com a página do manual que sustenta cada resposta. Ele foi escrito para ser lido sozinho.

Como ler este arquivo:

- **Manual:** versão **v1.3** (2024-09-24), conforme o histórico do capítulo 11.1 (p. 557). É a versão listada na pasta oficial do Google Drive em 03/11/2026. O texto foi lido em uma cópia de distribuidor com o mesmo número de versão. "p." indica a página do manual.
- **Esquemático:** "esq. p." indica a página do esquemático oficial V1.1.
- **Ficha:** é a página oficial do produto, lida em 03/11/2026.
- O site orangepi.org abre normalmente em navegador. Tentativas anteriores falharam por limitação da ferramenta automática de leitura, não por o site estar fora do ar.

---

## Parte 1: onde achar as fontes

### 1.1. Página oficial do produto

http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/details/Orange-Pi-5-Max.html

Traz a ficha técnica, as fotos e os links de compra das lojas oficiais. Dados da ficha que costumam gerar dúvida:

- PCB de 89 mm × 57 mm × 1,6 mm e peso de 62 g.
- Alimentação por USB-C, 5 V @ 5 A.
- RAM LPDDR5 de 4, 8 ou 16 GB.
- Soquete eMMC por padrão, ou eMMC soldada sob encomenda, de 32 a 256 GB.
- QSPI NOR Flash de 16 MB, microSD e M.2 M-Key (NVMe, PCIe 3.0 x4).
- Um botão BOOT e um botão liga/desliga. LED RGB lateral.
- Ventoinha: conector 5 V, 2 pinos, 1,25 mm. RTC: conector 3 V, 2 pinos, 1,25 mm.
- Serial de depuração incluída no conector de 40 pinos.
- Sistemas: Orange Pi OS (Droid), Orange Pi OS (Arch), Orange Pi OS (OH), Ubuntu, Debian e Android 13.

Lojas oficiais citadas na ficha: [AliExpress, Xunlong Store](https://www.aliexpress.com/item/1005007348974947.html), [Amazon, link 1](https://www.amazon.com/dp/B0HC2RRQY6?th=1) e [Amazon, link 2](https://www.amazon.com/dp/B0DB162HRV?th=1).

### 1.2. Página oficial de downloads

http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/service-and-support/Orange-Pi-5-Max.html

Os arquivos ficam em pastas públicas do Google Drive:

| Item | Link |
|---|---|
| Manual do usuário | https://drive.google.com/drive/folders/1kzMRI95yaXLbQuK86fUbs92NJ6QOYIGO?usp=sharing |
| Esquemático | https://drive.google.com/file/d/1cAKUBHF2yXtmmLJaVtK17nOKoMlI02wA/view?usp=sharing |
| Desenho mecânico | https://drive.google.com/file/d/1HTPhgRq5EuVlTSzSiLY6-X320II8eArF/view?usp=sharing |
| Ferramentas oficiais | https://drive.google.com/drive/folders/19OaB7_ND4dqxyczOTODb4hnIXGZ1C2g0?usp=sharing |
| Imagem Ubuntu | https://drive.google.com/drive/folders/1qSA5yr_UOll3LKMzdn74i_SCYygnIcJ8?usp=sharing |
| Imagem Debian | https://drive.google.com/drive/folders/1b6hqA6zdgiScWvohsUdopBrtmytF4-ma?usp=sharing |
| Imagem OpenWRT | https://drive.google.com/drive/folders/1B6obLNB60PAGGVB2-3wwjaxjteeA8SfE?usp=sharing |
| Orange Pi OS (Arch) | https://drive.google.com/drive/folders/1x2nDg804Zbb1uM-_fwjPfa70kYUMDDIX?usp=sharing |
| Orange Pi OS (Droid) | https://drive.google.com/drive/folders/142TL5_XiLSkHu6o7mTZfdhgB1Ey6Ap1A?usp=sharing |
| Imagem Android | https://drive.google.com/drive/folders/1SQcrQI8Qn1MbAcv3ggxhAPyIMTQ2-Xog?usp=sharing |
| Código-fonte Android | https://drive.google.com/drive/folders/1ZLCuijRyAWZLThL5T3kl5VoRAAC_GYms?usp=sharing |
| Código-fonte Linux | https://github.com/orangepi-xunlong/orangepi-build |

Nenhum desses arquivos foi copiado para este repositório, porque o conteúdo é do fabricante.

### 1.3. Manual do usuário

Pasta oficial na tabela acima, com o arquivo `OrangePi_5_Max_RK3588_User Manual_v1.3.pdf`. O manual cobre hardware, primeiro uso, gravação de imagens (TF, eMMC, SPI + NVMe, SPI + USB), Linux, OpenWRT, Android 13, compilação do SDK e o histórico de versões.

### 1.4. Esquemático

Arquivo oficial `OPI 5 MAX V1_1_SCH.pdf`, 29 páginas, revisão V1.1 (link na tabela acima). É dele que saem a ordem de boot, a entrada USB-C e a polaridade da ventoinha, descritas na parte 3.

### 1.5. Desenho mecânico

Arquivo oficial `OPi_5_Max_V1_1-DXF.pdf`, 2 páginas (referência inferior e superior), em vetor (link na tabela acima). O contorno da placa no desenho mede 89,0 mm × 57,0 mm na escala real do PDF.

O arquivo não traz cotas em texto. Por isso, a distância entre os furos não está registrada aqui: uma tentativa de medi-la pela geometria deu valores inconsistentes e foi descartada. Para medir, abra o PDF em um editor vetorial, como o Inkscape.

### 1.6. Wiki, fórum e canais oficiais

- Wiki: http://wiki.orangepi.org (a página antiga `Orange_Pi_5_Max` retornou 404 em 03/11/2026).
- Imagens de terceiros: http://www.orangepi.org/orangepiwiki/index.php/Third_Party_Images
- Fórum: http://www.orangepi.org/orangepibbsen/
- Discord: https://discord.gg/hE5wdMPm7z
- Telegram: https://t.me/OPi_official_group e https://t.me/Orange_Pi_Devices

### 1.7. Repositórios orangepi-xunlong no GitHub

Organização: https://github.com/orangepi-xunlong (página aberta; os repositórios fixados foram listados nela).

- **orangepi-build**: https://github.com/orangepi-xunlong/orangepi-build
  - Sistema de build para gerar imagens das placas Orange Pi. A descrição oficial cita H2+, H3, H5, H6, H616, RK3328, RK3399 e RK3588(s). Pelo README (resumo obtido pela ferramenta de leitura), o host de build indicado é Ubuntu 22.04, e a licença é GPL-2.0. Não verifiquei se há configuração específica para a Max.
- **wiringOP**: https://github.com/orangepi-xunlong/wiringOP
  - Descrição na organização: "wiringPi for Orange Pi". Biblioteca de GPIO, I2C, SPI, UART e PWM. O manual (p. 224 a 257) traz a instalação e o uso do wiringOP e do wiringOP-Python.
- **linux-orangepi**: https://github.com/orangepi-xunlong/linux-orangepi
  - Repositório fixado, sem descrição na página da organização. Pelo nome, deve ser o fonte do kernel Linux. Não abri o conteúdo.
- **u-boot-orangepi**: https://github.com/orangepi-xunlong/u-boot-orangepi
  - Fonte do U-Boot, sem descrição no repositório (ramo v2020.04). Na raiz, há só diretórios genéricos do U-Boot (arch, board, configs, common etc.), sem pasta específica da Max. A listagem da pasta `configs` é truncada pelo GitHub (303 entradas omitidas), e nos nomes visíveis não há arquivo com "orangepi-5-max" ou "rk3588". Não consigo afirmar que não existe, só que não apareceu.
- **OrangePi_Build**: https://github.com/orangepi-xunlong/OrangePi_Build
  - Sistema de build mais antigo, descrito na organização como voltado a A64, RDA e RK3399. Não verifiquei se cobre a Max.
- **openwrt**: https://github.com/orangepi-xunlong/openwrt
  - Descrito como voltado à Orange Pi R1 Plus. Não se aplica à Max.

---

## Parte 2: resumo do manual v1.3

### 2.1. Principais seções

- **Cap. 1, Recursos básicos (p. 11 a 16):** o que é a placa, usos, especificações (p. 12 e 13), vistas superior e inferior (p. 14 e 15), descrição das interfaces (p. 15) e furos de fixação (p. 16).
- **Cap. 2, Primeiro uso (p. 17 a 144):** acessórios (p. 17 a 20), gravação de imagem em cartão TF (Windows e Ubuntu), eMMC, SPI + NVMe, SPI + USB, OpenWRT, Android 13 e Orange Pi OS (Droid). Limpeza da SPI (p. 128). Ligação da placa (p. 135). Porta serial de depuração (p. 136 a 143). Alimentação pelo pino 5 V do conector de 40 pinos (p. 144).
- **Cap. 3, Linux Ubuntu/Debian (p. 145 a 425):** login padrão (p. 149), rede, Wi-Fi, hotspot, SSH, adb, HDMI, Bluetooth, USB, áudio, sensor de temperatura, conector de 40 pinos (p. 222), wiringOP (p. 224 a 257), Docker, instalação de desktop, ROS, Qt, NPU com RKNN (p. 360) e RKLLM (p. 382).
- **Cap. 4, Orange Pi OS Arch (p. 426 a 458).**
- **Cap. 5, SDK orangepi-build (p. 459).**
- **Cap. 6, Manual de desenvolvimento Linux (p. 483).**
- **Cap. 7 e 8, Android 13 (p. 485 e 516).**
- **Cap. 9 e 10, OpenWRT (p. 518 e 555).**
- **Cap. 11, Apêndice (p. 557):** histórico de versões do manual e das imagens.

### 2.2. O que importa na primeira ligação

1. **Fonte:** use um adaptador **5 V** com conector USB-C. O manual recomenda **5 V/5 A** (p. 13, p. 15 e p. 18). A porta USB-C **não faz negociação PD** e aceita só 5 V fixos (p. 18 e p. 135). Nunca use fonte com saída acima de 5 V, porque pode queimar a placa (p. 135). Não alimente pela porta USB de um computador (p. 136).
2. **Cartão de boot:** para o primeiro uso com cartão TF, use no mínimo 16 GB (de preferência 32 GB ou mais), classe 10 (p. 17). Para gravar no Windows, o manual cita balenaEtcher, RKDevTool e Win32DiskImager (cap. 2.3, p. 21 a 37).
3. **Vídeo e periféricos:** HDMI para HDMI ou tela MIPI de 10,1 polegadas (p. 17 a 19), mouse e teclado USB, cabo Ethernet.
4. **Armazenamento:** a placa tem slot microSD, soquete eMMC (ou eMMC soldada, conforme o modelo), e slot M.2 M-Key para NVMe. Se a imagem já estiver gravada na SPI + NVMe, o cartão não é necessário (p. 135).
5. **Serial (opcional):** para ver o boot sem monitor, use um adaptador USB para TTL de **3,3 V**, de preferência CH340. O manual pede velocidade de 1.500.000 bps (p. 136).
6. **Login padrão:** veja a parte 3, pergunta 8.
7. **Se reiniciar em loop,** troque a fonte ou o cabo USB-C antes de qualquer outra coisa (p. 135).

---

## Parte 3: respostas com base no manual v1.3

Todas as respostas usam o manual da **Orange Pi 5 Max** (v1.3). Não usei o manual da Orange Pi 5, 5 Plus ou 5 Pro.

### 1. A entrada USB-C negocia PD ou é 5 V fixo? O que o manual pede de fonte?

- **Resposta:** é **5 V fixo**, sem negociação PD. O manual diz que a interface Type-C "não suporta negociação PD e aceita somente tensão fixa de 5 V" (p. 18). Repete isso na p. 135 ("The Type-C power port does not support PD negotiation").
- **Fonte pedida:** recomendada **5 V/5 A** (p. 13, p. 15 e p. 18). A figura da p. 15 rotula a entrada como "Type-C/Power port (5V/5A)". Em outro trecho, a p. 135 e a p. 144 falam em **5 V/4 A**. Use uma fonte de pelo menos 4 A com saída de 5 V, e nunca acima de 5 V (p. 135).
- **Alimentação alternativa:** pelo pino 5 V do conector de 40 pinos, com cabo próprio. O manual diz para voltar à USB-C se a fonte estiver instável (p. 144).

- **Esquemático (esq. p. 14):** na entrada USB-C, os pinos CC1 e CC2 têm só resistores de 5,1 kΩ para o GND. Não há controlador PD, o que confirma o texto do manual. O bloco traz o rótulo "DC:5V 4A".

### 2. A RAM é LPDDR5 ou LPDDR4X na versão de 16 GB?

- **Resposta:** **LPDDR5**. O manual diz "4GB/8GB/16GB (LPDDR5)" (p. 11), e a tabela de hardware da p. 12 traz "4GB/8GB/16GB LPDDR5". A figura da p. 15 também indica "4GB/8GB/16GB LPDDR5". LPDDR4X não aparece no manual.

### 3. A SPI NOR é de 16 MB? Existe procedimento específico para gravar SPI + NVMe na Max?

- **SPI NOR:** sim, **16 MB QSPI NOR Flash** (p. 12). A figura da vista inferior (p. 15) mostra "16M QSPI Nor FLASH".
- **Procedimento SPI + NVMe:** sim, há seção específica, **cap. 2.6 (p. 53 a 63)**, com três métodos:
  - **RKDevTool (Windows), cap. 2.6.1 (p. 53 a 63):** NVMe instalado no slot M.2 antes de começar; placa sem fonte e sem cartão TF; cabo USB 2.0 macho-macho (p. 54). Carregar a configuração `rk3588_linux_pcie.cfg` e os arquivos `MiniLoaderAll.bin` e `rkspi_loader.img`, depois a imagem Linux, com a opção "force write by address" marcada (p. 59 a 62). Se falhar, apagar a SPI antes de tentar de novo (p. 62).
  - **dd no Linux, cap. 2.6.2 (p. 63):** método por linha de comando.
  - **balenaEtcher, cap. 2.6.3 (p. 69).**
- **Apagar a SPI:** cap. 2.14, p. 128, com RKDevTool.
- Também há a gravação em SPI + USB (cap. 2.7, p. 84). Não é o mesmo que NVMe.

### 4. Qual a ordem de boot? Como entrar no modo MaskROM?

- **Ordem de boot da BootROM:** o manual não traz, mas o esquemático traz. A tabela BOOT MODE CONFIG (esq. p. 6) associa cada tensão no pino `BOOT_SARADC_IN0` a uma ordem. Na Max, R170 é de 100K e R171 não é montado (DNP), então o pino fica em 1,8 V, o nível 7: **FSPI M2, FSPI M1, FSPI M0, eMMC, cartão SD, USB**. A leitura vem do texto do PDF do esquemático.
- **NVMe não está nessa lista.** Ele só dá boot por meio do bootloader gravado na SPI, o que explica o método SPI + NVMe do manual. O que o bootloader faz depois é outra etapa e depende da imagem gravada.
- **Botão BOOT:** liga o mesmo pino ao GND por 100 Ω (esq. p. 28). Com 0 V, a tabela indica "USB (Maskrom mode)". Por isso o botão força o modo MaskROM. A serigrafia da placa diz "BOOT", e o manual o chama de "MaskROM Key".
- **Outras fontes checadas:** o repositório u-boot-orangepi (ver 1.6) não mostra arquivo específico da Max na listagem visível. A página do Rockchip, https://opensource.rock-chips.com/wiki_Boot_option, não resolveu o nome de domínio (erro DNS), então não foi lida.
- **Modo MaskROM (cap. 2.6.1, p. 56 a 57, e procedimento igual em outros métodos):**
  1. Desligue a placa, sem fonte e sem cartão TF.
  2. Conecte o cabo USB 2.0 macho-macho entre a placa e o computador (porta USB da placa indicada na figura da p. 56).
  3. Segure o botão **MaskROM** da placa (p. 15 mostra a posição: "MaskROM Key", ao lado do conector "Fan (5V)").
  4. Conecte a fonte USB-C e ligue a placa.
  5. Solte o botão MaskROM.
  6. O RKDevTool deve mostrar "Found a MASKROM device" (p. 57).

### 5. O slot M.2 fica embaixo da placa? Aceita 2280? A placa traz parafuso e espaçador?

- **Lado:** **embaixo**. A figura "Bottom View" da p. 15 mostra o "M.2 M-Key (NVME PCIe3.0*4Lane)" na vista inferior, na borda esquerda da placa.
- **2280:** a mesma figura (p. 15) traz o rótulo "M.2 2280 Mounting PCB Nut". A porca fica na extremidade oposta ao slot. Pela escala da figura (cerca de 6 px por mm, usando os 89 mm da placa), ela fica a uns 80 mm do slot, coerente com um SSD de 80 mm. Essa medida é estimada na imagem, não é cota do fabricante. Confirme antes de comprar.
- **Porca:** o suporte do SSD é uma **porca de PCB** (PCB nut), que aparece na figura como parte da placa. Não aparece como espaçador avulso.
- **Parafuso e espaçador:** **não encontrados.** A lista de acessórios (p. 17 a 20) não cita parafuso, porca ou espaçador. A única menção é "insert the NVMe SSD... and secure it in place" (p. 53), sem detalhes de fixação. Busca na web por unboxing ou review da Orange Pi 5 Max não achou nada sobre o conteúdo da caixa. Os resultados eram de cases Pironman 5 Max, que é outro produto, e não servem para esta pergunta.

### 6. Qual a distância e o diâmetro dos furos de montagem? Medidas exatas da placa.

- **Diâmetro dos furos:** **2,7 mm**, "The diameter of the four positioning holes is 2.7mm" (p. 16). São quatro furos.
- **Distância entre furos:** não registrada. O manual não traz a cota, e o desenho mecânico oficial (seção 1.5) não tem cotas em texto.
- **Dimensões da placa:** **89 mm × 57 mm × 1,6 mm**, segundo a ficha oficial e o contorno do desenho mecânico. O texto da p. 13 do manual diz 56 mm, o que parece erro do manual, porque a figura da p. 15 mostra 57 mm. Peso de **62 g** na ficha (58 g na p. 13).

### 7. Conector da ventoinha: pinos, tensão, controle por PWM?

- **Conector:** "Fan (5V)" na vista superior (p. 15). É de **2 pinos, passo de 1,25 mm** (p. 19).
- **Tensão:** **5 V**, ventoinha de 5 V (p. 19).
- **Controle:** velocidade e liga/desliga pelo **PWM** da própria placa (p. 19).
- **Ordem dos pinos:** **pino 1 é + (5 V) e pino 2 é −**. O lado negativo é chaveado por um MOSFET (Q11), comandado pelo sinal `FAN_CTL_H` (esq. p. 11). Mesmo assim, confira o "+" na serigrafia antes de ligar.
- **Sinal de controle:** `FAN_CTL_H` fica em GPIO3_D1 / PWM9_M2 (esq. p. 10). O kernel mainline usa `pwm9` para a ventoinha (`rk3588-orangepi-5-compact.dtsi`).

### 8. Usuário e senha padrão da imagem oficial (só se o manual informar).

- **Informado pelo manual (p. 149), para as imagens Linux oficiais:**
  - Usuário `root`, senha `orangepi`.
  - Usuário `orangepi`, senha `orangepi`.
- O manual também diz que o login automático no terminal está ativado por padrão (p. 149), e que a senha é a mesma para ssh e para o sistema.
- Isso vale só para as imagens Linux da Orange Pi. Imagens Android e OpenWRT têm outras regras e não foram verificadas aqui.

---

## Divergências e pontos de atenção

- **Fonte:** a ficha e o manual (p. 13, 15 e 18) pedem 5 V/5 A. As p. 135 e 144 do manual e o esquemático (esq. p. 14) citam 5 V/4 A. Use 5 V e 5 A, que é o valor da ficha e atende aos dois.
- **Largura:** resolvida, são 57 mm (ficha e desenho mecânico). O texto da p. 13 diz 56 mm.
- **Peso:** a ficha diz 62 g, e o texto da p. 13 diz 58 g. A diferença é pequena e não foi medida.
- **Cabo para gravação:** as p. 20 e 54 do manual falam em USB macho-macho, e a p. 57 fala em "male to female". Use o macho-macho, que é o usado nas instruções detalhadas.
- **CAN0:** a imagem da p. 223 e o texto da p. 245 trocam TX e RX nos pinos 3 e 5. O esquemático confirma a imagem: pino 3 = RX, pino 5 = TX. Ver `07-gpio-e-pinout.md`.

---

## Lacunas

- **Distância entre os furos de montagem:** não registrada. Medir no desenho mecânico oficial (seção 1.5).
- **Parafuso e espaçador do M.2:** a figura da p. 15 mostra a porca soldada na placa para SSD 2280 ("M.2 2280 Mounting PCB Nut"). Nada confirma se o parafuso vem junto, nem a medida dele.
- **Polaridade do conector RTC:** o esquemático (esq. p. 11) mostra o conector de 2 pinos, mas o texto extraído não traz a numeração. Confira a serigrafia antes de ligar.

---

## Fontes

- Ficha oficial: http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/details/Orange-Pi-5-Max.html
- Página oficial de downloads: http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/service-and-support/Orange-Pi-5-Max.html
- Esquemático oficial V1.1: https://drive.google.com/file/d/1cAKUBHF2yXtmmLJaVtK17nOKoMlI02wA/view?usp=sharing
- Desenho mecânico oficial V1.1: https://drive.google.com/file/d/1HTPhgRq5EuVlTSzSiLY6-X320II8eArF/view?usp=sharing
- Device tree no kernel mainline: https://raw.githubusercontent.com/torvalds/linux/master/arch/arm64/boot/dts/rockchip/rk3588-orangepi-5-compact.dtsi
- https://macrogroup.ru/upload/iblock/7d7/v2e2xfzmvyofdk60kwwfixzqcefyi09a/OrangePi_5_Max_RK3588_User-Manual_v1.3.pdf (manual v1.3 lido, cópia de distribuidor)
- https://github.com/orangepi-xunlong (página da organização, repositórios fixados)
- https://github.com/orangepi-xunlong/orangepi-build (resumo do README)
- https://github.com/orangepi-xunlong/u-boot-orangepi (página do repositório, lista de diretórios)
- https://github.com/orangepi-xunlong/u-boot-orangepi/tree/v2017.09-rk3588/configs (listagem truncada; sem arquivo da Max visível)
- https://wiki.geekworm.com/N515 (página de case de terceiros; medidas do case; sem distância de furos da placa)
- https://circuitpython.org/blinka/orange_pi_5_max/ (página de terceiros; usada só para conferir que não há link oficial de manual nem esquemático; não é fonte primária)
