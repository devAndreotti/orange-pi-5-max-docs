# 04 - Software e primeiro boot

Este documento reúne o que foi possível levantar sobre sistemas operacionais, gravação de imagem, ordem de boot e primeiro acesso da Orange Pi 5 Max.

> Dados de sistemas, gravação e login vêm da ficha e da página oficial de downloads, lidas em 03/11/2026, e do manual v1.3. Itens marcados como **[não confirmado]** dependem de teste ou de fonte que não foi verificada.

## 1. Sistemas operacionais

A [ficha oficial](http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/details/Orange-Pi-5-Max.html) lista Orange Pi OS (Droid), Orange Pi OS (Arch), Orange Pi OS (OH), Ubuntu, Debian e Android 13. A [página de downloads](http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/service-and-support/Orange-Pi-5-Max.html) também oferece OpenWRT. Veja nela a data de cada imagem antes de baixar.

### Imagens genéricas ARM podem não funcionar

Uma imagem ARM genérica (por exemplo, uma imagem "para RK3588" ou "para qualquer ARM64") nem sempre traz os drivers da placa. Os pontos que costumam faltar são:

- Wi-Fi e Bluetooth, que dependem de módulo e firmware específicos;
- áudio e vídeo acelerado;
- a árvore de dispositivos (device tree) da Max, que difere de outras placas RK3588;
- NVMe e eMMC, em alguns casos.

Por isso, prefira imagens publicadas para a própria Orange Pi 5 Max ou para a família exata da placa.

### Relato de um caso concreto

Em um fórum da comunidade Armbian, um usuário relatou que a imagem da Orange Pi 5 comum rodou na Max, mas a imagem específica da Max não iniciou. Isso mostra que a variante importa e que o nome da placa não garante compatibilidade.

## 2. Onde baixar imagens

### Página oficial da Orange Pi

- [Página oficial de downloads](http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/service-and-support/Orange-Pi-5-Max.html): imagens de Ubuntu, Debian, Orange Pi OS, Android e OpenWRT, em pastas públicas do Google Drive. Os links de cada pasta estão em `13-manual-oficial-e-arquivos.md`.

### Armbian (comunidade)

- O Armbian lista a Orange Pi 5 Max na categoria de suporte **comunitário**, ou seja, não é testada oficialmente pelo projeto. A página da placa fica em `armbian.com/boards/orangepi5-max`.
- A página mostra imagens de Ubuntu 26.04 (com ambiente gráfico GNOME ou KDE) e de Debian 13 em versão mínima, com kernel "edge" e "vendor". As versões de kernel e as datas de build aparecem diferentes entre as traduções e entre as consultas, então confira a data antes de baixar.
- O Armbian também permite compilar a imagem por conta própria a partir do código-fonte. Isso exige bastante tempo e conhecimento de Linux embarcado.
- Em fóruns, há relatos de falha de boot com a imagem da Max e de Wi-Fi que não funciona no kernel mais novo, por falta de driver dkms compatível. Trate esses pontos como **[não confirmado]** para a versão atual.

### Outras fontes

- Tutoriais de terceiros (como o do Altium e o do Linux.org) indicam Ubuntu com kernel 6.1 para primeiro uso. São úteis como referência, mas podem estar desatualizados.

## 3. Como gravar a imagem

### Raspberry Pi Imager

**Não serve automaticamente.** O Raspberry Pi Imager foi feito para placas Raspberry Pi. Ele até permite escolher um arquivo de imagem próprio, mas não reconhece a Orange Pi 5 Max e não aplica as configurações de boot dessa placa. Use outra ferramenta.

### balenaEtcher

- Ferramenta gráfica para Windows, Linux e macOS. Grave o arquivo `.img` direto no microSD, com qualquer leitor de cartão, inclusive um leitor USB OTG. O manual cita balenaEtcher, RKDevTool e Win32DiskImager no Windows (p. 21 a 37).

### dd (Linux)

- Com a imagem descompactada, use `dd` apontando para o dispositivo de bloco do cartão (por exemplo `/dev/sdX`), não para uma partição. Confirme o dispositivo com `lsblk` antes, porque um dispositivo errado apaga outro disco.
- **[não confirmado]**: opções ideais de `bs`, `conv` e `status` para este caso. Os valores padrão costumam funcionar.

### Verificar a imagem

- Confira o hash SHA-256 publicado junto com a imagem, se existir, antes de gravar.
- Depois de gravar, faça a verificação da escrita (o Etcher faz isso por padrão; com `dd` use `sha256sum` sobre o arquivo e compare com a leitura do dispositivo nos mesmos bytes).
- Imagens sem hash publicado devem ser tratadas com mais cuidado: baixe só da fonte oficial ou de um projeto conhecido.

## 4. Ordem de boot

**Ordem da BootROM, segundo o esquemático oficial V1.1 (p. 6):** SPI (FSPI M2, M1, M0), depois eMMC, depois cartão SD, depois USB. O manual não traz essa ordem.

