+++
title = "Sistemas de autenticação"
weight = 2
description = "Fatores de autenticação, chaves SSH, 2FA com TOTP e autenticação centralizada."
+++

## Fatores

| Fator | Exemplos |
|---|---|
| Algo que você **sabe** | Senha, PIN |
| Algo que você **tem** | Chave privada, celular (TOTP), chave de segurança FIDO2 |
| Algo que você **é** | Biometria |

**Autenticação multifator** combina fatores de tipos diferentes: roubar a senha não basta.

## Senhas

- Comprimento importa mais que complexidade: frases longas (`cavalo-correto-bateria-grampo`).
- Uma senha por serviço, guardadas em um **gerenciador de senhas** (KeePassXC, Bitwarden).
- No Linux, as senhas ficam em `/etc/shadow` como hash **yescrypt** (`$y$`), legível só pelo root.

```bash
sudo grep maria /etc/shadow | cut -d: -f2 | cut -c1-4    # $y$ = yescrypt
```

## SSH com chave + segundo fator

Chaves SSH já são fortes. Para servidores críticos, exija **chave e** código TOTP:

```bash
sudo apt install libpam-google-authenticator
google-authenticator          # como o usuário: gera o QR code para o app
```

`/etc/pam.d/sshd` — acrescente no início:

```text
auth required pam_google_authenticator.so
```

E comente a linha `@include common-auth` (a senha deixa de ser pedida; o fator "sabe" é a
passphrase da chave).

`/etc/ssh/sshd_config.d/30-2fa.conf`:

```text
KbdInteractiveAuthentication yes
AuthenticationMethods publickey,keyboard-interactive
```

```bash
sudo sshd -t && sudo systemctl reload ssh
```

{{% notice warning "Teste em outra sessão" %}}
Mantenha uma sessão aberta e teste o login em outro terminal antes de desconectar.
{{% /notice %}}

Alternativa ainda mais forte: chaves SSH em **dispositivo FIDO2** (`ssh-keygen -t ed25519-sk`).

## Autenticação centralizada

Em várias máquinas, contas locais se multiplicam e ninguém desativa todas quando alguém
sai. Centralize: **LDAP + SSSD** (capítulo de LDAP de Serviços de Rede), **FreeIPA** ou
**Active Directory**. Desativar a conta em um lugar bloqueia o acesso em todos.
