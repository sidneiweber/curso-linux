+++
title = "Configurando o GRUB"
weight = 2
description = "/etc/default/grub, parâmetros do kernel, update-grub e recuperação."
+++

A configuração que o GRUB lê de fato é `/boot/grub/grub.cfg`, mas **nunca edite esse arquivo**:
ele é gerado automaticamente. As mudanças são feitas em:

| Arquivo | Para quê |
|---|---|
| `/etc/default/grub` | Opções gerais: tempo do menu, entrada padrão, parâmetros do kernel |
| `/etc/grub.d/` | Scripts que montam o menu (`40_custom` é para entradas próprias) |

## Opções mais usadas

```bash
sudo nano /etc/default/grub
```

```ini
GRUB_DEFAULT=0                      # entrada padrão (0 = primeira) ou "saved"
GRUB_TIMEOUT=5                      # segundos com o menu na tela
GRUB_TIMEOUT_STYLE=menu             # menu | hidden | countdown
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"   # parâmetros só no boot normal
GRUB_CMDLINE_LINUX=""               # parâmetros em todas as entradas
```

Depois de editar, gere o `grub.cfg` novamente:

```bash
sudo update-grub      # Debian/Ubuntu (atalho para grub-mkconfig -o /boot/grub/grub.cfg)
```

## Parâmetros úteis do kernel

| Parâmetro | Efeito |
|---|---|
| `quiet` / `splash` | Esconde mensagens e mostra a tela gráfica de boot |
| `systemd.unit=multi-user.target` | Inicia em modo texto |
| `systemd.unit=rescue.target` | Modo de recuperação (monousuário) |
| `init=/bin/bash` | Pula o systemd e abre um shell direto (emergência) |
| `nomodeset` | Desativa o driver de vídeo do kernel (problemas de vídeo) |
| `mem=4G`, `maxcpus=2` | Limita memória/CPUs (testes) |

## Editando a entrada no momento do boot

No menu do GRUB, selecione a entrada e pressione **`e`**. Na linha que começa com `linux`,
acrescente parâmetros ao final e pressione **`Ctrl+X`** para iniciar. A mudança vale
**só para esse boot** — ótimo para testar antes de tornar permanente.

{{% notice tip "Esqueci a senha de root" %}}
Edite a entrada, acrescente `init=/bin/bash` à linha `linux` e inicie. No shell:

```bash
mount -o remount,rw /
passwd root          # ou passwd <usuário>
sync
exec /sbin/init      # ou reinicie com: reboot -f
```

Isso mostra por que **acesso físico é acesso total** — em servidores, proteja o GRUB com
senha e use criptografia de disco.
{{% /notice %}}

## Reinstalando o GRUB

Se outro sistema sobrescreveu o carregador, inicie por um pendrive *live*, monte o sistema
instalado e reinstale:

```bash
sudo mount /dev/sda2 /mnt                       # partição raiz
sudo mount /dev/sda1 /mnt/boot/efi              # partição EFI (UEFI)
for d in dev proc sys run; do sudo mount --bind /$d /mnt/$d; done
sudo chroot /mnt
grub-install            # UEFI; em BIOS: grub-install /dev/sda
update-grub
exit
```
