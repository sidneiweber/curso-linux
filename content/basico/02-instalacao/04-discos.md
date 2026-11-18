+++
title = "Discos e partições"
weight = 4
description = "Nomenclatura de discos e partições, e como consultá-los."
+++

Os discos aparecem como arquivos dentro de `/dev`:

| Nome | Dispositivo |
|---|---|
| `/dev/sda`, `/dev/sdb` … | 1º, 2º … disco SATA/SCSI/USB |
| `/dev/sda1`, `/dev/sda2` | 1ª e 2ª partição do disco `sda` |
| `/dev/nvme0n1` | 1º SSD NVMe |
| `/dev/nvme0n1p1` | 1ª partição do SSD NVMe |
| `/dev/vda` | Disco virtual (máquinas virtuais KVM) |
| `/dev/sr0` | Leitor de CD/DVD |

## Montagem

Para acessar o conteúdo de uma partição, ela precisa ser **montada** em um diretório
(ponto de montagem). Pendrives costumam ser montados automaticamente em `/media/<usuário>/`.

## Comandos úteis

```bash
lsblk            # lista discos e partições em árvore
df -h            # espaço usado/livre das partições montadas
sudo fdisk -l    # detalhes das tabelas de partição
mount            # mostra o que está montado
cat /etc/fstab   # partições montadas automaticamente no boot
```
