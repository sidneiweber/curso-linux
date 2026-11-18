+++
title = "Swap"
weight = 4
description = "Partição ou arquivo de swap, swappiness e zram."
+++

A **swap** é uma área do disco usada como extensão da memória RAM: páginas de memória pouco
usadas são movidas para lá, liberando RAM. Também é usada na **hibernação**.

```bash
swapon --show
free -h
```

## Arquivo de swap (o jeito mais simples)

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

{{% notice note "Btrfs" %}}
Em Btrfs, crie o arquivo com `sudo btrfs filesystem mkswapfile --size 2G /swapfile`.
{{% /notice %}}

## Partição de swap

```bash
sudo mkswap /dev/sdb3
sudo swapon /dev/sdb3
# no fstab: UUID=<uuid>  none  swap  sw  0  0
```

## Swappiness

Controla o quanto o kernel prefere usar swap (0–200; padrão 60). Servidores de banco de
dados costumam usar valores baixos:

```bash
cat /proc/sys/vm/swappiness
sudo sysctl vm.swappiness=10                                 # temporário
echo 'vm.swappiness=10' | sudo tee /etc/sysctl.d/99-swap.conf # permanente
```

## zram

O **zram** cria uma swap **comprimida na própria RAM** — bem mais rápida que disco e muito
útil em máquinas com pouca memória. Fedora e Ubuntu recentes já o habilitam por padrão.

```bash
sudo apt install systemd-zram-generator
zramctl
```
