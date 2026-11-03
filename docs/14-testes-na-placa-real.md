# Orange Pi 5 Max: testes para fazer com a placa em mãos

Data: 03/11/2026. Estes testes fecham o que nenhuma fonte pública confirmou até essa data. Nenhum deles apaga dados. Rode em imagem Linux recente e anote também a saída de `uname -r` e o nome da imagem, porque o resultado depende do kernel. Os nomes de interface e de dispositivo podem mudar na sua imagem.

## 1. Caixa e conectores

- Veja se veio o parafuso do M.2. Se não veio, use um kit de parafusos M2. Anote a medida que servir.
- Confira o "+" da serigrafia da ventoinha e do RTC antes de ligar. O esquemático diz que o pino 1 é o positivo nos dois.

## 2. SSD NVMe

Velocidade e largura do link, que devem mostrar 8 GT/s e x4:

```
sudo lspci -vv | grep -E "LnkCap:|LnkSta:"
```

Leitura real contra leitura em cache. O `-T` mede o cache, e o `-t` mede o disco. É a diferença que explica números acima de 3,9 GB/s:

```
sudo hdparm -Tt /dev/nvme0n1
```

Leitura sequencial sem cache, sem gravar nada no disco:

```
sudo fio --name=leitura --filename=/dev/nvme0n1 --rw=read --bs=1M --iodepth=32 --direct=1 --runtime=30 --time_based --readonly
```

## 3. Kernel mainline

Com uma imagem de kernel mainline, como a edge do Armbian:

| Recurso | Comando | O que indica que funciona |
|---|---|---|
| HDMI | `cat /sys/class/drm/card*-HDMI*/status` | `connected` e imagem na tela |
| GPU | `dmesg \| grep -i panthor` e `vulkaninfo --summary` | driver carregado e Mali listado |
| NPU | `dmesg \| grep -i rocket` e `ls /dev/accel` | driver carregado e dispositivo presente |
| Rede 2,5 GbE | `ip -br link`, depois `sudo ethtool NOME \| grep Speed` | `2500Mb/s`, se o outro lado for de 2,5 Gb |
| Velocidade da rede | `iperf3 -c SERVIDOR` e `iperf3 -c SERVIDOR --reverse` | perto de 2,3 Gb/s nos dois sentidos |
| Wi-Fi | `dmesg \| grep -iE "bcmdhd\|brcm\|wifi"` e `iw dev` | interface sem fio listada. Em 2026, o mainline ainda depende de overlay e de driver externo; veja `10-kernel-e-distribuicoes.md` |

O sentido `--reverse` importa: uma medição publicada na Max deu 2,35 Gb/s de envio, mas só 773 Mb/s de recebimento.

## 4. Ollama e llama.cpp

Os núcleos rápidos do RK3588 costumam ser os `cpu4` a `cpu7`. Confirme com `lscpu -e`.

```
ollama run llama3.2:1b --verbose "Explique o que é um kernel em duas frases."
```

Anote o `eval rate`, em tokens por segundo.

```
taskset -c 4-7 ./build/bin/llama-bench -m modelo.gguf -t 4
```

Anote modelo, quantização e número de threads.

## 5. YOLO na NPU

Siga o exemplo de RKNN do manual v1.3 (cap. 3.37, a partir da p. 360) ou o de [rknn_model_zoo](https://github.com/airockchip/rknn_model_zoo). Anote o modelo, o tamanho da entrada e o tempo de inferência em milissegundos. Confira a versão do driver antes:

```
sudo cat /sys/kernel/debug/rknpu/version
```

## 6. Jellyfin com H.264 e HEVC

A [documentação do Jellyfin](https://jellyfin.org/docs/general/post-install/transcoding/hardware-acceleration/rockchip/) diz que o RKMPP depende do kernel BSP da Rockchip. Use uma imagem com ele e confira os dispositivos e os drivers:

```
ls -l /dev | grep -E "mpp|rga|dri|dma_heap"
sudo /usr/lib/jellyfin-ffmpeg/ffmpeg -v debug -init_hw_device rkmpp=rk -init_hw_device opencl=ocl@rk
```

Depois, reproduza um vídeo H.264 e um HEVC com bitrate reduzido, para forçar a transcodificação, e veja no log dela se aparece `rkmpp`.

## 7. Temperatura e consumo

**Pare o teste se passar de 85 °C**, que é o primeiro limite térmico do RK3588 no kernel mainline.

```
sudo apt install stress-ng
stress-ng --cpu 8 --timeout 600 &
watch -n 5 "cat /sys/class/thermal/thermal_zone*/temp"
```

Os valores saem em milésimos de grau. Compare com e sem ventoinha. Para medir o consumo, use um medidor de tomada na fonte, com e sem SSD.

## 8. Relógio (RTC)

Com a pilha instalada, confira se o relógio é detectado e mantém a hora depois de desligar a placa da tomada:

```
dmesg | grep -i rtc
sudo hwclock -r
```

## Fontes

- Medição de referência (sbc-reviews, issue 49): https://github.com/geerlingguy/sbc-reviews/issues/49
- Jellyfin, aceleração em Rockchip: https://jellyfin.org/docs/general/post-install/transcoding/hardware-acceleration/rockchip/
- Modelos RKNN: https://github.com/airockchip/rknn_model_zoo
