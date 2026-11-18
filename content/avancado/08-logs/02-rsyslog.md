+++
title = "rsyslog e /var/log"
weight = 2
description = "Facilities, prioridades, regras e envio de logs para um servidor central."
+++

O **rsyslog** implementa o protocolo **syslog**, o padrão histórico de logs no Unix. Ele
grava arquivos de texto em `/var/log` e pode **enviar logs pela rede** para um servidor central.

{{% notice note "Está instalado?" %}}
O Debian 12+ não instala mais o rsyslog por padrão — só o journal. Ubuntu ainda instala.
Para ter os arquivos clássicos: `sudo apt install rsyslog`.
{{% /notice %}}

## Arquivos comuns

| Arquivo | Conteúdo |
|---|---|
| `/var/log/syslog` | Mensagens gerais do sistema |
| `/var/log/auth.log` | Logins, `sudo`, SSH |
| `/var/log/kern.log` | Kernel |
| `/var/log/dpkg.log`, `/var/log/apt/` | Instalação de pacotes |
| `/var/log/nginx/`, `/var/log/apache2/` | Logs próprios das aplicações |
| `/var/log/wtmp`, `/var/log/btmp` | Logins e tentativas falhas (binários: `last`, `lastb`) |

```bash
sudo tail -f /var/log/auth.log
sudo grep 'Failed password' /var/log/auth.log | tail
last -n 10          # últimos logins
sudo lastb | head   # tentativas de login que falharam
```

## Classificação: facility e prioridade

Toda mensagem syslog tem uma **facility** (quem gerou) e uma **prioridade** (gravidade):

| Facilities | Prioridades (da mais grave para a menos) |
|---|---|
| `kern`, `user`, `mail`, `daemon`, `auth`, `authpriv`, `cron`, `local0`–`local7` | `emerg`, `alert`, `crit`, `err`, `warning`, `notice`, `info`, `debug` |

## Regras

Em `/etc/rsyslog.conf` e `/etc/rsyslog.d/*.conf`, cada regra é `seletor   destino`:

```text
auth,authpriv.*          /var/log/auth.log
*.*;auth,authpriv.none   -/var/log/syslog
kern.*                   -/var/log/kern.log
mail.err                 /var/log/mail.err
*.emerg                  :omusrmsg:*
local3.*                 /var/log/minha-app.log
```

- `mail.err` = `err` **e tudo mais grave**; `mail.=err` = só `err`.
- `.none` exclui uma facility.
- `-` antes do arquivo: não força a gravação imediata em disco (melhor desempenho).

```bash
sudo systemctl restart rsyslog
logger -p local3.info "teste" && tail -1 /var/log/minha-app.log
```

## Log centralizado

Enviar os logs para outro servidor protege contra perda (e contra um invasor apagando rastros).

No **servidor** (`/etc/rsyslog.d/10-receber.conf`):

```text
module(load="imtcp")
input(type="imtcp" port="514")
$template PorHost,"/var/log/remoto/%HOSTNAME%/%PROGRAMNAME%.log"
*.* ?PorHost
```

Em cada **cliente** (`/etc/rsyslog.d/90-enviar.conf`):

```text
*.* @@192.168.1.10:514
```

(`@@` = TCP, `@` = UDP.) Reinicie o rsyslog nos dois lados e libere a porta 514 no firewall.