- **NVMe não está na lista.** Ele só dá boot por meio do bootloader gravado na SPI. O manual descreve esse método no cap. 2.6 (p. 53 a 63).
- **Botão BOOT:** ao ligar a placa com ele pressionado, ela entra em modo MaskROM e aguarda gravação pela USB (manual, p. 56 e 57; esquemático, p. 28).
- **Depois da BootROM:** quem decide o que iniciar é o bootloader encontrado. Ele vem da imagem gravada, então o comportamento entre microSD e NVMe pode mudar conforme a imagem. Se a SPI tiver um bootloader ruim, ela é lida primeiro. Para apagá-la, veja o cap. 2.14 do manual (p. 128) e `12-recuperacao-e-problemas.md`.

**Recomendação prática:** no primeiro boot, use só o microSD gravado, sem eMMC e sem NVMe com sistema. Depois de confirmar que funciona, passe para o eMMC ou o NVMe.

## 5. Primeiro login e SSH

**Login padrão das imagens Linux oficiais (manual, p. 149):** usuário `root` ou `orangepi`, ambos com a senha `orangepi`. O manual diz que o login automático no terminal vem ativado. Imagens de outros projetos, como o Armbian, têm regras próprias.

1. Ligue a placa com a fonte adequada (veja a seção de problemas comuns) e um cabo de rede ligado ao roteador. Se a imagem for de servidor, o acesso pelo teclado e pela tela HDMI pode não estar disponível.
2. Descubra o endereço IP da placa pelo painel do roteador, por ferramenta de rede (por exemplo, varredura na sub-rede) ou pelo nome de host padrão da imagem.
3. Conecte por SSH com o usuário inicial da imagem:
   `ssh orangepi@endereco-ip`
4. Troque a senha inicial logo no primeiro acesso.
5. Atualize o sistema e os pacotes da distribuição antes de qualquer outra configuração.
6. Para acesso sem senha, copie sua chave pública para o arquivo de chaves autorizadas da placa e, depois, desative o login por senha no SSH. Faça isso só depois de confirmar que a chave funciona em outra sessão.

**Troque a senha padrão antes de deixar a placa acessível pela rede.** Ela é pública, porque está no manual.

## 6. Problemas comuns

### Fonte de alimentação insuficiente

- O manual atribui a maioria das instabilidades de inicialização à fonte. Se a placa reiniciar em loop, troque a fonte ou o cabo USB-C primeiro (p. 135).
- A porta USB-C não negocia PD e aceita só 5 V fixos (p. 18 e 135). Use fonte com saída fixa de 5 V e 5 A. Nunca use saída acima de 5 V, porque queima a placa (p. 135).
- O significado das cores do LED está em `12-recuperacao-e-problemas.md`.

### Falta de drivers

- Sintomas comuns: Wi-Fi ausente, áudio sem saída, vídeo sem aceleração, NVMe não detectado.
- Causa mais provável: imagem genérica ou kernel que não tem o device tree da Max.
- Solução: usar imagem publicada para a Max e conferir a versão do kernel. Atualizar o kernel sem atualizar o device tree pode quebrar o que funcionava.

### Armbian sem boot ou sem Wi-Fi

- Relatos de fórum indicam falha de boot com a imagem da Max e falha de dkms do driver Broadcom em kernel 6.12.
- **[não confirmado]** para a versão atual. Antes de usar, veja a página da placa no Armbian e o fórum da placa.

### NVMe instável

- Sintomas: disco some após reboot, erros de I/O, sistema lento sob carga.
- Causas comuns: fonte fraca (queda de tensão sob carga do SSD), adaptador ou cabo ruim, e bootloader que não suporta o modelo do SSD.
- Teste: outra fonte, outro SSD, sistema só em microSD e NVMe apenas como dado.
- **[não confirmado]** quais modelos de SSD são compatíveis com a Max.

## Fontes

- Armbian, página da placa Orange Pi 5 Max: https://www.armbian.com/boards/orangepi5-max
- Armbian, página em inglês da placa (catálogo): https://www.armbian.com/orangepi-5-max/
- Fórum Armbian, tópico sobre builds da Orange Pi 5 Max: https://forum.armbian.com/topic/49131-orange-pi-5-max-builds/
- Fórum Armbian, tópico sobre boot por SD em Orange Pi 5 Max: https://forum.armbian.com/topic/51994-orange-pi-5-max-sd-card-booting-issue-system-normally-starts-from-nvme-ssd/
- Fórum Armbian, tópico sobre Debian na Max: https://forum.armbian.com/topic/44637-orange-pi-5-max-debian/
- Tutorial Altium, primeiros passos com Orange Pi 5 e RK3588: https://resources.altium.com/p/getting-started-orange-pi-5-and-rockchip-rk3588-processor
- Linux.org, instalação de Ubuntu na Orange Pi 5 Max com eMMC e NVMe: https://www.linux.org/threads/installing-ubuntu-on-the-orange-pi-5-max-with-emmc-and-nvme.58119/latest
- Fórum DietPi, boot por NVMe na Orange Pi 5: https://dietpi.com/forum/t/orange-pi-5-boot-from-nvme/16457?page=5
- Ficha oficial da Orange Pi 5 Max: http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/details/Orange-Pi-5-Max.html
- Página oficial de downloads: http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/service-and-support/Orange-Pi-5-Max.html
- Manual oficial v1.3: https://drive.google.com/drive/folders/1kzMRI95yaXLbQuK86fUbs92NJ6QOYIGO?usp=sharing
- Esquemático oficial V1.1: https://drive.google.com/file/d/1cAKUBHF2yXtmmLJaVtK17nOKoMlI02wA/view?usp=sharing
