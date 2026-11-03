# SSD NVMe na Orange Pi 5 Max

## O que o slot aceita

- Conector **M.2 M-Key**, tamanho **2280** (22 mm × 80 mm), embaixo da placa. A figura da p. 15 do manual marca o ponto de fixação "M.2 2280".
- Interface **PCIe 3.0 x4** (Gen3, 4 vias). Só **NVMe**. SSD M.2 SATA (anúncios com "SATA" ou "NGFF") não funciona.
- SSD Gen4 funciona, mas roda na velocidade do Gen3.

## O que "limita a Gen3" quer dizer

O PCIe 3.0 x4 tem teto teórico perto de 3,9 GB/s. Em placas com o mesmo chip RK3588 (Rock 5B), as medições publicadas ficam entre 2,5 e 3 GB/s de leitura sequencial, dependendo do SSD, do kernel e de como o teste é feito. Um SSD Gen4 anunciado com 5.000 a 7.400 MB/s não passa desse teto nesta placa.

Na própria Max, a medição publicada por Jeff Geerling (sbc-reviews, issue 49) deu **1,04 GB/s de leitura e 2,26 GB/s de escrita sequencial** com iozone em blocos de 1 MB. A leitura ficou bem abaixo do teto, e o relatório não informa o link PCIe negociado. Trate 1 a 3 GB/s como a faixa realista.

Para servidor acessado pela rede, o limite real costuma ser a rede: a porta de 2,5 Gb/s transfere cerca de 280 MB/s. Qualquer NVMe Gen3 já passa disso.

A review do tinycomputers.io cita 5,7 GB/s de leitura na Max. O número está acima do teto do PCIe 3.0 x4 e provavelmente reflete cache; não use como referência.

## Como escolher

- Gen3 de marca conhecida, de preferência sem DRAM: mais barato e costuma consumir menos, o que ajuda com a fonte de 5 V / 5 A.
- 512 GB costuma sair mais barato por GB que 256 GB.
- O SSD é opcional. O sistema roda no microSD ou eMMC, e o SSD pode vir depois.

Não há lista oficial de SSDs testados na Max. Os relatos de SSD não detectado em placas RK3588 apontam mais para versão de kernel e device tree do que para a marca do SSD. Antes de culpar o SSD, teste outra imagem ou kernel e reencaixe o SSD com a placa desligada.

## Boot pelo NVMe

A placa tem memória **SPI NOR de 16 MB**, onde fica o bootloader (ficha oficial). O NVMe não está na ordem de boot da BootROM (esquemático V1.1, p. 6), por isso precisa desse bootloader. O manual da Max descreve três métodos no cap. 2.6 (p. 53 a 63): RKDevTool, `dd` e balenaEtcher. O caminho usado pela comunidade em placas Orange Pi 5:

1. Ligar pelo microSD com a imagem oficial.
2. Gravar o bootloader na SPI pela ferramenta de configuração da imagem (`orangepi-config` ou `armbian-config`).
3. Instalar o sistema no NVMe e desligar.
4. Tirar o microSD e ligar.

Leia o cap. 2.6 do manual antes de gravar. Para apagar a SPI, veja o cap. 2.14 (p. 128). Guarde um microSD com o sistema para recuperação: há relatos de atualização de bootloader ou de kernel que quebraram o boot pelo NVMe.

## Preços de referência (Mercado Livre, 01/11/2026)

Preço à vista na página de busca, sem abrir o anúncio. Mudam rápido.

| SSD | Capacidade | Geração | Preço (R$) |
|---|---|---|---|
| Patriot P300 | 512 GB | Gen3 x4 | 639,90 |
| Kingston NV2 (Amazon) | 500 GB | Gen3 x4 | 645,05 |
| Patriot P320 | 512 GB | Gen3 x4 | 680,00 |
| Samsung PM981 (OEM) | 512 GB | Gen3 x4 | 690,00 |
| Netac NV2000 | 512 GB | Gen3 x4 | 741,00 |
| Patriot P300 | 256 GB | Gen3 x4 | 413,00 |
| Netac | 256 GB | Gen3 | 419,90 |
| Adata Legend 710 | 256 GB | Gen3 x4 | 498,99 |

Gen4 de 512 GB custava de R$ 750 a R$ 1.100 na mesma busca.

## Fontes

- Página oficial: http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/details/Orange-Pi-5-Max.html
- CNX Software, lançamento da Max: https://www.cnx-software.com/2024/08/01/rockchip-rk3588-powered-orange-pi-5-max-sbc-features-up-to-16gb-lpddr5-2-5gbe-onboard-wifi-6e-and-bluetooth-5-3/
- Review da Max (tinycomputers.io): https://tinycomputers.io/posts/rk3588-orange-pi-5-max-review.html
- Jeff Geerling, Rock 5 B (RK3588, NVMe ~3 GB/s): https://www.jeffgeerling.com/blog/2023/rock-5-b-not-raspberry-pi-killer-yet/
- Jeff Geerling, medições da Max (sbc-reviews, issue 49): https://github.com/geerlingguy/sbc-reviews/issues/49
- Radxa, teste de NVMe na Rock 5B: https://forum.radxa.com/t/rock-5b-nvme-speed-test/11466
- Armbian, NVMe não detectado na Orange Pi 5: https://forum.armbian.com/topic/49225-orange-pi-5-unable-to-detect-nvme-drive/
- Boot por NVMe na Orange Pi 5 (guia da comunidade): https://jamesachambers.com/orange-pi-5-ssd-boot-guide/
- Manual oficial da Max v1.3: https://drive.google.com/drive/folders/1kzMRI95yaXLbQuK86fUbs92NJ6QOYIGO?usp=sharing
- Esquemático oficial V1.1: https://drive.google.com/file/d/1cAKUBHF2yXtmmLJaVtK17nOKoMlI02wA/view?usp=sharing
