+++
title = "Auditoria de eventos com auditd"
weight = 3
description = "Registrando acessos a arquivos sensíveis e comandos executados."
+++

Os logs comuns registram o que os **programas** decidem registrar. O **auditd** registra,
direto do kernel, **eventos** que você escolhe: quem leu `/etc/shadow`, quem alterou o
`sudoers`, quais comandos o root executou.

```bash
sudo apt install auditd audispd-plugins
sudo systemctl enable --now auditd
```

## Regras

`/etc/audit/rules.d/curso.rules`:

```text
# alterações em arquivos de identidade e privilégio
-w /etc/passwd -p wa -k identidade
-w /etc/shadow -p rwa -k identidade
-w /etc/group -p wa -k identidade
-w /etc/sudoers -p wa -k privilegio
-w /etc/sudoers.d/ -p wa -k privilegio

# configuração do SSH e chaves autorizadas do root
-w /etc/ssh/sshd_config -p wa -k ssh
-w /root/.ssh/ -p wa -k ssh

# tarefas agendadas (persistência)
-w /etc/crontab -p wa -k cron
-w /etc/cron.d/ -p wa -k cron
-w /etc/systemd/system/ -p wa -k systemd

# todos os comandos executados como root
-a always,exit -F arch=b64 -S execve -F euid=0 -k comandos_root
```

`-w` vigia um caminho; `-p` as operações (`r`ead, `w`rite, `x` execute, `a`ttribute);
`-k` é uma etiqueta para buscar depois.

```bash
sudo augenrules --load
sudo auditctl -l               # regras ativas
```

## Consultando

```bash
sudo ausearch -k identidade -i               # -i traduz UIDs e chamadas para texto
sudo ausearch -k comandos_root --start today -i | less
sudo aureport --summary
sudo aureport -au                            # autenticações
sudo aureport -x --summary                   # executáveis mais usados
```

Cada evento traz o **auid** (*audit UID*): o usuário que **fez login originalmente**, que
não muda com `sudo` ou `su`. Assim, mesmo um comando executado como root aponta quem foi.

{{% notice tip "Volume de logs" %}}
Registrar todos os `execve` gera muito volume em servidores movimentados. Comece pelos
arquivos sensíveis e amplie conforme a necessidade; envie os logs para um servidor central.
{{% /notice %}}

## Exercícios

1. Ative as regras acima, edite `/etc/sudoers.d/` com `visudo` e encontre o evento com `ausearch -k privilegio`.
2. Entre com um usuário comum, use `sudo -i` e rode alguns comandos; confirme que o `auid`
   identifica o usuário original.
