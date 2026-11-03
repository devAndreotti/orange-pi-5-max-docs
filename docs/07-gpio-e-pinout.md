# Orange Pi 5 Max: GPIO, pinout de 40 pinos e interfaces

Data da coleta: 03/11/2026. Revisado com o manual v1.3 da Orange Pi 5 Max (manual do usuário, Shenzhen Xunlong Software Co., Ltd).

Este documento reúne o que está confirmado sobre o conector de 40 pinos (GPIO) da Orange Pi 5 Max, placa baseada no SoC Rockchip RK3588. Ele foi escrito para ser lido isoladamente. A referência de página é a numeração impressa no rodapé do manual.

## Como ler as marcações

- **[oficial: manual v1.3, p. X]**: afirmação lida no manual do fabricante, com a página. Quando o dado vem de imagem, a página da imagem é citada.
- **[comunidade/Max]**: vem de fonte de terceiros que trata da Max (páginas de produto, wiki de usuário, imprensa). Não substitui o manual.
- **[não confirmado]**: não há resposta no manual nem em fonte confiável. A nota diz onde foi procurado.

## Resumo

- O conector é de 40 pinos, em duas fileiras, passo de 2,54 mm. [oficial: manual v1.3, p. 223]
- Há 28 pinos de GPIO. Todos operam em **3,3 V**. [oficial: manual v1.3, p. 224]
- Os pinos 1 e 17 são 3,3 V. Os pinos 2 e 4 são 5 V. Os pinos 6, 9, 14, 20, 25, 30, 34 e 39 são GND. [oficial: manual v1.3, p. 223 e p. 225]
- A tabela pino a pino está na seção 1 deste documento, transcrita da imagem do manual.
- O console serial de depuração fica nos pinos 6 (GND), 8 (TX da placa) e 10 (RX da placa), a 1500000 baud. [oficial: manual v1.3, p. 136 e p. 225]
- A wiringOP tem uma branch específica para a Max (`-b next`) e já vem instalada na imagem Linux. [oficial: manual v1.3, p. 224 e p. 225]
- Overlays (device tree) habilitam SPI, I2C, UART, PWM e CAN. Os nomes estão na seção 5. [oficial: manual v1.3, p. 230 a 245 e seções 4.x]

## 1. Pinout de 40 pinos

Tabela transcrita da imagem oficial da seção 3.16 (manual v1.3, p. 223). Os valores de wPi e o nome principal de cada pino foram conferidos com a saída de `gpio readall` do manual (manual v1.3, p. 225).

Colunas: **GPIO (SoC)** é o nome do pino no RK3588; **GPIO Linux** é o número global usado pelo kernel; **wPi** é o número usado pela wiringOP; **Função principal** é o nome que `gpio readall` mostra; **Funções multiplexadas** são as funções alternativas listadas na imagem da p. 223. Na imagem, as funções aparecem em três colunas por lado. Aqui elas foram listadas na ordem das colunas.

