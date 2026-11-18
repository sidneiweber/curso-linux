+++
title = "Dovecot e envio autenticado"
weight = 3
description = "IMAP, SASL para o Postfix, porta 587 e TLS."
+++

## Instalação

```bash
sudo apt install dovecot-imapd dovecot-lmtpd
```

## Configuração

Os arquivos ficam em `/etc/dovecot/conf.d/`. Ajustes principais:

`10-mail.conf`:

```text
mail_location = maildir:~/Maildir
```

`10-auth.conf`:

```text
disable_plaintext_auth = yes
auth_mechanisms = plain login
```

`10-ssl.conf`:

```text
ssl = required
ssl_cert = </etc/ssl/curso/cert.pem
ssl_key = </etc/ssl/curso/key.pem
```

`10-master.conf` — expõe a autenticação para o Postfix:

```text
service auth {
  unix_listener /var/spool/postfix/private/auth {
    mode = 0660
    user = postfix
    group = postfix
  }
}
```

```bash
sudo doveconf -n                 # configuração efetiva
sudo systemctl restart dovecot
```

## Envio autenticado (porta 587)

Usuários fora de `mynetworks` precisam se autenticar para enviar. Em
`/etc/postfix/master.cf`, descomente e ajuste o serviço `submission`:

```text
submission inet n       -       y       -       -       smtpd
  -o syslog_name=postfix/submission
  -o smtpd_tls_security_level=encrypt
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_sasl_type=dovecot
  -o smtpd_sasl_path=private/auth
  -o smtpd_client_restrictions=permit_sasl_authenticated,reject
```

E o TLS do Postfix em `main.cf`:

```bash
sudo postconf -e 'smtpd_tls_cert_file = /etc/ssl/curso/cert.pem'
sudo postconf -e 'smtpd_tls_key_file = /etc/ssl/curso/key.pem'
sudo postconf -e 'smtpd_tls_security_level = may'
sudo systemctl restart postfix
```

## Configurando o cliente

No Thunderbird do `cli01`:

| | Servidor | Porta | Segurança | Usuário |
|---|---|---|---|---|
| Entrada (IMAP) | `mail.curso.lan` | 993 | SSL/TLS | `maria` |
| Saída (SMTP) | `mail.curso.lan` | 587 | STARTTLS | `maria` |

## Testando pela linha de comando

```bash
openssl s_client -connect mail.curso.lan:993 -quiet
a LOGIN maria SenhaDaMaria
b LIST "" "*"
c SELECT INBOX
d LOGOUT

swaks --to maria@curso.lan --server mail.curso.lan --port 587 --tls --auth-user joao
```

(`sudo apt install swaks` — o "canivete suíço" de testes SMTP.)
