+++
title = "Coletando informações"
weight = 1
description = "lscpu, lspci, lsusb, lshw, dmidecode, sensores e SMART."
+++

## Visão geral

```bash
sudo apt install lshw hwinfo inxi
inxi -Fxz              # resumo completo e legível (-z oculta dados pessoais)
sudo lshw -short       # árvore de hardware
sudo lshw -html > hardware.html
```

## Por componente

| Componente | Comando |
|---|---|
| CPU | `lscpu`, `nproc` |
| Memória | `free -h`, `sudo dmidecode -t memory` (pentes, slots, velocidade) |
| Placa-mãe / BIOS | `sudo dmidecode -t baseboard -t bios` |
| Dispositivos PCI | `lspci`, `lspci -k` (mostra o driver em uso) |
| Dispositivos USB | `lsusb`, `lsusb -t` |
| Discos | `lsblk -o NAME,MODEL,SIZE,ROTA,TRAN` |
| Rede | `ip link`, `lspci -k \| grep -A3 -i net` |
| Vídeo | `lspci -k \| grep -A3 -i vga` |

Exemplo — qual driver cuida da placa de rede?

```bash
lspci -k | grep -A3 -i ethernet
# 02:00.0 Ethernet controller: Intel Corporation I211 Gigabit Network Connection
#         Kernel driver in use: igb
```

## Temperatura e saúde

```bash
sudo apt install lm-sensors smartmontools
sudo sensors-detect      # uma vez, aceitando os padrões
sensors                  # temperaturas e ventoinhas

sudo smartctl -H /dev/sda      # saúde do disco (PASSED/FAILED)
sudo smartctl -a /dev/nvme0    # todos os atributos SMART
```

Em discos, fique de olho em **Reallocated_Sector_Ct**, **Current_Pending_Sector** e, em
NVMe, **Percentage Used** e **Media and Data Integrity Errors**.
