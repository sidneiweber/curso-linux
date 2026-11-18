+++
title = "Montagem e /etc/fstab"
weight = 3
description = "mount, umount, UUIDs, opções de montagem e fstab."
+++

## Montagem manual

```bash
sudo mkdir -p /mnt/dados
sudo mount /dev/sdb1 /mnt/dados
sudo mount -o ro /dev/sdb1 /mnt/dados   # somente leitura
sudo umount /mnt/dados
```

Se o `umount` disser *target is busy*, algum processo está usando a pasta:

```bash
sudo lsof +D /mnt/dados
sudo fuser -vm /mnt/dados
```

## O que está montado?

```bash
findmnt              # árvore de montagens
findmnt /            # de onde vem a raiz
df -hT               # espaço e tipo de cada sistema montado
```

## Montagem automática: /etc/fstab

Cada linha do `/etc/fstab` descreve uma montagem feita no boot:

```text
# <dispositivo>                           <ponto>     <tipo> <opções>          <dump> <fsck>
UUID=3e6be9de-8139-4c2b-8f0d-1a2b3c4d5e6f /           ext4   errors=remount-ro 0      1
UUID=7A1B-2C3D                            /boot/efi   vfat   umask=0077        0      1
UUID=9b1f0c52-5a7e-4d41-a0b8-6f2e8d9c1a00 /mnt/dados  ext4   defaults,noatime  0      2
/swapfile                                 none        swap   sw                0      0
```

- **Dispositivo**: prefira `UUID=` (obtido com `blkid`). Nomes como `/dev/sdb1` podem
  mudar se você trocar discos de porta.
- **fsck**: `1` para a raiz, `2` para os demais, `0` para não checar.

| Opção | Efeito |
|---|---|
| `defaults` | rw, suid, dev, exec, auto, nouser, async |
| `noatime` | Não grava a data de acesso a cada leitura (menos escrita) |
| `ro` | Somente leitura |
| `noexec` | Impede executar programas (bom para `/tmp` e dados) |
| `nofail` | Não trava o boot se o disco não estiver presente |
| `user` | Permite que usuários comuns montem |

## Testando antes de reiniciar

Um erro no `fstab` pode impedir o boot. **Sempre** teste:

```bash
sudo findmnt --verify     # valida a sintaxe
sudo systemctl daemon-reload
sudo mount -a             # monta tudo o que ainda não estiver montado
```

{{% notice tip "Discos removíveis" %}}
Para discos externos que nem sempre estão conectados, use `nofail` e
`x-systemd.device-timeout=5s` — assim o boot não fica 90 segundos esperando por eles.
{{% /notice %}}
