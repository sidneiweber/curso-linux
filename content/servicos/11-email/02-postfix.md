+++
title = "Postfix"
weight = 2
description = "Instalação, main.cf, entrega em Maildir, aliases e testes."
+++

## Instalação

```bash
sudo apt install postfix mailutils
```

No assistente, escolha **Site da Internet** e informe `curso.lan` como nome de e-mail do
sistema. (Para refazer: `sudo dpkg-reconfigure postfix`.)

## Configuração inicial

O Postfix é configurado em `/etc/postfix/main.cf`. Use o `postconf` para ler e alterar:

```bash
sudo postconf -e 'myhostname = mail.curso.lan'
sudo postconf -e 'mydomain = curso.lan'
sudo postconf -e 'myorigin = $mydomain'
sudo postconf -e 'mydestination = $myhostname, $mydomain, localhost'
sudo postconf -e 'inet_interfaces = all'
sudo postconf -e 'mynetworks = 127.0.0.0/8 [::1]/128 192.168.56.0/24'
sudo postconf -e 'home_mailbox = Maildir/'
sudo postconf -e 'smtpd_banner = $myhostname ESMTP'
sudo systemctl restart postfix
```

| Parâmetro | Significado |
|---|---|
| `myhostname` | Nome do servidor (deve existir no DNS) |
| `mydomain` / `myorigin` | Domínio usado nos endereços de quem envia |
| `mydestination` | Domínios cujas mensagens são entregues **localmente** |
| `mynetworks` | Redes que podem **retransmitir** (*relay*) sem autenticação |
| `home_mailbox = Maildir/` | Entrega em `~/Maildir` |

{{% notice warning "Open relay" %}}
Nunca coloque `0.0.0.0/0` em `mynetworks`: o servidor vira um **relay aberto**, usado por
spammers em poucas horas, e o IP entra em listas negras.
{{% /notice %}}

## DNS

Na zona `curso.lan` (capítulo 4), o MX já aponta para `mail`, que é um apelido de `srv01`.
Confira:

```bash
dig +short curso.lan MX
```

## Testando

```bash
echo "Teste local" | mail -s "Olá" maria
ls ~maria/Maildir/new/

sudo tail -f /var/log/mail.log      # ou: journalctl -u postfix@- -f
mailq                               # fila de mensagens
sudo postqueue -f                   # tenta reenviar a fila
sudo postsuper -d ALL               # apaga a fila (cuidado)
```

Conversando "na mão" com o servidor — ótimo para entender o protocolo:

```bash
nc srv01 25
EHLO cli01.curso.lan
MAIL FROM:<joao@curso.lan>
RCPT TO:<maria@curso.lan>
DATA
Subject: teste manual

Olá, Maria!
.
QUIT
```

## Aliases

`/etc/aliases` redireciona endereços:

```text
postmaster: root
root:       maria
contato:    maria, joao
```

```bash
sudo newaliases
```

## Domínios virtuais

Para receber e-mails de **outros domínios** com usuários que não existem no Linux:

```bash
sudo postconf -e 'virtual_alias_domains = empresa2.lan'
sudo postconf -e 'virtual_alias_maps = hash:/etc/postfix/virtual'
```

`/etc/postfix/virtual`:

```text
contato@empresa2.lan    maria
vendas@empresa2.lan     joao, carla
@empresa2.lan           maria          # catch-all
```

```bash
sudo postmap /etc/postfix/virtual
sudo systemctl reload postfix
```

Com muitos domínios e caixas sem conta no sistema, usa-se `virtual_mailbox_domains` com
entrega via Dovecot (LMTP) e usuários em arquivo, banco de dados ou LDAP.
