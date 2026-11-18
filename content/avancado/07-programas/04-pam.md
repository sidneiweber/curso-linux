+++
title = "PAM"
weight = 4
description = "Módulos de autenticação: como funcionam e exemplos práticos."
+++

O **PAM** (*Pluggable Authentication Modules*) é a camada que decide **como** os programas
autenticam usuários. `login`, `sudo`, `su`, `sshd` e o gerenciador de login gráfico não
verificam senhas sozinhos: perguntam ao PAM, que segue as regras configuradas.

Isso permite trocar ou combinar métodos (senha, LDAP, 2FA, biometria) sem alterar os programas.

## Arquivos

Cada serviço tem um arquivo em `/etc/pam.d/`, que normalmente inclui os arquivos comuns
(`common-auth`, `common-account`, `common-password`, `common-session`):

```bash
ls /etc/pam.d/
cat /etc/pam.d/sudo
```

```text
#tipo     controle    módulo                 argumentos
auth      required    pam_env.so
@include  common-auth
account   required    pam_nologin.so
```

## Tipos

| Tipo | Responsável por |
|---|---|
| `auth` | Provar quem é o usuário (senha, token…) |
| `account` | A conta pode entrar agora? (expirada, horário, nologin) |
| `password` | Troca de senha e suas regras |
| `session` | Preparar/limpar a sessão (montar home, limites, logs) |

## Controles

| Controle | Efeito |
|---|---|
| `required` | Precisa passar; se falhar, continua avaliando, mas o resultado final é falha |
| `requisite` | Precisa passar; se falhar, para na hora |
| `sufficient` | Se passar (e nada anterior falhou), aprova sem avaliar o resto |
| `optional` | Só importa se for o único módulo |

## Exemplos práticos

**Senhas fortes** — `pam_pwquality`:

```bash
sudo apt install libpam-pwquality
# /etc/security/pwquality.conf
#   minlen = 12
#   minclass = 3
```

**Bloqueio após tentativas erradas** — `pam_faillock` (configurado em
`/etc/security/faillock.conf`; consulte com `faillock --user maria`).

**Criar o home no primeiro login** (útil com LDAP):

```bash
sudo pam-auth-update --enable mkhomedir
```

**Autenticação em dois fatores no SSH** com `libpam-google-authenticator`.

{{% notice warning "Não se tranque para fora" %}}
Um erro no PAM pode impedir **qualquer** login, inclusive o `sudo`. Antes de editar,
mantenha um terminal aberto como root (`sudo -i`) e teste o login em outro terminal.
No Debian/Ubuntu, prefira `pam-auth-update` a editar os `common-*` à mão.
{{% /notice %}}

## Exercícios

1. Descubra de qual pacote veio o arquivo `/usr/bin/ssh` e quais bibliotecas ele usa.
2. Compile e instale o GNU Hello em `/usr/local` com `checkinstall` e depois remova-o com `apt`.
3. Configure `minlen = 12` no `pwquality` e tente trocar sua senha para uma curta.