| Pino | GPIO (SoC) | GPIO Linux | wPi | Função principal (readall) | Funções multiplexadas (p. 223) |
|---:|---|---:|---:|---|---|
| 1 | 3,3 V | – | – | 3,3 V | – |
| 2 | 5 V | – | – | 5 V | – |
| 3 | GPIO0_C0 | 16 | 0 | SDA.2 | PWM1_M0 (fd8b0010); CAN0_RX_M0; I2C2_SDA_M0 |
| 4 | 5 V | – | – | 5 V | – |
| 5 | GPIO0_B7 | 15 | 1 | SCL.2 | PWM0_M0 (fd8b0000); CAN0_TX_M0; I2C2_SCL_M0 |
| 6 | GND | – | – | GND | – |
| 7 | GPIO1_A7 | 39 | 2 | PWM3 | PWM3_IR_M3 (fd8b0030) |
| 8 | GPIO0_B5 | 13 | 3 | TXD.2 | UART2_TX_M0; I2C1_SCL_M0 |
| 9 | GND | – | – | GND | – |
| 10 | GPIO0_B6 | 14 | 4 | RXD.2 | UART2_RX_M0; I2C1_SDA_M0 |
| 11 | GPIO1_A0 | 32 | 5 | RXD.6 | SPI4_MISO_M2; I2C2_SDA_M4; UART6_RX_M1 |
| 12 | GPIO4_A6 | 134 | 6 | GPIO4_A6 | I2C5_SCL_M2 |
| 13 | GPIO1_A1 | 33 | 7 | TXD.6 | SPI4_MOSI_M2; I2C2_SCL_M4; UART6_TX_M1 |
| 14 | GND | – | – | GND | – |
| 15 | GPIO1_A2 | 34 | 8 | GPIO1_A2 | SPI4_CLK_M2; PWM0_M2 (fd8b0000); I2C4_SDA_M3 |
| 16 | GPIO1_A3 | 35 | 9 | GPIO1_A3 | I2C4_SCL_M3; PWM1_M2 (fd8b0010); SPI4_CS0_M2 |
| 17 | 3,3 V | – | – | 3,3 V | – |
| 18 | GPIO1_A4 | 36 | 10 | GPIO1_A4 | – |
| 19 | GPIO1_B2 | 42 | 11 | SPI0_TXD | UART4_RX_M2; SPI0_MOSI_M2 |
| 20 | GND | – | – | GND | – |
| 21 | GPIO1_B1 | 41 | 12 | SPI0_RXD | SPI0_MISO_M2 |
| 22 | GPIO1_B0 | 40 | 13 | GPIO1_B0 | – |
| 23 | GPIO1_B3 | 43 | 14 | SPI0_CLK | UART4_TX_M2; SPI0_CLK_M2 |
| 24 | GPIO1_B4 | 44 | 15 | SPI0_CS0 | SPI0_CS0_M2 |
| 25 | GND | – | – | GND | – |
| 26 | GPIO1_B5 | 45 | 16 | SPI0_CS1 | SPI0_CS1_M2 |
| 27 | GPIO1_B7 | 47 | 17 | RXD.1 | PWM13_M2 (febf0010); I2C5_SDA_M3; UART1_RX_M1 |
| 28 | GPIO1_B6 | 46 | 18 | TXD.1 | UART1_TX_M1; I2C5_SCL_M3 |
| 29 | GPIO3_C1 | 113 | 19 | GPIO3_C1 | SPI1_CLK_M1 |
| 30 | GND | – | – | GND | – |
| 31 | GPIO3_B5 | 109 | 20 | CAN1_RX | PWM12_M0 (febf0000); UART3_TX_M1; CAN1_RX_M0 |
| 32 | GPIO1_D6 | 62 | 21 | PWM14 | PWM14_M2 (febf0020); I2C8_SCL_M2 |
| 33 | GPIO3_B6 | 110 | 22 | CAN1_TX | PWM13_M0 (febf0010); UART3_RX_M1; CAN1_TX_M0 |
| 34 | GND | – | – | GND | – |
| 35 | GPIO3_C2 | 114 | 23 | GPIO3_C2 | PWM14_M0 (febf0020); SPI1_CS0_M1 |
| 36 | GPIO1_D7 | 63 | 24 | GPIO3_D7 (rótulo do readall, ver nota) | I2C8_SDA_M2 |
| 37 | GPIO4_A7 | 135 | 25 | GPIO4_A7 | I2C5_SDA_M2 |
| 38 | GPIO3_C0 | 112 | 26 | GPIO3_C0 | SPI1_MISO_M1 |
| 39 | GND | – | – | GND | – |
| 40 | GPIO3_B7 | 111 | 27 | GPIO3_B7 | SPI1_MOSI_M1 |

Fonte: manual v1.3, p. 223 (tabela de funções, imagem) e p. 225 (`gpio readall`, imagem).

### Notas sobre a tabela

