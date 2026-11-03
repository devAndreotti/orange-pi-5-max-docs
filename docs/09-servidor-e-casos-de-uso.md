# Orange Pi 5 Max como servidor 24/7: casos de uso, limites e comparação com mini PC x86

Data da coleta: 03/11/2026 (rodada de revisão, mesma data).

Este documento responde: dá para usar a Orange Pi 5 Max como servidor ligado o tempo todo (Docker, NAS, mídia, automação residencial, Kubernetes leve, acesso remoto)? Onde ela funciona bem, onde não funciona, e como ela se compara com um mini PC x86 usado, como o N100.

## Como ler as marcas

Cada afirmação relevante tem uma marca:

- **[oficial]**: veio de documentação de projeto ou fabricante que eu li de fato. Inclui o manual v1.3 da Orange Pi 5 Max (citado como "manual v1.3, p. X"), Frigate, Jellyfin, Home Assistant, k3s, Tailscale, OpenMediaVault e Docker.
- **[comunidade/Max]**: relato de usuário ou review que fala da Orange Pi 5 Max especificamente.
- **[placa irmã: nome]**: relato sobre outra placa RK3588 (Orange Pi 5, 5 Plus, Rock 5B etc.). Vale como indicação, não como fato para a Max.
- **[não confirmado]**: não achei fonte que sustente, ou a fonte achada é fraca ou contraditória. Onde procurei está dito no texto.

Sobre a fonte de hardware: o manual v1.3 do fabricante substituiu a ficha técnica que faltava na primeira rodada. Ele confirma CPU, memória, slots, rede, Wi-Fi e alimentação. Números de desempenho e de consumo vêm de reviews, que são fontes mais fracas.

## Resumo

- A Max serve bem para: NAS pequeno com um NVMe, Jellyfin com transcodificação por hardware (com os cuidados da seção 3), Home Assistant em contêiner Linux (se a imagem tiver arm64 para a placa), Frigate com NPU (a documentação lista o RK3588 e descreve os requisitos de host), e serviços Docker com imagem arm64.
- Ela exige mais cuidado que um mini PC x86 em: compatibilidade de imagens Docker, kernel e drivers do fornecedor (BSP em vez de kernel padrão), e escolha de armazenamento (microSD desgasta; NVMe é a opção indicada para uso contínuo).
- Não é adequada para: software que só existe para x86, virtualização pesada, qualquer coisa que exija ECC ou redundância de hardware (o manual não cita ECC; ver seção 10), e cargas que precisem de muitas vias PCIe.
- Frente a um N100 usado, a vantagem da Max é a NPU (6 TOPS, manual v1.3, p. 11), a porta 2,5 GbE e o NVMe no mesmo conjunto. A vantagem do N100 é a compatibilidade de software. No consumo ocioso, a Max ficou em 2,4 W e o N100 em 6,5 W no mesmo teste (seção 11). Preço do N100 em reais: não achei cotação atual com data (seção 11).

## 1. Docker em ARM64

- A imagem Linux da Orange Pi traz o Docker instalado, mas o serviço não vem ativado. O manual manda rodar o script `enable_docker.sh` e testar com `docker run hello-world` [oficial, manual v1.3, p. 273].
- O erro mais comum é a imagem sem variante arm64. Um relato de Orange Pi 5 (não Max) com "no matching manifest for linux/arm64/v8" mostrou que a imagem do banco não tinha arm64, enquanto a da aplicação tinha. Trocar por uma imagem com arm64 resolveu [placa irmã: Orange Pi 5, trecho de busca; página não aberta].
- Como verificar antes de instalar: `docker buildx imagetools inspect <imagem:tag>` lista as plataformas da tag. A documentação do Docker dá esse comando e mostra entradas como `linux/arm64/v8` [oficial, documentação do Docker, buildx imagetools inspect].
- O Frigate documenta uma tag `stable-rk` para Rockchip, descrita como build mantida pela comunidade [oficial, Frigate]. Prova de que a tag existe, não de que funciona na Max.
- Recomendação, sem fonte lida: verificar arm64 em cada imagem antes de planejar, e manter o `docker-compose.yml` com a imagem verificada e a versão que funcionou.

## 2. NAS: porta 2,5 GbE, NVMe, Samba e OpenMediaVault

