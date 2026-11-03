# Orange Pi 5 Max: alimentação, refrigeração e armazenamento

Este documento cobre três pontos para colocar a Orange Pi 5 Max para funcionar: como alimentá-la, como resfriá-la e onde guardar o sistema. Cada afirmação marca se é confirmada (documentação ou fonte que a sustenta) ou se é relato (experiência de usuário ou de revendedor, sem verificação independente).

## 1. Alimentação

**Entrada:** USB-C, **5 V fixos**, 5 A, sem negociação USB-PD. Confirmado pela [ficha oficial](http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/details/Orange-Pi-5-Max.html), pelo manual v1.3 (p. 13 e 18) e pelo esquemático V1.1 (p. 14).

- **Sem PD.** O manual diz que a porta não negocia PD e só aceita 5 V fixos (p. 18 e 135). O esquemático confirma: os pinos CC1 e CC2 têm só resistores de 5,1 kΩ para o GND, sem controlador PD.
- **Nunca use saída acima de 5 V.** O manual avisa que isso queima a placa (p. 135).
- **Não alimente pela USB de um computador** (p. 136).
- **Reinício em loop:** troque a fonte ou o cabo USB-C antes de qualquer outra coisa (p. 135).
- **4 A ou 5 A:** a ficha e parte do manual pedem 5 A. As p. 135 e 144 e o esquemático citam 4 A. Use 5 A.

**Por que carregador de celular ou PD não serve:** sem negociação PD, o padrão USB-C oferece no máximo 3 A em 5 V. Um carregador "de 27 W" ou "de 65 W" usa PD para chegar nessa potência, em geral em tensões mais altas. Isso inclui as fontes de 27 W para Raspberry Pi 5, que usam PD para entregar 5 A. A placa até pode ligar nelas, mas fica limitada a cerca de 3 A e pode reiniciar sob carga. Essa conclusão é dedução a partir do padrão USB-C e do esquemático; não encontrei um teste publicado na Max.

**O que procurar em uma fonte:**

- Saída fixa de 5 V e 5 A, escrita na etiqueta da fonte. Nomes de produto enganam: uma listagem da fonte oficial diz "5V4A" e "5V 5A" no mesmo anúncio.
- Cabo USB-C que suporte 5 A. Cabo com fios finos limita a corrente mesmo com fonte boa.
- Marca com certificação conhecida.

## 2. Refrigeração

**Conector de ventoinha:** conector de 5 V, 2 pinos, passo de 1,25 mm.

- **Oficial (esquemático V1.1, p. 10 e 11):** pino 1 é + (5 V) e pino 2 é −. O lado negativo é chaveado por um MOSFET, comandado pelo sinal `FAN_CTL_H`, que fica em GPIO3_D1 / PWM9_M2. O manual diz que a velocidade é controlada por PWM (p. 19). Mesmo assim, confira o "+" na serigrafia antes de ligar.
- Relato (fórum Armbian): um usuário com Orange Pi 5 Max disse que uma ventoinha ligada ao conector de 1,25 mm não respondeu em nenhuma carga de CPU. Não confirmado; pode ser fiação, polaridade ou configuração de software.
- Não ligue ventoinha em pino de sinal do conector de 40 pinos: esses pinos trabalham em 3,3 V e não foram feitos para alimentar motor. Os pinos de energia de 5 V e GND do mesmo conector servem para isso, e a placa também tem um par "5V Out" ao lado dele (manual, p. 15). Ventoinha ligada direto em 5 V fica sempre ligada, sem controle de velocidade.
- Conector de 1,25 mm (PicoBlade, 2 pinos) existe em ventoinhas miniatura comuns. Confira o encaixe antes de comprar.

**Quando um cooler é necessário:**

- Carga contínua (compilação longa, IA, transcodificação, servidor 24/7): sim, quase sempre.
- Uso leve (navegação, SSH, rede): o dissipador pode bastar, mas a temperatura sobe em carga prolongada.
- Ambiente quente ou case fechado: sim.
- Relato de usuário de Orange Pi 5 Max em carga com ventoinha PWM de 40 x 40 x 10 mm nos pinos GPIO 4 (5 V), 6 (GND) e 7 (PWM): temperatura da CPU perto de 70 °C sob carga. Relato, não medição verificada.

**Cases declarados como compatíveis:**

- **Geekworm N515** (vendido também como eleUniverse em algumas lojas): a página do fabricante diz que o case é projetado para Orange Pi 5 Pro, 5 Max e 5 Ultra, e que foi testado e é compatível com a 5 Max. O cooler de CPU do N515 é um pouco menor na Max, porque o case foi lançado antes da placa. Isso é declaração do fabricante.
- O kit inclui uma ventoinha 5 V (modelo 3010) e quatro dissipadores de alumínio.
- Outros cases de terceiros existem para o layout da Pi 5 Pro/N515; confirme as medidas de furação e de altura antes de comprar.

**Aviso:** o cooler da N515 é um pouco menor na Max. Na prática, pode cobrir menos área de dissipação; se a temperatura em carga for alta, considere dissipador maior ou ventoinha mais forte.

