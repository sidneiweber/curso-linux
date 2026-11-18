+++
title = "Sistemas de arquivos"
weight = 2
description = "ext4, XFS e Btrfs; mkfs, fsck e tune2fs."
+++

Uma partição só guarda arquivos depois de **formatada** com um sistema de arquivos.

| Sistema | Destaques | Quando usar |
|---|---|---|
| **ext4** | Estável, maduro, pode crescer e encolher | Padrão do Debian/Ubuntu; uso geral |
| **XFS** | Ótimo com arquivos grandes e paralelismo; só cresce | Padrão do RHEL; servidores de dados |
| **Btrfs** | Snapshots, compressão, checksums, subvolumes | Padrão do Fedora/openSUSE; desktops |
| **vfat / exFAT** | Compatível com Windows/macOS | Pendrives e a partição EFI |

## Formatando

```bash
sudo mkfs.ext4 -L dados /dev/sdb1     # -L define um rótulo
sudo mkfs.xfs  -L backup /dev/sdb2
sudo mkfs.vfat -F 32 /dev/sdc1
```

## Verificando e reparando

```bash
sudo umount /dev/sdb1
sudo fsck -f /dev/sdb1         # ext4: nunca rode em partição montada!
sudo xfs_repair /dev/sdb2      # XFS
```

{{% notice warning "Partição montada" %}}
Rodar `fsck` em um sistema de arquivos montado em leitura e escrita pode **corromper** os
dados. Para checar a raiz, use um pendrive *live* ou o `emergency.target`.
{{% /notice %}}

## Ajustes no ext4

```bash
sudo tune2fs -l /dev/sdb1 | less      # informações detalhadas
sudo tune2fs -L novo_rotulo /dev/sdb1 # muda o rótulo
sudo tune2fs -m 1 /dev/sdb1           # reserva só 1% para o root (padrão: 5%)
sudo resize2fs /dev/sdb1              # ajusta ao novo tamanho da partição
```

## Inodes

Cada arquivo usa um **inode** (que guarda dono, permissões, datas e onde estão os dados).
Um disco pode ficar "cheio" mesmo com espaço livre, se acabarem os inodes — comum com
milhões de arquivos pequenos:

```bash
df -i          # uso de inodes por partição
ls -i arquivo  # número do inode de um arquivo
```
