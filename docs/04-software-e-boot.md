# 04 - Software e primeiro boot

Este documento reúne o que foi possível levantar sobre sistemas operacionais, gravação de imagem, ordem de boot e primeiro acesso da Orange Pi 5 Max.

> **Aviso de confirmação.** O site oficial da Orange Pi não respondeu durante a pesquisa (conexão recusada), então a lista de sistemas do fabricante abaixo vem de outras fontes e precisa ser conferida na página de suporte da placa. Itens marcados como **[não confirmado]** dependem de teste ou de fonte que não foi verificada.

## 1. Sistemas operacionais

O fabricante costuma listar, para a Orange Pi 5 Max, as seguintes opções:

- Orange Pi OS (variante baseada em Arch)
- Orange Pi OS (variante Droid)
- Arch Linux
- OH (OpenHarmony)
- Android 13
- Ubuntu
- Debian
- OpenWRT

**Status desta lista:** [não confirmado] na página oficial. A lista foi informada como "listada pelo fabricante" no pedido, mas a página do fabricante não pôde ser acessada nesta pesquisa. Antes de escolher um sistema, confira na página de suporte da Max quais imagens existem, em que data foram publicadas e para qual versão de kernel.

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

- A página de suporte da placa, no site orangepi.org, é o lugar indicado para as imagens oficiais. Um tutorial de terceiros aponta essa página como fonte das imagens. Não foi possível abri-la nesta pesquisa.
- **[não confirmado]**: os links exatos de cada imagem e os sistemas disponíveis na data de hoje.

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

- Ferramenta gráfica para Windows, Linux e macOS. Grave o arquivo `.img` (ou `.raw`) direto no microSD ou em um leitor USB.
- **[não confirmado nesta pesquisa]**: versão atual e URL oficial. Confira no site da balena antes de instalar.

### dd (Linux)

- Com a imagem descompactada, use `dd` apontando para o dispositivo de bloco do cartão (por exemplo `/dev/sdX`), não para uma partição. Confirme o dispositivo com `lsblk` antes, porque um dispositivo errado apaga outro disco.
- **[não confirmado]**: opções ideais de `bs`, `conv` e `status` para este caso. Os valores padrão costumam funcionar.

### Verificar a imagem

- Confira o hash SHA-256 publicado junto com a imagem, se existir, antes de gravar.
- Depois de gravar, faça a verificação da escrita (o Etcher faz isso por padrão; com `dd` use `sha256sum` sobre o arquivo e compare com a leitura do dispositivo nos mesmos bytes).
- Imagens sem hash publicado devem ser tratadas com mais cuidado: baixe só da fonte oficial ou de um projeto conhecido.

## 4. Ordem de boot

Há mais de uma origem possível de boot na placa: microSD, eMMC (se instalada) e NVMe. A prioridade exata da Orange Pi 5 Max não foi confirmada.

- As fontes pesquisadas se contradizem. Um tutorial da Orange Pi 5 comum cita uma ordem (SD, NVMe, eMMC) e outro cita outra (eMMC antes do microSD).
- Em placas RK3588 parecidas, o bootloader gravado na memória SPI pode mudar a prioridade. Um relato em fórum mostra que, após atualizar o SPI, a placa passou a iniciar pelo NVMe.
- Em outra placa RK3588 (Rock 5B), a prioridade relatada foi microSD, depois eMMC, depois SSD M.2.
- Algumas imagens só iniciam pelo microSD e aceitam apenas a partição raiz no NVMe. Isso foi relatado para a Orange Pi 5 Pro, sem SPI.

**Recomendação prática:** **[não confirmado]** para a Max. Para evitar surpresas:

1. Teste com um único dispositivo de boot por vez (só o microSD gravado, sem eMMC e sem NVMe com sistema).
2. Só depois de confirmar o boot pelo microSD, teste o eMMC ou o NVMe.
3. Anote a versão do bootloader gravado no SPI, se houver.

## 5. Primeiro login e SSH

Os passos abaixo são os padrões comuns em imagens Linux para placas ARM. **[não confirmado]** para a imagem específica que você escolher; o nome de usuário e a senha inicial mudam conforme a distribuição e devem ser lidos na documentação da imagem.

1. Ligue a placa com a fonte adequada (veja a seção de problemas comuns) e um cabo de rede ligado ao roteador. Se a imagem for de servidor, o acesso pelo teclado e pela tela HDMI pode não estar disponível.
2. Descubra o endereço IP da placa pelo painel do roteador, por ferramenta de rede (por exemplo, varredura na sub-rede) ou pelo nome de host padrão da imagem.
3. Conecte por SSH com o usuário inicial da imagem:
   `ssh usuario@endereco-ip`
4. Troque a senha inicial logo no primeiro acesso.
5. Atualize o sistema e os pacotes da distribuição antes de qualquer outra configuração.
6. Para acesso sem senha, copie sua chave pública para o arquivo de chaves autorizadas da placa e, depois, desative o login por senha no SSH. Faça isso só depois de confirmar que a chave funciona em outra sessão.

**Não grave senhas reais nesta documentação.** Use sempre a senha que você define no primeiro acesso.

## 6. Problemas comuns

### Fonte de alimentação insuficiente: LED vermelho e sem boot

- Sinal típico: LED vermelho aceso e nenhuma atividade de boot (sem vídeo, sem rede, sem LED de atividade).
- A especificação de alimentação informada por usuários da Orange Pi 5 é de 5 V e 4 A. Fontes de 5 V e 2 A costumam ser fracas demais para o conjunto, com NVMe ou várias portas em uso.
- A placa não negocia tensões mais altas por USB-PD, então uma fonte PD serve, mas em 5 V fixos.
- **[não confirmado]** que o LED vermelho significa exatamente falta de energia nessa placa. Muitas fontes de informação sobre LED vermelho são de Raspberry Pi 5, que tem outro circuito de sinalização.
- Antes de culpar a imagem, teste com fonte de 5 V e 4 A ou mais, de boa qualidade, com cabo USB-C adequado.

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
- Página oficial da Orange Pi 5 Max (não acessível durante a pesquisa): http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/details/Orange-Pi-5-Max.html
