# 02 - Hardware da Orange Pi 5 Max

Este documento descreve o hardware da Orange Pi 5 Max, uma placa de desenvolvimento baseada no SoC Rockchip RK3588. Os dados vêm de fontes públicas listadas na seção "Fontes". A página oficial da Orange Pi era a fonte principal planejada, mas não respondeu durante a pesquisa (conexão recusada), então os valores abaixo foram conferidos em artigos de imprensa técnica. Antes de comprar ou projetar em cima destes números, confirme-os na página oficial.

## Especificações técnicas

| Item | Valor informado |
|---|---|
| SoC | Rockchip RK3588 |
| CPU | 4x Cortex-A76 (até 2,4 GHz) + 4x Cortex-A55 (até 1,8 GHz) |
| GPU | Mali-G610 MP4 (OpenGL ES 3.2, OpenCL 2.2, Vulkan 1.2) |
| NPU | 6 TOPS |
| Memória RAM | LPDDR5, 4, 8 ou 16 GB (ver divergências) |
| Ethernet | 1x 2.5GbE, chip RTL8125BG |
| Wi-Fi / Bluetooth | Módulo AP6611: Wi-Fi 6E e Bluetooth 5.3 |
| Vídeo | 2x HDMI 2.1, até 8K a 60 Hz, com eARC |
| USB | 2x USB 3.0 Type-A e 2x USB 2.0 Type-A |
| M.2 | Slot M-Key para NVMe PCIe 3.0 x4 (formato 2280) |
| Armazenamento | Leitor microSD; socket para eMMC (32 a 256 GB) ou eMMC soldada, conforme a versão |
| GPIO | Header de 40 pinos (GPIO, UART, I2C, SPI, CAN, PWM) |
| Alimentação | Entrada USB Type-C, 5 V / 5 A |
| Dimensões | 89 x 57 mm |
| Peso | cerca de 62 g |

Pontos de atenção para quem vai usar a placa:

- O único slot de armazenamento rápido é o M.2 M-Key. O microSD é lento para sistema operacional e serve mais para boot ou dados.
- A placa tem uma porta de Ethernet, diferente de modelos anteriores com duas.
- Não há entrada de vídeo HDMI, só saídas.

## Diferenças em relação às outras placas da linha

Os modelos Orange Pi 5 Plus e Orange Pi 5 Pro têm chips e conjuntos de portas diferentes. A Max é descrita como uma versão mais compacta e mais barata da Plus, com Wi-Fi integrado e memória de maior largura de banda.

| Item | Orange Pi 5 Max | Orange Pi 5 Plus | Orange Pi 5 Pro |
|---|---|---|---|
| SoC | RK3588 | RK3588 | RK3588S |
| RAM | LPDDR5, 4/8/16 GB | LPDDR4/LPDDR4X, 4/8/16 GB (versão V2.1 indicada) | LPDDR5, até 16 GB |
| Ethernet | 1x 2.5GbE | 2x 2.5GbE | 1x 1GbE |
| Wi-Fi / BT | Wi-Fi 6E, BT 5.3 integrados | Sem Wi-Fi fixo; slot M.2 E-Key para módulo sem fio | Wi-Fi 5, BT 5.0 (módulo AP6256) |
| Vídeo | 2x HDMI 2.1 (saída) | HDMI com entrada (relatada em review); número de saídas não confirmado | 1x HDMI 2.1 e 1x HDMI 2.0 |
| USB | 2x USB 3.0 + 2x USB 2.0 | Mais portas USB que a Max | 1x USB 3.1 + 3x USB 2.0 |
| M.2 | NVMe PCIe 3.0 x4 | NVMe PCIe 3.0 x4 | M.2 2280 com PCIe 2.0 x1 (NVMe ou SATA) |
| eMMC | Socket ou soldada, conforme a versão | Opcional | Opcional |
| Dimensões | 89 x 57 mm | 100 x 75 mm | 89 x 56 mm |
| Alimentação | USB-C 5 V / 5 A | 5 V / 4 A | USB-C 5 V / 5 A |

Resumo prático: a Max é a opção com Wi-Fi 6E integrado e PCIe 3.0 x4 no menor formato da linha. Quem precisa de duas portas 2.5GbE ou de entrada HDMI deve olhar a Plus. Quem quer Wi-Fi integrado e custo menor, sem precisar de alta velocidade de rede, pode olhar a Pro, que tem Wi-Fi 5 e Gigabit.

## Divergências entre fontes

- **Tipo de memória RAM.** Não encontrei nenhuma fonte que diga LPDDR4X para a Max. As reviews e notícias de lançamento consultadas dizem LPDDR5 (4, 8 ou 16 GB). Uma fonte chinesa citada em buscas dizia DDR5 para a Pro e a Max, mas os artigos de imprensa dizem LPDDR5. Vale registrar que o tipo de memória deve ser confirmado na etiqueta da placa ou no datasheet do lote que você comprar.
- **Memória da Plus.** A comparação OpenELab indica LPDDR4/LPDDR4X (versão V2.1 da placa). Não confirmei se todas as revisões da Plus usam o mesmo tipo.
- **Página oficial.** A página oficial da Orange Pi não respondeu durante a pesquisa. Os dados da Max vêm de artigos de imprensa. Peso e dimensões aparecem em uma única fonte no caso do peso (62 g), então trate esse valor como aproximado.

## Fontes

- Página oficial da Orange Pi 5 Max (fonte principal planejada; não acessível na pesquisa): http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/details/Orange-Pi-5-Max.html
- CNX Software, "Rockchip RK3588-powered Orange Pi 5 Max SBC features up to 16GB LPDDR5, 2.5GbE, onboard WiFi 6E and Bluetooth 5.3" (2024-08-01): https://www.cnx-software.com/2024/08/01/rockchip-rk3588-powered-orange-pi-5-max-sbc-features-up-to-16gb-lpddr5-2-5gbe-onboard-wifi-6e-and-bluetooth-5-3/
- Liliputing, "Orange Pi 5 Max is a credit card sized RK3588 PC with 2.5 GbE Ethernet, PCIe 3.0 and WiFi 6E": https://liliputing.com/orange-pi-5-max-is-a-credit-card-sized-rk3588-pc-with-2-5-gbe-ethernet-pcie-3-0-and-wifi-6e/
- Liliputing, "Orange Pi 5 Pro is a Raspberry Pi sized PC with an RK3588S chip, LPDDR5 memory and a PCIe 2.0 socket": https://liliputing.com/orange-pi-5-pro-is-a-raspberry-pi-sized-pc-with-an-rk3588s-chip-lpddr5-memory-and-a-pcie-2-0-socket/
- OpenELab, "Orange Pi 5 Plus vs Orange Pi 5 Pro vs Orange Pi 5: Which RK3588 SBC Should You Buy?": https://openelab.io/blogs/learn/orange-pi-5-plus-vs-5-pro-vs-5