## 3. Armazenamento

**microSD (boot):** o caminho mais simples para o primeiro boot. O manual pede no mínimo 16 GB, de preferência 32 GB ou mais, classe 10 (p. 17).

**eMMC:** a versão padrão tem soquete para módulo eMMC de 32, 64, 128 ou 256 GB. A eMMC soldada é opção sob encomenda (ficha oficial).

**SSD NVMe M.2:**

- Slot M.2 M-key com PCIe 3.0 x4 (Gen3). Isso vem de análises e do anúncio do fabricante.
- Um SSD Gen4 funciona, mas fica limitado a Gen3 nesse slot.
- O slot aceita SSD 2280: a figura da p. 15 do manual marca o ponto de fixação "M.2 2280 Mounting PCB Nut", e o slot fica embaixo da placa.
- SSD M.2 SATA não funciona: o slot é só NVMe.
- O NVMe não está na ordem de boot da BootROM (esquemático V1.1, p. 6). Para dar boot por ele, é preciso gravar o bootloader na SPI. O manual descreve esse método no cap. 2.6 (p. 53 a 63).

**Relato de travamento sob carga:** um usuário relatou que um SSD NVMe em uma Orange Pi 5+ travava quando o uso ficava intenso, e que o mesmo SSD funcionava sem falhas via USB. Não é relato direto da Max e não está confirmado. Em fóruns de NVMe em Raspberry Pi, há relatos de quedas de controlador ligadas a corrente insuficiente no pico. Se acontecer, teste outro SSD, verifique a fonte e procure no dmesg a mensagem de reset do controlador NVMe.

## 4. Lista mínima para o primeiro boot

1. **Fonte** com saída fixa de 5 V / 5 A e cabo USB-C que suporte 5 A. Carregador PD de celular não serve.
2. **Armazenamento para o sistema:** cartão microSD de boa marca, ou módulo eMMC se a variante tiver socket.
3. **Refrigeração:** case com ventoinha (N515 ou similar) ou, no mínimo, dissipador. Para carga contínua, use ventoinha.
4. **Para administrar sem monitor:**
   - Cabo de rede. A placa tem porta de 2,5 Gb, então um switch ou roteador com 2,5 Gb evita gargalo, mas um cabo Gigabit também funciona para o primeiro boot.
   - Um computador para gravar a imagem no cartão ou eMMC (leitor de cartão ou adaptador, e ferramenta de gravação como balenaEtcher ou Rufus).
   - Acesso por SSH a partir do computador, depois de descobrir o IP da placa no roteador.

## Fontes

- Ficha oficial: http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/details/Orange-Pi-5-Max.html
- Manual oficial v1.3: https://drive.google.com/drive/folders/1kzMRI95yaXLbQuK86fUbs92NJ6QOYIGO?usp=sharing
- Esquemático oficial V1.1: https://drive.google.com/file/d/1cAKUBHF2yXtmmLJaVtK17nOKoMlI02wA/view?usp=sharing
- Tinytronics, Orange Pi 5 Max 16 GB (requisito de adaptador de 5 A declarado pelo revendedor): https://www.tinytronics.nl/en/development-boards/single-board-computers/orange-pi-5-max-rk3588-16gb
- Newegg, listagem de fonte 5 V / 5 A para Orange Pi 5 Max: https://www.newegg.com/p/2S7-09MK-007U9
- Armbian Forum, relato de ventoinha no conector de 1,25 mm da Orange Pi 5 Max: https://forum.armbian.com/topic/50442-pwm-fan-connector-not-working-on-orange-pi-5-max
- Armbian Forum, configuração de PWM da Orange Pi 5 Max: https://forum.armbian.com/topic/60076-pwm-fan-configuration-on-orange-pi-5-max-running-armbian-debian-minimal/
- Armbian Forum, controle de temperatura e ventoinha na Orange Pi 5: https://forum.armbian.com/topic/29253-temperature-and-fan-control-on-opi5/
- Geekworm, N515 (compatibilidade declarada com Orange Pi 5 Max e nota sobre o cooler menor): https://geekworm.com/en-au/products/n515
- Notebookcheck, análise da Orange Pi 5 Max (eMMC e PCIe 3.0 x4): https://www.notebookcheck.com/Orange-Pi-5-Max-Neuer-Einplatinenrechner-ist-eine-starke-Alternative-zum-Raspberry-Pi-5-und-sticht-diesen-mehrmals-aus.864435.0.html
- James Chambers, guia de boot por SSD em Orange Pi 5 (placa base, não Max): https://jamesachambers.com/orange-pi-5-ssd-boot-guide/
- DietPi Forum, fonte regulada de 5 V para SBCs (contexto sobre conversão USB-PD para 5 V): https://dietpi.com/forum/t/alternative-high-current-regulated-5vdc-power-supply-for-sbcs/19914
- SunFounder Forum, relatos de corrente insuficiente em HAT NVMe (Raspberry Pi 5, contexto de Pi): https://forum.sunfounder.com/t/problem-with-2xnvme-hat-in-pironman-5-max/4264
