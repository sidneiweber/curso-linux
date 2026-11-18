+++
title = "udev: dispositivos dinâmicos"
weight = 3
description = "Como o Linux cria /dev, nomes persistentes e regras próprias."
+++

O **udev** (hoje parte do systemd: `systemd-udevd`) recebe do kernel os eventos de
dispositivos conectados e removidos. Ele cria os arquivos em `/dev`, carrega drivers,
ajusta permissões e cria **nomes persistentes**.

## Vendo os eventos

```bash
udevadm monitor --udev --property     # conecte um pendrive e observe
```

## Nomes persistentes

`/dev/sdb` pode virar `/dev/sdc` amanhã. O udev cria links que não mudam:

```bash
ls -l /dev/disk/by-uuid/ /dev/disk/by-id/ /dev/disk/by-label/
```

É o mesmo motivo de usar `UUID=` no `fstab`. Interfaces de rede também têm nomes
previsíveis baseados na posição física (`enp3s0`, `wlp2s0`) em vez de `eth0`/`eth1`.

## Atributos de um dispositivo

```bash
udevadm info --query=all --name=/dev/sdb
udevadm info --attribute-walk --name=/dev/sdb | less
```

## Regras próprias

Regras ficam em `/etc/udev/rules.d/`. Exemplo: dar a um pendrive específico um nome
fixo e liberar acesso a um grupo:

```bash
sudo tee /etc/udev/rules.d/99-pendrive-backup.rules <<'R'
SUBSYSTEM=="block", ENV{ID_FS_UUID}=="7A1B-2C3D", SYMLINK+="backup", GROUP="backup", MODE="0660"
R
sudo udevadm control --reload
sudo udevadm trigger
ls -l /dev/backup
```

## Exercícios

1. Gere um relatório `inxi -Fxz` e identifique o driver da sua placa de vídeo e de rede.
2. Verifique a saúde de todos os discos com `smartctl -H`.
3. Escreva uma regra udev que crie `/dev/meupendrive` para o seu pendrive.