- **Pinos com pull-up externo de 3,3 V:** 29, 31, 33, 35, 38 e 40. Nesses pinos, o modo pull-down não funciona. [oficial: manual v1.3, p. 228, imagem do `gpio readall`]
- **Rótulo do pino 36:** a imagem da p. 223 diz GPIO1_D7 e o `gpio readall` da p. 228 mostra "GPIO3_D7". O número GPIO Linux (63) corresponde ao banco GPIO1 (32 a 63), então GPIO1_D7 é o nome correto. Essa leitura é inferência a partir da numeração. [oficial: manual v1.3, p. 223 e p. 228]
- **CAN0:** pino 3 é CAN0_RX_M0 e pino 5 é CAN0_TX_M0, como na imagem da p. 223. O esquemático confirma: GPIO0_C0 (pino 3) tem CAN0_RX_M0, e GPIO0_B7 (pino 5) tem CAN0_TX_M0. [oficial: esquemático V1.1, p. 2] O texto das p. 245 e da seção 4.6.7 do manual diz o contrário e está trocado.
- **CAN1:** pino 31 é RX e pino 33 é TX. Imagem, texto (p. 245) e readall (p. 225) concordam. [oficial: manual v1.3, p. 223, p. 225 e p. 245]
- **Pinos 3 e 5:** aparecem como I2C2_M0 e também como CAN0_M0 e PWM. Como as funções são alternativas, use uma por vez. Essa leitura é inferência a partir da tabela multiplexada.
- **Pinos 8 e 10:** são UART2 (TXD.2 e RXD.2). A UART2 não aparece na lista de UARTs que o manual testa nos pinos 40 (uart1, uart3, uart4 e uart6). [oficial: manual v1.3, p. 225 e p. 236]

## 2. Tensão lógica e 5 V

- **Todos os 28 GPIO operam em 3,3 V.** [oficial: manual v1.3, p. 224]
- O manual não diz se os GPIO toleram 5 V. [não confirmado: tolerância a 5 V não consta do manual]. Por cautela, não ligue sinal de 5 V em pino de GPIO sem conversor de nível ou divisor. Essa é recomendação deste documento, não dado do fabricante.
- **Pinos de 5 V (2 e 4):** o manual descreve o pino 5 V do 40 pinos como forma de alimentar a placa, como alternativa ao Type-C. O manual recomenda o Type-C com 5 V/4 A como forma principal e pede que a fonte aguente a carga. Não ligue ao contrário. [oficial: manual v1.3, p. 144]
- A fonte recomendada no Type-C é de 5 V/5 A, com alimentação fixa de 5 V (sem negociação PD). [oficial: manual v1.3, p. 18]
- A serigrafia da placa mostra um conector separado rotulado **"5V Out"** (com sinais + e -) perto dos pinos 1 e 2. [oficial: manual v1.3, p. 15 e imagem da p. 223]
- **Pinos 1 e 17:** 3,3 V. Consulte a corrente máxima no manual antes de carregar esses pinos. [não confirmado: corrente máxima não consta do manual]

## 3. wiringOP (biblioteca de GPIO em C)

### Já vem instalada

A imagem Linux oficial traz a wiringOP pré-instalada. Para testar, rode `gpio readall`. Se a tabela aparecer, a biblioteca está pronta e não é preciso recompilar. [oficial: manual v1.3, p. 224]

```
orangepi@orangepi:~$ gpio readall
```

A saída mostra as colunas GPIO, wPi, Name, Mode, V, Physical, V, Mode, Name, wPi e GPIO, com o título "PI5 MAX". A saída de referência está na seção 1 (imagem da p. 225).

### Instalação e compilação

O manual pede a branch `next` para a Max. [oficial: manual v1.3, p. 225]

```
orangepi@orangepi:~$ sudo apt update
orangepi@orangepi:~$ sudo apt install -y git
orangepi@orangepi:~$ git clone https://github.com/orangepi-xunlong/wiringOP.git -b next
orangepi@orangepi:~$ cd wiringOP
orangepi@orangepi:~/wiringOP$ sudo ./build clean
orangepi@orangepi:~/wiringOP$ sudo ./build
```

Se o GitHub não estiver acessível, o manual manda usar o código que vem na imagem, em `/usr/src/wiringOP`. [oficial: manual v1.3, p. 225]

Na seção 4.5 (imagem Arch), o manual repete o mesmo fluxo. [oficial: manual v1.3, seção 4.5, p. 444 a 446]

### Uso básico (números wPi, não números físicos)

