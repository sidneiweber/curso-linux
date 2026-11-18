+++
title = "Módulos"
weight = 2
description = "lsmod, modprobe, parâmetros e blacklist."
+++

A maioria dos drivers não fica "dentro" do kernel, e sim em **módulos** (`.ko`) carregados
sob demanda — normalmente de forma automática quando o hardware é detectado.

```bash
lsmod                       # módulos carregados
lsmod | grep -i snd         # só os de som
modinfo e1000e              # descrição, autor, parâmetros aceitos
ls /lib/modules/$(uname -r)/kernel/drivers
```

## Carregando e descarregando

```bash
sudo modprobe loop          # carrega (e as dependências)
sudo modprobe -r loop       # descarrega, se não estiver em uso
```

Prefira `modprobe` a `insmod`/`rmmod`: ele resolve dependências automaticamente.

## Carregando no boot

```bash
echo 'loop' | sudo tee /etc/modules-load.d/loop.conf
```

## Parâmetros de módulos

```bash
modinfo -p snd_hda_intel                  # parâmetros disponíveis
cat /sys/module/snd_hda_intel/parameters/power_save
```

Para definir de forma permanente, crie um arquivo em `/etc/modprobe.d/`:

```bash
echo 'options snd_hda_intel power_save=1' | sudo tee /etc/modprobe.d/som.conf
```

## Impedindo que um módulo carregue (blacklist)

Útil para trocar de driver (ex.: `nouveau` → NVIDIA proprietário) ou desativar hardware:

```bash
echo 'blacklist pcspkr' | sudo tee /etc/modprobe.d/blacklist-beep.conf
sudo update-initramfs -u    # se o módulo também está no initramfs
```

## DKMS

Drivers de terceiros (VirtualBox, NVIDIA, alguns Wi-Fi) precisam ser recompilados para cada
novo kernel. O **DKMS** faz isso automaticamente sempre que um kernel é instalado:

```bash
dkms status
```
