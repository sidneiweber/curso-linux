+++
title = "Informações do sistema"
weight = 2
description = "Datas, kernel, /proc, processos, rede e console."
+++

## Calcular datas

```bash
date                          # data e hora atuais
date --date '30 days'         # daqui a 30 dias
date --date '15 days ago'     # 15 dias atrás
date --date 'next friday'
date +%d/%m/%Y                # formato personalizado: 17/11/2026
```

## Alterar data e hora

```bash
timedatectl                                     # situação atual (fuso, NTP)
sudo timedatectl set-time "2026-11-17 14:30:00"
sudo timedatectl set-timezone America/Sao_Paulo
sudo hwclock --systohc                          # grava a hora no relógio da placa-mãe (BIOS)
```

{{% notice note %}}
Hoje em dia o relógio normalmente é sincronizado pela internet (NTP). Se `set-time`
reclamar, desative antes com `sudo timedatectl set-ntp false`.
O antigo `clock -w` corresponde ao `hwclock --systohc`.
{{% /notice %}}

## Informações do kernel e do hardware via /proc

```bash
uname -r                  # versão do kernel (ou: cat /proc/version)
cat /proc/cpuinfo         # processadores   (ou: lscpu)
cat /proc/meminfo         # memória         (ou: free -h)
cat /proc/partitions      # partições       (ou: lsblk)
cat /proc/mounts          # sistemas de arquivos montados (ou: findmnt)
cat /proc/interrupts      # interrupções (IRQs)
cat /proc/ioports         # portas de entrada e saída
cat /proc/modules         # módulos carregados (ou: lsmod)
```

## Processos

```bash
ps aux            # todos os processos
pstree            # árvore de processos
pstree -p | less  # com os PIDs
```

## Rede

| Tarefa | Comando atual | Comando antigo |
|---|---|---|
| Conexões TCP/UDP abertas | `ss -tuna` | `netstat -naut` |
| Portas escutando | `ss -tulpn` | `netstat -tulpn` |
| Tabela de roteamento | `ip route` | `route -n` |
| Endereços IP | `ip addr` | `ifconfig` |

### Problemas de DNS

Antigamente se editava `/etc/resolv.conf` diretamente. Nas distribuições atuais esse
arquivo costuma ser gerado automaticamente (systemd-resolved / NetworkManager):

```bash
resolvectl status                       # DNS em uso
nmcli connection modify "Minha Rede" ipv4.dns "1.1.1.1 8.8.8.8"
nmcli connection up "Minha Rede"
```

## Histórico de comandos

| Atalho/comando | Ação |
|---|---|
| `↑` ou `Ctrl` + `P` | comando anterior |
| `Ctrl` + `R` | busca no histórico (digite parte do comando) |
| `history` | lista o histórico |
| `!123` | executa o comando nº 123 do histórico |
| `!!` | repete o último comando (ex.: `sudo !!`) |

## Console (modo texto)

```bash
setterm -blank 5                                  # apaga a tela após 5 min sem uso
setterm -background blue -foreground white -store # muda as cores
sudo dpkg-reconfigure console-setup               # muda a fonte do console
```

## Volume do áudio

```bash
alsamixer                       # mixer no terminal
wpctl set-volume @DEFAULT_AUDIO_SINK@ 50%   # PipeWire (distros atuais)
```