O manual usa o pino 7 (GPIO1_A7, wPi 2) como exemplo. [oficial: manual v1.3, p. 227]

```
root@orangepi:~/wiringOP# gpio mode 2 out
root@orangepi:~/wiringOP# gpio write 2 0
root@orangepi:~/wiringOP# gpio write 2 1
```

Para ler e configurar pull-up e pull-down, o manual usa o pino 11 (GPIO1_A0, wPi 5). [oficial: manual v1.3, p. 228 e p. 229]

```
root@orangepi:~/wiringOP# gpio mode 5 in
root@orangepi:~/wiringOP# gpio mode 5 up
root@orangepi:~/wiringOP# gpio read 5
root@orangepi:~/wiringOP# gpio mode 5 down
root@orangepi:~/wiringOP# gpio read 5
```

Lembre-se dos pinos com pull-up externo (seção 1). [oficial: manual v1.3, p. 228]

Para testar todos os GPIO de uma vez, o manual cita o programa `blink_all_gpio`, pré-instalado na imagem. Ele alterna os 28 pinos entre 0 V e 3,3 V. Rode com sudo. [oficial: manual v1.3, p. 226]

```
orangepi@orangepi5max:~$ sudo blink_all_gpio
```

### wiringOP-Python

O manual tem uma seção de instalação e uso do wiringOP-Python (seção 3.20, p. 257), com exemplo de RTC DS1307 no barramento `/dev/i2c-2`. [oficial: manual v1.3, p. 257 e p. 268]

## 4. Alternativas à wiringOP

### CircuitPython com Blinka

- Existe uma página do CircuitPython para a Max, em circuitpython.org/blinka/orange_pi_5_max/. Ela descreve a placa e o conector, mas na versão lida não traz comandos de instalação, pinout nem instruções de habilitação de interface. [comunidade/Max]
- O manual da Orange Pi não cita CircuitPython nem Blinka. [oficial: manual v1.3, busca por termo sem resultado]
- Não há comando de instalação confirmado para a Max. Consulte a página do CircuitPython antes de instalar.

### python-periphery

- Biblioteca Python pura, para Python 2 e 3, para acesso a periféricos Linux: GPIO, LED, PWM, SPI, I2C, MMIO e Serial. Licença MIT. [comunidade/Max, uso geral]
- Instalação: `pip install python-periphery`. [comunidade/Max, uso geral]
- A página não menciona Orange Pi nem Rockchip. O uso na Max depende de o sistema expor `gpiochip`, `i2c`, `spidev` e `tty` (ver seção 5 para os overlays que criam esses dispositivos). [não confirmado: não testado na Max]

## 5. Interfaces (SPI, I2C, UART, PWM e CAN) e overlays

### Como habilitar (imagem Ubuntu/Debian)

As interfaces de 40 pinos vêm **desligadas por padrão** no Linux. Para ligar, use `orangepi-config`:

1. `sudo orangepi-config`
2. System
3. Hardware
4. Marque a opção desejada com a barra de espaço
5. Save, Back e Reboot

Os passos estão nas páginas 230 a 231 (SPI), 233 a 234 (I2C), 236 a 237 (UART), 239 a 240 (PWM) e 243 (CAN). [oficial: manual v1.3]

Depois do reboot, confirme o dispositivo (ex.: `ls /dev/spidev*`, `ls /dev/i2c-*`, `ls /dev/ttyS*`, `ls /sys/class/pwm/`, `ifconfig -a`). [oficial: manual v1.3, p. 232, 235, 237, 241 e 244]

Na imagem Ubuntu/Debian, o manual mostra a linha `overlays=` em `/boot/orangepiEnv.txt` para o LCD (`overlays=opi5max-lcd`). [oficial: manual v1.3, p. 335] Para os nomes das tabelas abaixo, o manual não mostra a linha `overlays=` equivalente. [não confirmado: use o `orangepi-config`]

### Como habilitar (imagem Arch, seção 4)

Na imagem OPi OS Arch, o manual edita `/boot/extlinux/extlinux.conf` e usa a linha `FDTOVERLAYS` com caminhos completos em `/dtbs/rockchip/overlay/`, separados por espaço numa só linha. [oficial: manual v1.3, seções 4.2.2, 4.6 e 4.6.7]

