+++
title = "Estrutura de usuários"
weight = 1
description = "UID, root, usuários de sistema e os arquivos /etc/passwd e /etc/shadow."
+++

Cada usuário é identificado internamente por um número, o **UID**:

| Tipo | UID | Exemplo |
|---|---|---|
| **root** (superusuário) | `0` | administrador, pode tudo |
| **Usuários de sistema** | `1`–`999` | `www-data`, `sshd` — executam serviços |
| **Usuários comuns** | `1000+` | pessoas |

```bash
id          # mostra UID, GID e grupos do usuário atual
id maria
```

## /etc/passwd

Contém uma linha por usuário, com campos separados por `:`.

```text
maria:x:1000:1000:Maria Silva,,,:/home/maria:/bin/bash
```

| Campo | Valor |
|---|---|
| 1 — login | `maria` |
| 2 — senha | `x` (a senha fica em `/etc/shadow`) |
| 3 — UID | `1000` |
| 4 — GID (grupo primário) | `1000` |
| 5 — comentário (GECOS) | `Maria Silva` |
| 6 — diretório pessoal | `/home/maria` |
| 7 — shell | `/bin/bash` |

## /etc/shadow

Guarda as **senhas criptografadas** (hash) e as regras de expiração. Só o root consegue lê-lo.

```bash
sudo cat /etc/shadow
```

## sudo

Em vez de logar como root, usuários autorizados executam comandos administrativos com
`sudo`, informando a **própria** senha. Os autorizados pertencem ao grupo `sudo`
(Debian/Ubuntu) ou `wheel` (Fedora/RHEL).
