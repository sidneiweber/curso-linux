+++
title = "Gerenciamento seguro de usuários"
weight = 1
description = "Auditoria de contas, sudo, contas de serviço e expiração de senhas."
+++

## Auditando contas

```bash
awk -F: '$3 == 0 {print $1}' /etc/passwd               # quem tem UID 0 (só root!)
sudo awk -F: '$2 == "" {print $1}' /etc/shadow          # contas sem senha
awk -F: '$7 !~ /(nologin|false)$/ {print $1, $7}' /etc/passwd   # quem tem shell
getent group sudo                                       # quem é administrador
lastlog | grep -v 'Never'                               # último login de cada conta
```

## sudo

- Dê `sudo` a pessoas, **nunca** compartilhe a senha de root.
- Prefira permissões específicas a `ALL`:

```bash
sudo visudo -f /etc/sudoers.d/web
```

```text
# o grupo webadmins só pode gerenciar o Apache
%webadmins ALL=(root) /usr/bin/systemctl reload apache2, /usr/bin/systemctl restart apache2
```

```bash
sudo -l -U joao                   # o que o joao pode fazer
journalctl _COMM=sudo             # tudo o que foi executado com sudo
```

{{% notice warning "Escapes de sudo" %}}
Liberar via `sudo` programas como `vim`, `less`, `find`, `tar` ou `python` equivale a dar
root: todos permitem abrir um shell. Consulte o [GTFOBins](https://gtfobins.github.io/)
antes de liberar um comando.
{{% /notice %}}

## Contas de serviço

Serviços devem rodar com usuários próprios, sem shell e sem senha:

```bash
sudo useradd --system --no-create-home --shell /usr/sbin/nologin meuapp
```

No systemd, `DynamicUser=yes` cria um usuário temporário automaticamente, e opções como
`ProtectSystem=strict`, `ProtectHome=yes` e `NoNewPrivileges=yes` isolam o serviço.
Avalie um serviço com:

```bash
systemd-analyze security nginx.service
```

## Senhas: validade e bloqueio

```bash
sudo chage -l maria                   # política atual
sudo chage -M 365 -W 14 maria         # validade máxima e aviso
sudo chage -d 0 maria                 # obriga a trocar no próximo login
sudo usermod -L joao                  # bloqueia a senha
sudo usermod -e 1 joao                # expira a conta (bloqueia qualquer login)
sudo usermod -s /usr/sbin/nologin joao
```

Desligamento de um funcionário: bloqueie a conta, remova chaves de
`~/.ssh/authorized_keys`, revise o `crontab` dele e troque senhas compartilhadas que ele
conhecia.

As regras de complexidade e bloqueio por tentativas (`pam_pwquality`, `pam_faillock`) estão
no capítulo de PAM do [Linux Avançado]({{% relref "/avancado/07-programas/04-pam" %}}).