```
LABEL Orange Pi
LINUX /Image
FDT /dtbs/rockchip/rk3588-orangepi-5-max.dtb
FDTOVERLAYS /dtbs/rockchip/overlay/rk3588-i2c2-m0.dtbo /dtbs/rockchip/overlay/rk3588-i2c4-m3.dtbo
```

Faça backup antes de editar. Um erro no `extlinux.conf` pode impedir o boot.

### SPI

Pinos (manual v1.3, p. 230):

| Sinal | SPI0_M2 | SPI1_M1 | SPI4_M2 |
|---|---|---|---|
| MOSI | 19 | 40 | 13 |
| MISO | 21 | 38 | 11 |
| CLK | 23 | 29 | 15 |
| CS0 | 24 | 35 | 16 |
| CS1 | 26 | – | – |

- Overlays (nomes da tabela): `spi0-m2-cs0-spidev`, `spi0-m2-cs1-spidev`, `spi0-m2-cs0-cs1-spidev`, `spi1-m1-cs0-spidev`, `spi4-m2-cs0-spidev`. [oficial: manual v1.3, p. 230]
- Imagem Arch: `rk3588-spi0-m2-cs0-cs1-spidev.dtbo`, `rk3588-spi1-m1-cs0-spidev.dtbo`, `rk3588-spi4-m2-cs0-spidev.dtbo`. [oficial: manual v1.3, seção 4.6, linhas de SPI da imagem Arch]
- Dispositivos após habilitar: `/dev/spidev0.0`, `/dev/spidev0.1`, `/dev/spidev1.0`, `/dev/spidev4.0`. [oficial: manual v1.3, p. 232]
- Teste: `sudo spidev_test -v -D /dev/spidev0.0`, com MOSI e MISO em curto para eco. [oficial: manual v1.3, p. 232 e p. 233]

### I2C

Barramentos disponíveis: i2c2, i2c4, i2c5 e i2c8. [oficial: manual v1.3, p. 233]

| Barramento | SDA (pino) | SCL (pino) | Overlay (Ubuntu) | Overlay (Arch) |
|---|---|---|---|---|
| I2C2_M0 | 3 | 5 | `i2c2-m0` | `rk3588-i2c2-m0.dtbo` |
| I2C2_M4 | 11 | 13 | `i2c2-m4` | não listado no manual |
| I2C4_M3 | 15 | 16 | `i2c4-m3` | `rk3588-i2c4-m3.dtbo` |
| I2C5_M2 | 37 | 12 | `i2c5-m2` | `rk3588-i2c5-m2.dtbo` |
| I2C5_M3 | 27 | 28 | `i2c5-m3` | não listado no manual |
| I2C8_M2 | 36 | 32 | `i2c8-m2` | `rk3588-i2c8-m2.dtbo` |

- I2C2_M0 e I2C2_M4, assim como I2C5_M2 e I2C5_M3, são o mesmo barramento ligado a pinos diferentes. Use apenas um de cada par. [oficial: manual v1.3, p. 233]
- Teste: `sudo i2cdetect -y 2` (e 4, 5, 8). [oficial: manual v1.3, p. 235]
- Nota: na imagem Arch o manual lista só i2c2 (M0), i2c4 (M3), i2c5 (M2) e i2c8 (M2). Os overlays de I2C2_M4 e I2C5_M3 não aparecem nas listas Arch. [oficial: manual v1.3, seção 4.6.4, p. 451 e p. 452]

### UART

| Barramento | RX (pino) | TX (pino) | Overlay (Ubuntu) | Overlay (Arch) |
|---|---|---|---|---|
| UART1_M1 | 27 | 28 | `uart1-m1` | `rk3588-uart1-m1.dtbo` |
| UART3_M1 | 33 | 31 | `uart3-m1` | `rk3588-uart3-m1.dtbo` |
| UART4_M2 | 19 | 23 | `uart4-m2` | `rk3588-uart4-m2.dtbo` |
| UART6_M1 | 11 | 13 | `uart6-m1` | `rk3588-uart6-m1.dtbo` |

