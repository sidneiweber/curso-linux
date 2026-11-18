+++
title = "O processo de boot"
weight = 1
description = "Firmware, GRUB, kernel, initramfs e systemd — a sequência completa."
+++

O boot de um Linux moderno acontece em etapas, cada uma passando o controle para a próxima:

```text
Firmware (UEFI/BIOS) → GRUB → kernel + initramfs → systemd (PID 1) → serviços → login
```

## 1. Firmware: UEFI ou BIOS

Ao ligar, a placa-mãe executa o firmware, que testa o hardware (POST) e procura algo para iniciar.

| | BIOS (legado) | UEFI |
|---|---|---|
| Tabela de partição | MBR | GPT |
| Onde está o carregador | Primeiros 446 bytes do disco | Arquivo `.efi` na partição EFI (ESP) |
| Secure Boot | Não | Sim |

Para saber como o seu sistema iniciou:

```bash
[ -d /sys/firmware/efi ] && echo UEFI || echo BIOS
```

Em UEFI, a partição EFI fica montada em `/boot/efi`. Para ver as entradas de boot do firmware:

```bash
sudo efibootmgr -v
```

## 2. Gerenciador de boot: GRUB 2

O **GRUB** mostra o menu de sistemas/kernels, carrega o kernel escolhido e o *initramfs*
na memória e passa para o kernel os **parâmetros de linha de comando**. Veremos como
configurá-lo na próxima página.

## 3. Kernel e initramfs

O kernel inicializa a CPU, a memória e os drivers essenciais. Mas para montar o disco raiz
ele pode precisar de módulos (driver do controlador, LVM, criptografia…) que estão… no disco raiz.
O **initramfs** resolve esse problema: é um pequeno sistema de arquivos temporário, carregado
na memória junto com o kernel, que contém só o necessário para encontrar e montar a raiz.

```bash
ls -lh /boot                 # vmlinuz-* (kernel) e initrd.img-* (initramfs)
cat /proc/cmdline            # parâmetros com que o kernel atual foi iniciado
lsinitramfs /boot/initrd.img-$(uname -r) | head
sudo update-initramfs -u     # recria o initramfs do kernel atual
```

## 4. systemd: o processo número 1

Com a raiz montada, o kernel executa `/sbin/init` — que hoje é um link para o **systemd**.
Ele é o primeiro processo (PID 1) e o pai de todos os outros:

```bash
ps -p 1 -o pid,comm
readlink -f /sbin/init
pstree -p | head
```

O systemd lê suas **units** e sobe, em paralelo, tudo o que for necessário para chegar ao
**target** padrão (por exemplo, o modo gráfico).

{{% notice info "E o SysV init?" %}}
Distribuições antigas usavam o **SysV init**, com scripts em `/etc/init.d/` e *runlevels*
numerados configurados em `/etc/inittab`. O systemd substituiu esse modelo na maioria das
distribuições a partir de 2015, mas mantém compatibilidade: comandos como `service` e
`init 3` ainda funcionam, traduzidos para o equivalente no systemd.
{{% /notice %}}

## Quanto tempo leva cada etapa?

```bash
systemd-analyze                  # tempo de firmware, loader, kernel e userspace
systemd-analyze blame | head     # serviços mais lentos
systemd-analyze critical-chain   # o caminho crítico até o target padrão
```
