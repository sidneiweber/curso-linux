+++
title = "Particionamento"
weight = 1
description = "MBR x GPT, fdisk e parted."
+++

## MBR ou GPT?

| | MBR (DOS) | GPT |
|---|---|---|
| Tamanho máximo do disco | 2 TiB | Na prática, ilimitado |
| Número de partições | 4 primárias (ou 3 + estendida com lógicas) | 128 por padrão |
| Firmware | BIOS | UEFI (e BIOS com uma partição *BIOS boot*) |
| Redundância | Não | Cópia da tabela no fim do disco |

Para discos novos, use **GPT**.

## Conhecendo o disco

```bash
lsblk -f                 # discos, partições, sistemas de arquivos e UUIDs
sudo fdisk -l /dev/sdb   # tabela de partição
sudo blkid               # UUID e tipo de cada partição
```

## fdisk (interativo)

```bash
sudo fdisk /dev/sdb
```

| Tecla | Ação |
|---|---|
| `g` | Cria uma tabela **GPT** vazia (`o` para MBR) |
| `n` | Nova partição (tamanho como `+10G`) |
| `t` | Muda o tipo (ex.: `swap`, `lvm`, `linux`) |
| `p` | Mostra a tabela |
| `d` | Apaga uma partição |
| `w` | **Grava** e sai — até aqui nada foi alterado |
| `q` | Sai sem gravar |

## parted (bom para scripts)

```bash
sudo parted /dev/sdb --script mklabel gpt \
  mkpart dados ext4 1MiB 10GiB \
  mkpart backup xfs 10GiB 100%
sudo parted /dev/sdb print
```

Depois de alterar a tabela de um disco em uso, avise o kernel:

```bash
sudo partprobe /dev/sdb
```

{{% notice tip "Ferramentas gráficas" %}}
O **GParted** faz tudo isso com interface gráfica e ainda redimensiona partições
existentes. Ele também está disponível como pendrive *live*.
{{% /notice %}}