[oficial: manual v1.3, p. 236 e seção 4.6.5, p. 453]

- Dispositivos: `/dev/ttyS*`. Teste de loopback: `sudo gpio serial /dev/ttySX` (com RX e TX em curto). [oficial: manual v1.3, p. 237 e p. 238]
- Na seção 4.6 (Arch), o exemplo de `FDTOVERLAYS` usa `rk3588-uart0-m2.dtbo` e `rk3588-i2c1-m4.dtbo`. Esses nomes não aparecem nas tabelas do manual. [não confirmado: não use como receita sem conferir]

### PWM

Canais: pwm0, pwm1, pwm3, pwm12, pwm13 e pwm14. [oficial: manual v1.3, p. 238]

| Canal | Pino | Overlay (Ubuntu) | Overlay (Arch) |
|---|---|---|---|
| PWM0_M0 | 5 | `pwm0-m0` | `rk3588-pwm0-m0.dtbo` |
| PWM0_M2 | 15 | `pwm0-m2` | – |
| PWM1_M0 | 3 | `pwm1-m0` | `rk3588-pwm1-m0.dtbo` |
| PWM1_M2 | 16 | `pwm1-m2` | – |
| PWM3_M3 | 7 | `pwm3-m3` | `rk3588-pwm3-m3.dtbo` |
| PWM12_M0 | 31 | `pwm12-m0` | `rk3588-pwm12-m0.dtbo` |
| PWM13_M0 | 33 | `pwm13-m0` | `rk3588-pwm13-m0.dtbo` |
| PWM13_M2 | 27 | `pwm13-m2` | – |
| PWM14_M0 | 35 | `pwm14-m0` | `rk3588-pwm14-m0.dtbo` |
| PWM14_M2 | 32 | `pwm14-m2` | – |

- Só um entre PWM0_M0 e PWM0_M2, PWM1_M0 e PWM1_M2, PWM13_M0 e PWM13_M2, e PWM14_M0 e PWM14_M2 pode estar ativo ao mesmo tempo. [oficial: manual v1.3, p. 238]
- Teste pelo sysfs: depois de habilitar, `/sys/class/pwm/` ganha um `pwmchipX`. O manual relaciona o pwmchip ao endereço base (pwm3 = fd8b0030, pwmchip0). [oficial: manual v1.3, p. 241 e p. 242]
- O manual exemplifica uma onda de 50 Hz no pwm3, com period 20000000 ns e duty_cycle 1000000 ns. [oficial: manual v1.3, p. 242]
- Na wiringOP, o PWM usa `gpio` (seção 3.19, p. 250 a 251) e a API em `/usr/src/wiringOP/examples/pwm.c`. [oficial: manual v1.3, p. 250 e p. 251]
- A ventoinha também é controlada por PWM (seção 7). O sinal `FAN_CTL_H` fica em GPIO3_D1 / PWM9_M2 [oficial: esquemático V1.1, p. 10], e o kernel mainline usa `pwm9` para ela.

### CAN

Barramentos: CAN0 e CAN1. [oficial: manual v1.3, p. 243]

- Habilitação (Ubuntu/Debian): `orangepi-config` > System > Hardware > CAN. Depois, `sudo ifconfig -a` deve mostrar `can0` e `can1`. [oficial: manual v1.3, p. 243 e p. 244]
- Habilitação (Arch): `rk3588-can0-m0.dtbo` e `rk3588-can1-m1.dtbo` em `FDTOVERLAYS`. [oficial: manual v1.3, seção 4.6.7] Note que o overlay do CAN1 é `can1-m1`, enquanto a tabela de pinos lista CAN1_RX_M0 e CAN1_TX_M0. [não confirmado: divergência de nome]
- Pinos de CAN1: RX no 31, TX no 33. CAN0: ver a divergência na seção 1.
- Teste validado pelo manual com analisador CANalyst-II, a 1000 kbps. O transceptor CAN precisa de 3,3 V, GND, TX e RX ligados aos pinos do 40 pinos. [oficial: manual v1.3, p. 245 a 248]

## 6. Console serial de depuração

