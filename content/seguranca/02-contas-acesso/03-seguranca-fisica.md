+++
title = "Segurança física"
weight = 3
description = "Firmware, senha no GRUB, criptografia de disco com LUKS."
+++

**Acesso físico é acesso total.** Com a máquina nas mãos, alguém pode iniciar por um
pendrive e ler o disco, ou editar o GRUB e entrar como root (como vimos no capítulo de
GRUB do Linux Avançado).

## Firmware (BIOS/UEFI)

- Senha de administrador no firmware.
- Boot apenas pelo disco interno (desative USB/rede na ordem de boot).
- **Secure Boot** ativo: só carregadores e kernels assinados.

## Senha no GRUB

Impede editar entradas e passar parâmetros como `init=/bin/bash`:

```bash
grub-mkpasswd-pbkdf2         # gera o hash
```

`/etc/grub.d/40_custom`:

```text
set superusers="admin"
password_pbkdf2 admin grub.pbkdf2.sha512.10000.ABC123...
```

Para que o boot **normal** continue sem senha, acrescente `--unrestricted` às entradas:
em `/etc/grub.d/10_linux`, altere `CLASS="--class gnu-linux --class gnu --class os"` para
incluir `--unrestricted`.

```bash
sudo update-grub
```

## Criptografia de disco: LUKS

Sem criptografia, firmware e GRUB protegidos não impedem alguém de **retirar o disco**.
Com **LUKS**, os dados ficam ilegíveis sem a senha (ou chave).

- **Na instalação**: o instalador do Debian/Ubuntu oferece "LVM criptografado" — é a forma
  mais simples para notebooks.
- **Em um disco de dados**:

```bash
sudo apt install cryptsetup
sudo cryptsetup luksFormat /dev/sdb1              # APAGA o conteúdo
sudo cryptsetup open /dev/sdb1 dados              # cria /dev/mapper/dados
sudo mkfs.ext4 /dev/mapper/dados
sudo mount /dev/mapper/dados /mnt/dados
sudo umount /mnt/dados && sudo cryptsetup close dados
sudo cryptsetup luksDump /dev/sdb1                # slots de chave
sudo cryptsetup luksAddKey /dev/sdb1              # senha extra (recuperação)
```

Montagem no boot: `/etc/crypttab` (`dados UUID=... none luks`) + `/etc/fstab` com
`/dev/mapper/dados`.

{{% notice warning "Guarde a senha de recuperação" %}}
Se a senha do LUKS for perdida, os dados estão perdidos. Faça backup do cabeçalho
(`cryptsetup luksHeaderBackup`) e guarde uma senha de recuperação em local seguro.
{{% /notice %}}

## Exercícios

1. Liste as contas com shell válido e os membros do grupo `sudo` do `srv01`; justifique cada um.
2. Crie uma regra de `sudo` que permita a um usuário apenas reiniciar o Apache.
3. Proteja o GRUB com senha mantendo o boot normal sem senha, e tente editar uma entrada.
4. Crie um volume LUKS em um disco extra da VM e configure a montagem pelo `crypttab`.
