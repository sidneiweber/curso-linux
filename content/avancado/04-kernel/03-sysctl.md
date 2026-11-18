+++
title = "Parâmetros em tempo de execução"
weight = 3
description = "/proc/sys, /sys e sysctl."
+++

Muitos comportamentos do kernel podem ser ajustados **sem reiniciar**, por meio dos
pseudo-sistemas de arquivos `/proc/sys` e `/sys`.

## sysctl

```bash
sysctl -a | less                       # todos os parâmetros
sysctl net.ipv4.ip_forward             # um parâmetro
cat /proc/sys/net/ipv4/ip_forward      # o mesmo, lido como arquivo
sudo sysctl -w net.ipv4.ip_forward=1   # altera até o próximo boot
```

O nome usa pontos no lugar das barras: `net.ipv4.ip_forward` ↔ `/proc/sys/net/ipv4/ip_forward`.

## Tornando permanente

Crie um arquivo em `/etc/sysctl.d/` e aplique:

```bash
sudo tee /etc/sysctl.d/90-servidor.conf <<'CONF'
net.ipv4.ip_forward = 1
vm.swappiness = 10
fs.inotify.max_user_watches = 524288
CONF
sudo sysctl --system
```

## Parâmetros comuns

| Parâmetro | Para quê |
|---|---|
| `net.ipv4.ip_forward` | Permite que a máquina **roteie** pacotes (roteador, VPN, containers) |
| `vm.swappiness` | Tendência a usar swap |
| `fs.file-max` | Máximo de arquivos abertos no sistema |
| `fs.inotify.max_user_watches` | Arquivos monitorados (IDEs, sincronizadores) |
| `kernel.hostname` | Nome da máquina |
| `net.ipv4.icmp_echo_ignore_all` | Ignora `ping` |
| `kernel.sysrq` | Habilita as teclas mágicas SysRq |

## /sys

O `/sys` expõe dispositivos e drivers. Alguns exemplos:

```bash
cat /sys/class/power_supply/BAT0/capacity          # bateria (%)
cat /sys/block/sda/queue/scheduler                 # escalonador de I/O do disco
echo 50 | sudo tee /sys/class/backlight/*/brightness   # brilho da tela
```