O manual descreve a porta de depuração em duas partes: a placa e o adaptador. [oficial: manual v1.3, p. 136 a 142]

### Adaptador

- Adaptador **USB para TTL de 3,3 V**. Não use 5 V. [oficial: manual v1.3, p. 136]
- O manual recomenda **CH340**. Não use CP2102 nem PL2303. [oficial: manual v1.3, p. 136]
- O adaptador precisa suportar **1500000 baud**. Confirme antes de comprar. [oficial: manual v1.3, p. 136]
- O jumper do adaptador deve estar colocado, como na figura. O pino 5 V do adaptador não é ligado. [oficial: manual v1.3, p. 136, imagem]

### Pinos na placa

| Sinal na placa | Pino do 40 pinos | Função | Fonte |
|---|---|---|---|
| GND | 6 | Terra | imagem p. 136 e readall p. 225 |
| TXD (saída da placa) | 8 | UART2_TX_M0 | imagem p. 136 e readall p. 225 (TXD.2) |
| RXD (entrada da placa) | 10 | UART2_RX_M0 | imagem p. 136 e readall p. 225 (RXD.2) |

Ligação (manual v1.3, p. 136 e p. 137):

- GND do adaptador vai ao GND do 40 pinos (pino 6).
- RX do adaptador vai ao TXD da placa (pino 8).
- TX do adaptador vai ao RXD da placa (pino 10).

O manual diz que, se não houver saída, basta inverter TX e RX. [oficial: manual v1.3, p. 137]

### Configuração do terminal

- Velocidade: **1500000 baud**. Dados: 8 bits. Paridade: nenhuma. Stop bits: 1. Controle de fluxo: nenhum. [oficial: manual v1.3, p. 138 e p. 139]
- Linux: o adaptador aparece como `/dev/ttyUSB0`. O manual usa PuTTY (`sudo apt-get install -y putty`), com Connection type Serial. [oficial: manual v1.3, p. 137 a 139]
- Windows: o manual usa MobaXterm (versão portátil), em sessão de tipo serial, com a mesma velocidade. [oficial: manual v1.3, p. 140 a 142]
- A wiki de usuário sugere `screen -fnL /dev/ttyUSB0 1500000`. O manual não cita `screen`. As opções `-f`, `-n` e `-L` não foram conferidas no `man screen`; confira antes de usar. [comunidade/Max; não confirmado]

Depois de ligar a placa, a saída do boot aparece no terminal. [oficial: manual v1.3, p. 139]

## 7. Conector da ventoinha

- Conector de **2 pinos, passo 1,25 mm**, para ventoinha de **5 V**. [oficial: manual v1.3, p. 15 (rótulo "Fan (5V)") e p. 19]
- A ventoinha pode ser ligada, desligada e ter a velocidade ajustada por PWM. [oficial: manual v1.3, p. 19]
- Ordem dos pinos: **pino 1 é + (5 V) e pino 2 é −**. O lado negativo é chaveado por um MOSFET, comandado por `FAN_CTL_H`. [oficial: esquemático V1.1, p. 11] Confira o "+" na serigrafia antes de ligar.
- Versões do Linux com o suporte à ventoinha: a tabela de funções lista "FAN Interface" como OK em Ubuntu/Debian (Linux 5.10 e 6.1). [oficial: manual v1.3, p. 146 e p. 147]

## 8. Conector da bateria RTC

- Conector de **2 pinos, passo 1,25 mm**, rotulado **"RTC Connector (3V 2Pin 1.25mm)"** na vista da placa. [oficial: manual v1.3, p. 15]
- Tensão: **3 V**. [oficial: manual v1.3, p. 15]
- Polaridade (qual pino é + e qual é -) e tipo de bateria recomendado: **não confirmado**. A imagem da p. 15 não mostra a numeração. Confirme na serigrafia da placa antes de ligar.
- O "RTC Chip" aparece nas tabelas de funções. Não confirmado como tratar o RTC interno. [oficial: manual v1.3, p. 146 e p. 147]
- Não confunda com o RTC externo ligado ao I2C do 40 pinos (exemplo DS1307, p. 268). [oficial: manual v1.3, p. 268]