**Rede.** A Max tem uma porta 2.5G Ethernet com controlador Realtek RTL8125BG [oficial, manual v1.3, p. 12]. Também tem Wi-Fi 6E e Bluetooth 5.3 no módulo AP6611 [oficial, manual v1.3, p. 12]. O manual pede o `nmcli` para conectar ao Wi-Fi e avisa para não configurar a rede editando `/etc/network/interfaces` [oficial, manual v1.3, p. 158]. Para servidor 24/7, use cabo (recomendação).

A velocidade teórica de 2,5 Gb/s dá cerca de 280 MB/s úteis (estimativa minha, sem medição). Não achei medição de iperf3 ou de transferência na Max: a review aberta não traz números de rede [comunidade/Max, review]. Um teste de gigabit numa Orange Pi 5 mostrou 940 Mb/s [placa irmã: Orange Pi 5, trecho de busca].

**NVMe.** O manual lista um slot M.2 M-Key PCIe 3.0 x4 para SSD [oficial, manual v1.3, p. 12] e mostra a gravação de imagem direto no NVMe [oficial, manual v1.3, p. 65-67]. Não há segundo slot listado. Uma review reporta leitura sequencial de até 5,7 GB/s [comunidade/Max, review], sem dizer a ferramenta, o bloco ou a fila. Esse número passa do teto teórico do PCIe 3.0 x4 (cerca de 3,9 GB/s, cálculo meu), e a mesma review diz LPDDR4X, o que o manual contradiz (seção 12). Não use o número como referência. Na prática, em rede o gargalo é o 2,5 GbE, e qualquer NVMe Gen3 já o satura. Um adaptador NVMe-SATA com chip ASM1166 numa Orange Pi 5 Plus teve LEDs que apagavam após o boot [placa irmã: Orange Pi 5 Plus, trecho de busca].

**Sistema operacional do NAS.** A documentação do OpenMediaVault lista cinco formas de instalação: ISO em disco dedicado (recomendada, para x86-64), pendrive USB, Debian sobre um sistema existente, debootstrap e imagem para cartão SD. Para placas ARM, a página indica a variante "Debian Operating System" [oficial, OpenMediaVault, instalação]. O OMV não é suportado em LXC nem em outros contêineres [oficial, OpenMediaVault]. A página não trata de separar o disco do sistema dos discos de dados, e não traz instrução específica de instalação em ARM64 [oficial, OpenMediaVault]. O aviso de que o Armbian inclui o OMV no `armbian-config` é de trecho de busca e não está confirmado para a Max.

**Samba.** Não há fonte aberta sobre Samba na Max nem benchmark de compartilhamento. Samba é um pacote de Debian; o uso é esperado, mas não verificado [não confirmado].

**Por que o NAS sozinho não é o melhor uso.** A placa tem uma porta de 2,5 Gb/s e um único slot M.2. Para um NAS com vários discos, faltam portas SATA. Um adaptador USB para vários HDs passa pelo mesmo barramento USB, o que limita a soma das velocidades (regra geral de barramento, não verificada para a Max).

## 3. Jellyfin e transcodificação por hardware (RKMPP)

