+++
title = "Antispam e antivírus"
weight = 4
description = "Restrições do Postfix, Rspamd, ClamAV e DKIM."
+++

## Restrições básicas do Postfix

Muito spam é barrado só com regras de protocolo, antes mesmo de receber a mensagem:

```bash
sudo postconf -e 'smtpd_helo_required = yes'
sudo postconf -e 'disable_vrfy_command = yes'
sudo postconf -e 'smtpd_recipient_restrictions = permit_mynetworks, permit_sasl_authenticated, reject_unauth_destination, reject_unknown_recipient_domain, reject_rbl_client zen.spamhaus.org'
```

- `reject_unauth_destination`: **essencial** — impede o relay aberto.
- `reject_rbl_client`: consulta listas negras públicas de IPs de spam.

## Rspamd

Os materiais clássicos usam **Amavis + SpamAssassin + ClamAV**. A solução atual mais usada é
o **Rspamd**: mais rápido, com interface web, e que também **assina com DKIM**. Ele se
integra ao Postfix como *milter*:

```bash
sudo apt install rspamd redis-server
```

`/etc/postfix/main.cf`:

```bash
sudo postconf -e 'smtpd_milters = inet:localhost:11332'
sudo postconf -e 'non_smtpd_milters = inet:localhost:11332'
sudo postconf -e 'milter_default_action = accept'
sudo systemctl restart postfix rspamd
```

```bash
rspamc stat
rspamc < mensagem.eml           # analisa uma mensagem e mostra a pontuação
sudo rspamadm configwizard      # assistente (senha da interface web, DKIM)
```

A interface web fica em `http://localhost:11334` (acesse por túnel SSH).

## Antivírus: ClamAV

```bash
sudo apt install clamav clamav-daemon
sudo systemctl enable --now clamav-freshclam clamav-daemon
```

Ative o módulo antivírus do Rspamd em `/etc/rspamd/local.d/antivirus.conf`:

```text
clamav {
  action = "reject";
  type = "clamav";
  servers = "/run/clamav/clamd.ctl";
  symbol = "CLAM_VIRUS";
}
```

```bash
sudo systemctl restart rspamd
# teste com o arquivo EICAR (inofensivo, reconhecido por todos os antivírus)
```

## DKIM

```bash
sudo rspamadm dkim_keygen -d curso.lan -s default -k /var/lib/rspamd/dkim/curso.lan.default.key
```

O comando mostra o registro TXT a publicar no DNS (`default._domainkey.curso.lan`).

## Exercícios

1. Configure Postfix + Dovecot e troque mensagens entre `maria` e `joao` pelo Thunderbird.
2. Crie os aliases `contato` e `postmaster` e teste.
3. Tente usar o servidor como relay a partir de um IP fora de `mynetworks` sem autenticação
   e confirme a recusa (`Relay access denied`).
4. Instale o Rspamd e envie uma mensagem com o texto de teste **GTUBE** para vê-la marcada como spam.