## 9. Divergências entre fontes

- **CAN0:** resolvido a favor da imagem da p. 223 (pino 3 = RX, pino 5 = TX), com o esquemático. O texto da p. 245 está trocado.
- **Rótulo do pino 36:** GPIO1_D7 na imagem da p. 223; GPIO3_D7 no `gpio readall` da p. 228. O número 63 indica GPIO1_D7. Ver seção 1.
- **Overlay de CAN1 (Arch):** `can1-m1` no texto, CAN1_*_M0 na tabela. [não confirmado]
- **Exemplo de overlay (Arch, seção 4.6):** usa `i2c1-m4` e `uart0-m2`, que não aparecem nas tabelas. [não confirmado]
- **Alimentação pelo 5 V do 40 pinos:** o manual cita 5 V/4 A (p. 144) e 5 V/5 A no Type-C (p. 18). Use a fonte de 5 V/5 A no Type-C, como o manual recomenda.
- **Sistema operacional:** o manual cita Android 13 (seção 7, p. 485). A fonte da comunidade que citava Android 12 está errada para a Max. [oficial: manual v1.3, p. 485]

## 10. Lacunas (o que continua sem resposta)

- Ordem física dos pinos da ventoinha (+ e -). Procurado no manual (seção 7 e imagem da p. 19): não consta.
- Polaridade do conector RTC. Procurado no manual (p. 15 e imagem): não consta.
- Corrente máxima dos pinos 3,3 V e 5 V. Procurado no manual (p. 223 a 228): não consta.
- Tolerância dos GPIO a 5 V. Procurado no manual: não consta.
- Canal PWM do ventilador. Procurado no manual (seção 7): não consta.
- Forma exata de ativar os overlays de 40 pinos na imagem Ubuntu/Debian sem o `orangepi-config` (linha `overlays=` em `/boot/orangepiEnv.txt`). Procurado no manual (p. 230 a 245 e p. 335): não consta.
- CAN0 (pinos 3 e 5). Ver seção 1.
- Nome `/dev` do console de depuração (`ttyS2` ou outro). Procurado no manual (p. 136 a 142 e seções 3.18 e 4.6): não consta.

## 11. Aviso sobre comandos destrutivos

- Uma página de instalação de sistema da wiki de usuário da Max traz comandos `dd`, `sfdisk` e `mkfs.ext4` sobre `/dev/sd?`. Eles **apagam ou sobrescrevem o dispositivo de destino**. Não os copiei aqui. Use-os apenas com o dispositivo correto confirmado.
- O manual também descreve gravação com `dd` para `/dev/mtdblock0` (SPI flash, seção 2.6.2) e para eMMC (seção 2.5.2). Confirme o dispositivo antes. [oficial: manual v1.3, seções 2.5.2 e 2.6.2]
- Os comandos desta seção (`apt`, `git clone`, `./build`, `gpio`, `screen`, `pip install`) não apagam dados do usuário.

## Fontes

- Esquemático oficial V1.1: https://drive.google.com/file/d/1cAKUBHF2yXtmmLJaVtK17nOKoMlI02wA/view?usp=sharing
- Manual do usuário da Orange Pi 5 Max, v1.3 (Shenzhen Xunlong Software Co., Ltd). Citado como "manual v1.3". Páginas e seções citadas no texto. Fonte principal deste documento.
- https://circuitpython.org/blinka/orange_pi_5_max/ (comunidade/Max)
- https://github.com/bunnyamin/bunnix/wiki/orangepi_5max (comunidade/Max; a parte de comandos `dd` e serial foi conferida com o manual)
- https://github.com/orangepi-xunlong/wiringOP (branch `next`, citada no manual)
- https://www.cnx-software.com/2024/08/01/rockchip-rk3588-powered-orange-pi-5-max-sbc-features-up-to-16gb-lpddr5-2-5gbe-onboard-wifi-6e-and-bluetooth-5-3/ (comunidade/Max)
- https://learn.adafruit.com/circuitpython-on-orangepi-linux/circuitpython-orangepi (comunidade, uso geral)
- https://github.com/vsergeev/python-periphery (comunidade, uso geral)
