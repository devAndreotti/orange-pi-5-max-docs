# Orange Pi 5 Max: alimentação, refrigeração e armazenamento

Este documento cobre três pontos para colocar a Orange Pi 5 Max para funcionar: como alimentá-la, como resfriá-la e onde guardar o sistema. Cada afirmação marca se é confirmada (documentação ou fonte que a sustenta) ou se é relato (experiência de usuário ou de revendedor, sem verificação independente).

## 1. Alimentação

**Entrada:** USB-C, com recomendação de 5 V / 5 A (até 5 A).

- Revendedores indicam que a versão de 16 GB exige um adaptador de pelo menos 5 A. Isso é uma declaração de revendedor, não uma especificação oficial encontrada. Confirmado como recomendação de venda; não confirmado como requisito técnico da placa.
- Um anúncio de fonte 5 V / 5 A vendido para a placa aparece em listagens de terceiros. A mesma listagem descreve o produto como "5V4A" no texto, o que mostra que nomes de produto são pouco confiáveis; leia a especificação impressa na fonte.

**Por que fonte de 2 A ou de celular não serve:** a placa pode puxar mais corrente que 2 A em carga alta (CPU, NVMe, rede e periféricos juntos). Com corrente insuficiente, a placa pode reiniciar, travar ou limitar USB. Carregadores de celular e power banks comuns raramente entregam 5 A em 5 V. Carregadores USB-PD comuns entregam potência em tensão mais alta (9 V, 12 V, 15 V ou 20 V) e só chegam a 5 V depois de conversão; por isso um carregador "de 27 W" nem sempre é 5 V / 5 A.

**O que procurar em uma fonte:**

- Saída de 5,1 V ou 5 V a 5 A, escrita na etiqueta da fonte.
- Cabo USB-C que suporte 5 A (cabo com fios finos e sem marca de corrente pode limitar a corrente mesmo com fonte boa).
- Preferir fontes com certificação conhecida e que declarem a corrente em 5 V, não só a potência total em watts.

**Suporte a USB-PD (dúvida):** há relato de que a placa não usa negociação USB-PD. Esta informação não foi confirmada por documentação oficial encontrada. Na prática, se a fonte precisar negociar um perfil PD para chegar a 5 A, a placa pode não conseguir pedir esse perfil. Teste com a fonte escolhida antes de confiar nela para carga contínua.

Referência de contexto: a Raspberry Pi 5 tem comportamento parecido (sem 5 A em 5 V, o sistema limita a corrente das portas USB a 600 mA). Isso é da Raspberry Pi, não da Orange Pi; não há confirmação de que a Orange Pi 5 Max faça o mesmo.

## 2. Refrigeração

**Conector de ventoinha:** conector de 5 V, 2 pinos, passo de 1,25 mm.

- Relato (fórum Armbian): um usuário com Orange Pi 5 Max disse que uma ventoinha ligada ao conector de 1,25 mm não respondeu em nenhuma carga de CPU. Não confirmado; pode ser fiação, polaridade ou configuração de software.
- Não encontrei pinagem oficial confirmada para esse conector nesta pesquisa. Antes de ligar, confira o esquema oficial da placa. Não ligue ventoinha de 2 fios diretamente em pino GPIO: há relatos de que a corrente é alta demais para o pino.
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

**microSD (boot):** o caminho mais simples para o primeiro boot. Use cartão de marca conhecida e classe rápida (A1/A2 ou superior). É o caminho de boot que as imagens oficiais costumam usar.

**eMMC:** a placa tem um socket para módulo eMMC. Módulos de 32, 64, 128 e 256 GB existem. Há divergência entre fontes sobre se a eMMC é soldada ou em socket em todas as variantes; uma fonte diz que há socket com opção de chip, outra diz que algumas variantes já vêm com eMMC instalada. Confira a variante exata antes de comprar.

**SSD NVMe M.2:**

- Slot M.2 M-key com PCIe 3.0 x4 (Gen3). Isso vem de análises e do anúncio do fabricante.
- Um SSD Gen4 funciona, mas fica limitado a Gen3 nesse slot.
- Não encontrei confirmação de que o slot aceite SSD de tamanho 2280 especificamente. Verifique a especificação oficial antes de comprar.
- Há um aviso geral de que muitas imagens não iniciam direto de NVMe em placas Orange Pi 5 (guia de James Chambers, sobre a placa base). Não verificado para a Max; pode exigir boot por microSD ou eMMC primeiro.

**Relato de travamento sob carga:** um usuário relatou que um SSD NVMe em uma Orange Pi 5+ travava quando o uso ficava intenso, e que o mesmo SSD funcionava sem falhas via USB. Não é relato direto da Max e não está confirmado. Em fóruns de NVMe em Raspberry Pi, há relatos de quedas de controlador ligadas a corrente insuficiente no pico. Se acontecer, teste outro SSD, verifique a fonte e procure no dmesg a mensagem de reset do controlador NVMe.

## 4. Lista mínima para o primeiro boot

1. **Fonte** de 5 V / 5 A com cabo USB-C que suporte 5 A.
2. **Armazenamento para o sistema:** cartão microSD de boa marca, ou módulo eMMC se a variante tiver socket.
3. **Refrigeração:** case com ventoinha (N515 ou similar) ou, no mínimo, dissipador. Para carga contínua, use ventoinha.
4. **Para administrar sem monitor:**
   - Cabo de rede. A placa tem porta de 2,5 Gb, então um switch ou roteador com 2,5 Gb evita gargalo, mas um cabo Gigabit também funciona para o primeiro boot.
   - Um computador para gravar a imagem no cartão ou eMMC (leitor de cartão ou adaptador, e ferramenta de gravação como balenaEtcher ou Rufus).
   - Acesso por SSH a partir do computador, depois de descobrir o IP da placa no roteador.

## Fontes

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
