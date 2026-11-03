# Orange Pi 5 Max: documentação reunida

Documentação em português sobre a Orange Pi 5 Max 16 GB (SoC Rockchip RK3588): especificações, alimentação, refrigeração, armazenamento, sistemas e links úteis. Feita para ser lida ou importada em ferramentas de estudo como o NotebookLM.

## Conteúdo

| Arquivo | Assunto |
|---|---|
| [docs/01-compra-e-checklist.md](docs/01-compra-e-checklist.md) | O que comprar, em que ordem, e riscos de compra |
| [docs/02-hardware.md](docs/02-hardware.md) | Especificações, diferenças entre Max, Plus e Pro |
| [docs/03-alimentacao-refrigeracao-armazenamento.md](docs/03-alimentacao-refrigeracao-armazenamento.md) | Fonte 5 V / 5 A, cooler, microSD, eMMC, NVMe |
| [docs/04-software-e-boot.md](docs/04-software-e-boot.md) | Sistemas, gravação de imagem, primeiro boot |
| [docs/05-links-e-comunidade.md](docs/05-links-e-comunidade.md) | Site oficial, comunidade, reviews, fabricantes |
| [docs/06-ssd-nvme.md](docs/06-ssd-nvme.md) | SSD NVMe: velocidade real do slot, como escolher, boot pelo NVMe, preços |
| [docs/07-gpio-e-pinout.md](docs/07-gpio-e-pinout.md) | Tabela dos 40 pinos, wiringOP, overlays, serial, ventoinha, RTC |
| [docs/08-npu-e-ia-local.md](docs/08-npu-e-ia-local.md) | NPU, RKNN, RKLLM, driver rocket, Ollama e llama.cpp |
| [docs/09-servidor-e-casos-de-uso.md](docs/09-servidor-e-casos-de-uso.md) | Servidor 24/7: Docker, NAS, Jellyfin, Home Assistant, Frigate, k3s |
| [docs/10-kernel-e-distribuicoes.md](docs/10-kernel-e-distribuicoes.md) | Kernel BSP e mainline, Armbian, Ubuntu Rockchip, DietPi, imagens oficiais |
| [docs/11-desempenho-temperatura-consumo.md](docs/11-desempenho-temperatura-consumo.md) | Números medidos: CPU, GPU, rede, armazenamento, temperatura, consumo |
| [docs/12-recuperacao-e-problemas.md](docs/12-recuperacao-e-problemas.md) | Diagnóstico e recuperação: MaskROM, SPI, serial, Wi-Fi, NVMe |
| [docs/13-manual-oficial-e-arquivos.md](docs/13-manual-oficial-e-arquivos.md) | Links oficiais, resumo do manual e respostas com base no manual e no esquemático |
| [docs/14-testes-na-placa-real.md](docs/14-testes-na-placa-real.md) | Comandos para confirmar na placa o que nenhuma fonte pública respondeu |

## Aviso

Preços e disponibilidade mudam. As informações foram levantadas entre 31/10/2026 e 03/11/2026 e não substituem a ficha técnica oficial, o manual e o esquemático da placa, cujos links estão em `docs/13-manual-oficial-e-arquivos.md`. Quando uma informação é relato de usuário, vem de outra placa ou não foi confirmada, o texto diz isso.

## Licença

Textos escritos para este repositório. Os links apontam para as fontes originais, e o conteúdo delas pertence aos respectivos donos.