- O Jellyfin exige a versão 10.9 ou mais nova para RKMPP [oficial, Jellyfin].
- O RK3588/3588S é o SoC mais recomendado: decodifica H.264 10 bits e AV1 [oficial, Jellyfin]. O RK3576 tem cerca da metade da capacidade de transcodificação do RK3588 e não tem tone mapping; o RK356x tem limite de codificação menor [oficial, Jellyfin].
- Exige kernel BSP da Rockchip, 5.10 LTS ou 6.1 LTS [oficial, Jellyfin]. Uma review da Max mostra kernel 6.1.115-vendor-rk35xx [comunidade/Max, review]. Isso é compatível com o requisito, mas não prova que o transcode funciona.
- Dispositivos a passar para o contêiner: a página usa um laço com `dri`, `dma_heap`, `mali0`, `rga`, `mpp_service` e outros nós, como `iep`, `vpu_service`, `rkvdec` e `rkvenc`, desde que existam no host [oficial, Jellyfin]. No Docker, a página indica modo privilegiado e as opções `systempaths=unconfined` e `apparmor=unconfined` [oficial, Jellyfin]. Isso reduz o isolamento do contêiner; é um custo a aceitar conscientemente.
- Verificação, no host: `sudo /usr/lib/jellyfin-ffmpeg/ffmpeg -v debug -init_hw_device rkmpp=rk -init_hw_device opencl=ocl@rk`. Para ver a carga do VPU durante um transcode, a página indica ler `/proc/mpp_service/load` [oficial, Jellyfin].
- A página oficial não fala em fallback silencioso para software. Para confirmar que o hardware está em uso, use o comando de verificação e a leitura da carga do VPU durante a reprodução.
- Tone mapping de HDR para SDR (OpenCL) só é suportado no RK3588 [oficial, Jellyfin].
- Relato de Max, 2024: um usuário de Orange Pi 5 Max, com Jellyfin em Docker (imagem `jellyfin/jellyfin`) e kernel 6.1.43-rockchip-rk3588, viu a transcodificação falhar em clientes Android e PC. A reprodução direta funcionava. No log, o erro era `av1_rkmpp ... Failed to init MPP context` ao decodificar AV1. No fórum, um membro da equipe apontou que o nó `av1d` vem com `status = "disabled"` na árvore de dispositivos da imagem da Orange Pi. O usuário afirmou ter ativado o nó e gerado uma imagem própria, em 04/09/2024, sem confirmação de terceiros [comunidade/Max, thread do fórum Jellyfin, posts 5, 19, 26 e 27]. Não há relato desse caso para H.264 ou HEVC. A documentação oficial não trata desse caso. Procurei no fórum da Armbian por busca web e não achei thread sobre isso.
- Outros relatos do jellyfin-ffmpeg: a issue #642 (RK3588, Jellyfin 10.11.3, falha em alguns arquivos) foi fechada em 01/12/2025 e confirmada como resolvida por quem reportou; ela não cita a Max [comunidade/RK3588, https://github.com/jellyfin/jellyfin-ffmpeg/issues/642]. A issue #765 (VP9 em RK3588) foi fechada em 11/09/2026, mas o autor corrigiu dados da própria tabela e a conclusão não está confirmada [não confirmado]. A busca nas issues do nyanmisaka/ffmpeg-rockchip não achou relato de Max nem de RK3588 com Jellyfin.
- Recomendação: antes de depender do transcode, rode o comando de verificação e teste um arquivo em H.264 e um em HEVC com o cliente que você mais usa. Se o AV1 falhar com `Failed to init MPP context`, o relato único indica problema da árvore de dispositivos da imagem, não do Docker.

## 4. Frigate e NPU

- A documentação de detectores do Frigate lista RK3562, RK3566, RK3568, RK3576 e RK3588 como SoCs Rockchip com detector RKNN, usando o RKNN-Toolkit2 v2.3.2 [oficial, Frigate, detectores de objeto].
- Requisitos de host, na seção Rockchip da instalação [oficial, Frigate, instalação]:
  - kernel BSP da Rockchip, 5.10 ou 6.1, com o sufixo `-rockchip` na versão (conferir com `uname -r`); a página recomenda Armbian quando a placa for suportada;
  - drivers `rkvdec2` e `rknpu`; a versão do driver NPU aparece em `/sys/kernel/debug/rknpu/version`, que deve ser v0.9.2 ou mais nova;
  - nós `/dev/dri` com `card0`, `card1`, `renderD128` (VPU) e `renderD129` (NPU).
- A página manda mapear para o contêiner `/dev/dri`, `/dev/dma_heap`, `/dev/rga`, `/dev/mpp_service` e `/sys` em somente leitura, com privilégio no primeiro teste e depois as opções de segurança indicadas [oficial, Frigate].
- A seção não cita a biblioteca `librknnrt` nem separa o RK3588 das outras placas Rockchip. Quanto à biblioteca, fica [não confirmado]; a seção vale para a família Rockchip.
- O modelo padrão é baixado do GitHub no primeiro início e fica em cache, depois funciona offline [oficial, Frigate]. O primeiro boot precisa de internet.
- Medição: uma review da Max roda o TinyLlama 1.1B quantizado a cerca de 20 tokens/s pela NPU (três execuções: 20,27, 20,04 e 20,40) [comunidade/Max, review]. Isso mostra que a NPU funciona no sistema testado, mas não é benchmark de visão computacional. Não achei medição de Frigate na Max.
- Recomendação: gravação contínua vai desgastar um microSD; use NVMe ou HD para gravação.

## 5. Home Assistant

