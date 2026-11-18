+++
title = "Como o e-mail funciona"
weight = 1
description = "MUA, MTA, MDA, SMTP, IMAP/POP3, registros MX e autenticação de domínio."
+++

## As peças

```text
 Thunderbird ──SMTP (587)──▶ Postfix (srv01) ──SMTP (25)──▶ MX do destino ──▶ caixa postal
     (MUA)                    (MTA)                           (MTA)            (MDA/Dovecot)
                                                                                    │
 Thunderbird do destinatário ◀───────────────── IMAP (993) ────────────────────────┘
```

| Sigla | Papel | Exemplos |
|---|---|---|
| **MUA** (*Mail User Agent*) | Programa do usuário | Thunderbird, Outlook, webmail (Roundcube) |
| **MTA** (*Mail Transfer Agent*) | Transporta mensagens entre servidores (SMTP) | **Postfix**, Exim, Sendmail |
| **MDA** (*Mail Delivery Agent*) | Entrega na caixa postal | **Dovecot** (LMTP), procmail |
| **IMAP/POP3** | Leitura das mensagens | **Dovecot** |

## Protocolos e portas

| Porta | Uso |
|---|---|
| 25 | SMTP entre servidores |
| 587 | *Submission*: clientes enviando, com autenticação e STARTTLS |
| 465 | *Submission* com TLS implícito |
| 993 | IMAP com TLS (as pastas ficam no servidor, sincronizadas entre dispositivos) |
| 995 | POP3 com TLS (baixa e, em geral, apaga do servidor) — pouco usado hoje |

## Como o servidor de destino é encontrado

Pelo registro **MX** do domínio do destinatário:

```bash
dig +short gmail.com MX
```

## Formatos de caixa postal

- **mbox**: todas as mensagens em um único arquivo por pasta.
- **Maildir**: um arquivo por mensagem — mais robusto. **Use este.**

## Reputação: SPF, DKIM e DMARC

Para que as mensagens de um servidor **público** não caiam no spam (ou sejam recusadas), o
domínio precisa de:

| Registro | Função | Exemplo (DNS, registro TXT) |
|---|---|---|
| **SPF** | Quais servidores podem enviar pelo domínio | `v=spf1 mx -all` |
| **DKIM** | Assinatura criptográfica das mensagens | `default._domainkey` com a chave pública |
| **DMARC** | O que fazer quando SPF/DKIM falham, e para onde enviar relatórios | `_dmarc`: `v=DMARC1; p=quarantine; rua=mailto:dmarc@exemplo.com.br` |

Também é preciso **DNS reverso** (PTR) do IP do servidor apontando para o seu nome.

{{% notice warning "Servidor de e-mail público na prática" %}}
Manter um servidor de e-mail na Internet é trabalhoso: muitos provedores bloqueiam a
porta 25 de saída em conexões residenciais e de nuvem, e a reputação do IP é decisiva.
Para produção, considere soluções integradas (**Mailcow**, **Mailu**, **iRedMail**) ou um
provedor. No laboratório, vamos montar o servidor só para a rede interna.
{{% /notice %}}
