+++
title = "LVM"
weight = 5
description = "Volumes físicos, grupos de volumes, volumes lógicos e como redimensioná-los."
+++

Com partições comuns, aumentar um disco cheio é complicado. O **LVM** (*Logical Volume
Manager*) cria uma camada entre os discos e os sistemas de arquivos:

```text
Discos/partições  →  PV (physical volume)
         vários PVs →  VG (volume group): uma "piscina" de espaço
              VG    →  LVs (logical volumes): as "partições" que você formata e monta
```

Assim dá para **aumentar volumes com o sistema no ar**, juntar vários discos e tirar snapshots.

```bash
sudo apt install lvm2
```

## Criando

```bash
sudo pvcreate /dev/sdb /dev/sdc              # marca os discos como PVs
sudo vgcreate vg_dados /dev/sdb /dev/sdc     # junta em um VG
sudo lvcreate -n lv_web -L 20G vg_dados      # LV de 20 GiB
sudo lvcreate -n lv_bkp -l 100%FREE vg_dados # LV com todo o resto

sudo mkfs.ext4 /dev/vg_dados/lv_web
sudo mount /dev/vg_dados/lv_web /var/www
```

## Consultando

```bash
sudo pvs; sudo vgs; sudo lvs    # resumo
sudo vgdisplay vg_dados         # detalhes
lsblk                           # mostra a hierarquia disco → LV
```

## Aumentando um volume (com o sistema no ar)

```bash
sudo lvextend -L +10G -r /dev/vg_dados/lv_web   # -r redimensiona o sistema de arquivos junto
```

Se o VG não tiver espaço livre, adicione um disco:

```bash
sudo pvcreate /dev/sdd
sudo vgextend vg_dados /dev/sdd
```

{{% notice warning "Diminuir é diferente" %}}
ext4 pode ser reduzido, mas **desmontado** e com cuidado (`lvreduce -r`). XFS **não**
pode ser reduzido. Faça backup antes.
{{% /notice %}}

## Snapshots

Um snapshot congela o estado de um LV naquele instante — ideal para fazer backup
consistente ou testar uma atualização:

```bash
sudo lvcreate -s -n web_snap -L 2G /dev/vg_dados/lv_web
sudo mount -o ro /dev/vg_dados/web_snap /mnt/snap
# ... copie o backup a partir de /mnt/snap ...
sudo umount /mnt/snap && sudo lvremove /dev/vg_dados/web_snap
```

O snapshot só guarda as **diferenças**; se ele encher (`-L 2G`), torna-se inválido.