- A página oficial de instalação lista: Home Assistant Green (HAOS de fábrica); HAOS em Raspberry Pi 4 ou 5 (mínimo 2 GB), Odroid e x86-64; Home Assistant Container em Raspberry Pi 3, 4 ou 5 e em Linux; e HAOS em máquina virtual em Linux, macOS e Windows [oficial, Home Assistant].
- A página não cita aarch64 nem RK3588, nem Orange Pi. Não há confirmação oficial de que HAOS rode na Max [não confirmado]. No repositório oficial da HAOS (lido em 03/11/2026), não há placa para RK3588 nem para Orange Pi. A imagem `generic-aarch64` usa `BOOTLOADER=grub`, ou seja, inicialização por UEFI, e kernel 6.18.60 [oficial: https://github.com/home-assistant/operating-system/blob/main/buildroot-external/configs/generic_aarch64_defconfig e https://github.com/home-assistant/operating-system/blob/main/buildroot-external/board/arm-uefi/generic-aarch64/meta]. A Max inicia por U-Boot gravado na SPI Flash [oficial, manual v1.3, p. 65] (seção 12). A única placa Rockchip na pasta de placas do repositório é a Home Assistant Green (`nabucasa/green`). Se a imagem inicia na Max, isso é um teste a fazer, não um dado.
- A opção que a página lista para Linux é o Home Assistant Container. A própria página diz que a instalação em contêiner não tem acesso a add-ons, que as atualizações são manuais, e que Thread e Z-Wave não têm suporte pronto [oficial, Home Assistant]. Para uso 24/7, isso significa que você assume a manutenção.
- Se for usar Zigbee ou Z-Wave, é preciso um coordenador USB. A página não diz isso com essas palavras; é consequência da falta de suporte pronto [não confirmado para a Max].

## 6. k3s e Kubernetes

- Requisitos oficiais do k3s: arm64 é suportado. Mínimos: servidor com 2 núcleos e 2 GB de RAM; agente com 1 núcleo e 512 MB [oficial, k3s]. Em Raspberry Pi, a página manda ativar cgroups de memória no cmdline; a página não cita Armbian nem RK3588, então esse ponto não se aplica à Max sem verificação [não confirmado].
- A página recomenda SSD externo ou NVMe em placas ARM, porque SD e eMMC não aguentam a escrita do etcd [oficial, k3s]. Mais um motivo para não colocar o sistema em microSD.
- Os relatos encontrados divergem. Um usuário de Orange Pi 5 Plus com Fedora CoreOS não conseguiu fazer o k3s funcionar e sugeriu imagens de fornecedor ou kernel mainline [placa irmã: Orange Pi 5 Plus, trecho de busca].
- Outro projeto em Orange Pi 5 escolheu kubeadm e rodou kernel do fornecedor para ter suporte ao hardware [placa irmã: Orange Pi 5, trecho de busca].
- Um relato sobre RK3588 (Turing RK1) diz que k3s funciona no Armbian com kernel BSP [placa irmã: Turing RK1, trecho de busca].
- Conclusão cautelosa: k3s é viável com sistema do fornecedor ou Armbian e kernel BSP. Não achei relato de k3s estável na Max em 24/7. Trate como experimento até testar.
- Recomendação: para uma máquina só, Docker Compose costuma ser mais simples que Kubernetes. Kubernetes faz sentido se você quiser aprender ou tiver mais de um nó.

## 7. Tailscale e acesso remoto

- SSH: o manual confirma o acesso por SSH com usuário `root` ou `orangepi`, mas as senhas padrão são públicas no manual. Troque-as antes de expor o servidor (recomendação) [oficial, manual v1.3, p. 182-183].
- A documentação oficial do Tailscale para Linux recomenda o script de instalação (`curl -fsSL https://tailscale.com/install.sh | sh`) em distribuições com gerenciador de pacotes, seguido de `sudo tailscale up` [oficial, Tailscale, instalação no Linux]. Para outros casos, a página indica binários estáticos por arquitetura.
- A página não cita ARM64 pelo nome, e não confirmei o suporte específico na Max [não confirmado]. Não achei relato de instalação do Tailscale na Max.
- Subnet router: a página manda ativar o encaminhamento de IP em `/etc/sysctl.d/99-tailscale.conf` (IPv4 e IPv6) e anunciar as sub-redes com `sudo tailscale set --advertise-routes=...`. O `tailscale up` tem a opção `--snat-subnet-routes=false` para desligar o NAT de origem. As rotas precisam ser aprovadas, a não ser que estejam em `autoApprovers` [oficial, Tailscale, subnets].
- Recomendação: rode o Tailscale no host, não dentro de contêiner, para ter acesso administrativo ao servidor mesmo se o Docker cair. A página não trata do caso de contêiner; a recomendação é minha.

## 8. Atualização sem perder o sistema

- Não achei fonte específica sobre atualização de sistema na Max sem perder dados. O risco principal, pelos relatos de boot, é atualizar o kernel ou o bootloader e perder o boot pelo NVMe [comunidade/Max e placa irmã, trecho de busca].
- Prática geral que eu recomendo, sem fonte lida: manter um microSD com a imagem anterior para recuperação, fazer backup da configuração e dos volumes do Docker antes de atualizar, e atualizar pacotes sem trocar de versão de kernel no mesmo passo [não confirmado; prática geral].

## 9. Cuidados de 24/7

**Alimentação.** O manual indica entrada Type-C de 5 V / 5 A [oficial, manual v1.3, p. 13]. Fontes de 2 A ou de celular podem reiniciar a placa em carga alta ([comunidade/Max e revendedor], não reabertos nesta rodada). Para 24/7, use fonte de 5 V / 5 A de marca conhecida (recomendação).

**Temperatura.** O comando `sensors` mostra as zonas térmicas do SoC, e o limite crítico do `soc_thermal` no exemplo do manual é +115 °C [oficial, manual v1.3, p. 221-222]. Para o NVMe, o manual usa `smartctl` com o campo `Temperature` [oficial, manual v1.3, p. 222]. Os valores do exemplo do manual são de uma captura, não de referência.

**Desgaste do microSD.** Fontes de comunidade dizem que cartões microSD têm desgaste por escrita e que a escrita de log é a maior parte disso [comunidade, discussões de Armbian e Raspberry Pi, trecho de busca]. Não achei medição de vida útil específica da Max. Recomendação: NVMe para o sistema e o microSD só para recuperação. O manual mostra que a Max inicia do NVMe com U-Boot na SPI Flash [oficial, manual v1.3, p. 65-67]. A placa também tem slot de eMMC, mas só com módulo externo [oficial, manual v1.3, p. 12].

**Logs em RAM e swap.** O Armbian usa log2ram por padrão, segundo um trecho de fórum; não abri a página e não confirmei o padrão [comunidade, trecho de busca]. Há discussão sobre se isso ajuda ou atrapalha. Um lado diz que ajuda a durabilidade; outro diz que pode dar problema com Kubernetes. O tamanho sugerido em um relato foi de 150 a 250 MB em máquinas com pelo menos 5 GB de RAM [comunidade, trecho de busca; relato de Pi-hole]. zram-config cobre swap, diretórios e logs para reduzir escrita, mas exige sincronizar antes de desligar [projeto zram-config, trecho de busca].

Recomendação: usar zram para swap, deixar log2ram ou log em RAM com tamanho adequado ao volume de log, e mandar a escrita pesada (banco, gravação de câmera, dados de Docker) para NVMe ou HD.

**Backup.** Backup de configuração e de volumes do Docker em outro disco ou máquina. Não achei fonte específica para a Max; é prática geral (recomendação).

**Queda de energia e no-break.** Não achei fonte sobre a Max com queda de energia ou no-break. Pela escrita de logs e pela corrupção de sistema de arquivos em queda abrupta, um no-break com desligamento automático pela rede (NUT ou equivalente) é recomendado (recomendação, sem fonte lida). Vale testar o desligamento limpo antes de confiar.

## 10. O que a Max NÃO faz bem

- **Software só x86.** Imagens e instaladores que só existem para x86 não rodam, ou rodam via emulação lenta. Antes de adotar um serviço, cheque se ele tem arm64 (seção 1).
- **Poucas vias PCIe.** O manual lista um único slot M.2 PCIe 3.0 x4 [oficial, manual v1.3, p. 12]. Para mais NVMe, adaptadores ou placa de expansão, a placa não tem vias sobrando (recomendação; não achei a lista completa de lanes).
- **Sem ECC.** O manual v1.3 descreve a memória como LPDDR5 de 4, 8 ou 16 GB [oficial, manual v1.3, p. 11-12] e não menciona ECC. Procurei ECC no texto do manual, sem resultado. Não abri outra fonte sobre o tema. Como a ausência no manual não prova a inexistência, o item segue [não confirmado]. Para dados que não podem corromper, trate como risco e use backup.
- **Kernel e drivers.** Transcodificação e NPU dependem de kernel BSP e de drivers do fornecedor (rknpu, rkvdec2), o que atrasa atualizações e complica o suporte [oficial, Jellyfin e Frigate].
- **Consumo.** A review da Max reporta 15 a 25 W em carga, sem condições de teste [comunidade/Max, review]. Em ocioso, o LinuxLinks mediu **2,4 W** com a tela desligada (sem o monitor), com Wi-Fi e Bluetooth ligados, Ubuntu 22.04, sem underclock e clock ocioso de 408 MHz. O valor foi lido do gráfico da página, em 03/11/2026. O texto diz que a Max fica abaixo da RV2 [comunidade/Max, LinuxLinks]. É uma medição, em uma configuração.

## 11. Comparação com mini PC x86 usado (N100)

**Consumo.** No mesmo teste de ocioso do LinuxLinks, a Max ficou em 2,4 W, a RV2 em 2,5 W, a Raspberry Pi 5 em 3,5 W, o N100 em 6,5 W, o N95 em 7,8 W e o i7-1360P em 12,7 W [comunidade/Max, LinuxLinks; gráfico lido em 03/11/2026]. O texto cita "N95 e N100" sem modelo exato, então use o número do N100 com essa ressalva. Sites de comparação reportam N100 com 5 a 12 W em ociosidade e cerca de 21 a 27 W em carga [sites de comparação, fontes fracas; não reabertos nesta rodada]. A diferença de consumo ocioso favorece a Max, em uma medição só.

**Preço.** Preço da Max: R$ 2.771,97 para a versão de 16 GB no Mercado Livre, em 31/10/2026, informado pelo usuário (não verificado por fonte aberta). Para o N100 em reais, busquei novo e usado, e não achei anúncio brasileiro com data. A busca em português trouxe lojas de Portugal, Austrália, Alemanha e revendedores internacionais, em outras moedas. O único preço brasileiro que li é do Canaltech, de 23/05/2023: o LarkBox X 2023 com N100, 12 GB e SSD de 512 GB saía por US$ 199 (cerca de R$ 991 na conversão direta). A própria matéria diz que o produto ainda não estava à venda, então é preço de lançamento, não de loja, e a configuração não é a de 16 GB [Canaltech, 23/05/2023]. Preço atual de N100 no Brasil continua [não confirmado]; é preciso cotar em loja.

**Compatibilidade.** Um N100 roda Windows, Linux x86 e a maior parte das imagens Docker sem ajuste. A Max exige atenção a imagens arm64, kernel e drivers do fornecedor. Esta é a maior diferença prática [oficial e comunidade, ver seções 1 a 4].

**Recursos.** A Max tem NPU de até 6 TOPS [oficial, manual v1.3, p. 11] e porta 2.5G Ethernet [oficial, manual v1.3, p. 12]. Sobre o N100, a iGPU Intel com QuickSync e o suporte no Jellyfin e no Frigate são conhecimento geral, sem fonte aberta nesta rodada [não confirmado; verifique antes de decidir].

**Conclusão honesta.** Para um servidor com uso previsível, serviço x86 e pouca manutenção, um mini PC N100 usado tende a ser a escolha mais simples (recomendação). A Max faz mais sentido se o objetivo é aprender ARM, usar a NPU ou montar algo compacto, aceitando mais configuração.

## 12. Divergências entre fontes

- **Memória da Max.** O manual v1.3 diz LPDDR5 em versões de 4, 8 e 16 GB [oficial, manual v1.3, p. 11-12]. A review diz LPDDR4X-2133 [comunidade/Max, review]. Vale o manual: LPDDR5. Sobre ECC, ver seção 10.
- **Velocidade do NVMe.** A review reporta 5,7 GB/s sem método; o limite teórico do PCIe 3.0 x4 é cerca de 3,9 GB/s. Não confirmada.
- **Boot.** O manual mostra U-Boot gravado na SPI Flash com `rkspi_loader.img` [oficial, manual v1.3, p. 65] e uma imagem de exemplo com Linux 5.10.160 [oficial, manual v1.3, p. 66]. Isso indica bootloader e kernel de fornecedor. Não confirmei se a placa é "mainline" em algum sentido, porque o manual não usa esse termo.
- **Jellyfin.** A documentação oficial recomenda RK3588. O relato de 2024 da Max falhou no AV1, por `av1d` desativado na DTS da imagem, e não cobre H.264 ou HEVC [oficial x comunidade/Max]. A causa de AV1 foi apontada no fórum; o ajuste do usuário não foi confirmado por terceiros.
- **k3s.** Relatos divergem entre tentativa com Fedora CoreOS (falha) e uso com kernel BSP no Armbian (funciona) [placas irmãs]. A documentação oficial do k3s não resolve isso.

## 13. O que ainda não consegui confirmar

- Preço do N100 em reais, novo ou usado: não achei anúncio brasileiro com data; o único preço brasileiro é de lançamento (2023). O preço da Max é dado do usuário.
- Velocidade real do 2,5 GbE na Max: sem iperf3 ou medição aberta.
- Samba e OMV na Max, com desempenho; e a afirmação do Armbian sobre o `armbian-config`.
- Home Assistant generic-aarch64 na Max: o repositório não tem placa RK3588, e não li relato de inicialização na Max.
- Tailscale na Max: a página oficial não cita ARM64 pelo nome.
- Frigate: a seção Rockchip não cita a biblioteca `librknnrt`.
- k3s estável na Max em 24/7.
- Procedimento de atualização sem perda de sistema na Max.
- Queda de energia e no-break com a Max.
- ECC e vias PCIe completas da Max.
- Transcodificação do Jellyfin na Max: o relato de 2024 aponta AV1 com `av1d` desativado; não achei confirmação de que o ajuste funcionou, nem de H.264 e HEVC na Max.

## Fontes

Páginas abertas nesta revisão:

- Frigate, detectores de objeto (RKNN): https://docs.frigate.video/configuration/object_detectors/
- Frigate, instalação (seção Rockchip): https://docs.frigate.video/frigate/installation/
- Home Assistant, instalação: https://www.home-assistant.io/installation/
- Jellyfin, aceleração por hardware Rockchip (RKMPP): https://jellyfin.org/docs/general/post-install/transcoding/hardware-acceleration/rockchip/
- k3s, requisitos: https://docs.k3s.io/installation/requirements
- Tailscale, instalação no Linux: https://tailscale.com/kb/1031/install-linux
- Tailscale, subnet routers: https://tailscale.com/kb/1019/subnets
- OpenMediaVault, documentação: https://docs.openmediavault.org/
- OpenMediaVault, instalação: https://docs.openmediavault.org/en/stable/installation/index.html
- Docker, `docker buildx imagetools inspect`: https://docs.docker.com/reference/cli/docker/buildx/imagetools/inspect/
- LinuxLinks, consumo da Orange Pi 5 Max: https://www.linuxlinks.com/orangepi5-max-single-board-computer-power-consumption/
- LinuxLinks, gráfico de consumo ocioso (lido como imagem): https://www.linuxlinks.com/wp-content/uploads/2025/07/OrangePi5-Max-Power-Consumption-Idle-Chart.png
- tinycomputers.io, review da Orange Pi 5 Max: https://tinycomputers.io/posts/rk3588-orange-pi-5-max-review.md
- Fórum do Jellyfin, thread com relato da Max (páginas 1 a 3): https://forum.jellyfin.org/t-solved-rk3588-hardware-acceleration
- jellyfin-ffmpeg, issue #642 (RK3588, resolvida): https://github.com/jellyfin/jellyfin-ffmpeg/issues/642
- jellyfin-ffmpeg, issue #765 (VP9 em RK3588, contestada): https://github.com/jellyfin/jellyfin-ffmpeg/issues/765
- HAOS, configuração generic-aarch64: https://github.com/home-assistant/operating-system/blob/main/buildroot-external/configs/generic_aarch64_defconfig
- HAOS, placa arm-uefi/generic-aarch64: https://github.com/home-assistant/operating-system/blob/main/buildroot-external/board/arm-uefi/generic-aarch64/meta
- Canaltech, LarkBox X 2023 com N100 (lançamento, 23/05/2023): https://canaltech.com.br/produtos/mini-pc-chines-de-us-200-tem-intel-n100-12-gb-de-ram-e-ssd-de-512-gb-250524/

Também usado, sem URL própria: Manual oficial da Orange Pi 5 Max, v1.3 (citado como "manual v1.3, p. X").

Os demais dados deste documento que vieram só de trechos de busca estão marcados no texto como trecho de busca e não aparecem aqui, porque as páginas não foram abertas.
