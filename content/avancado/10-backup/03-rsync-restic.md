+++
title = "rsync e restic"
weight = 3
description = "Sincronização eficiente e backups versionados, deduplicados e criptografados."
+++

## rsync

Copia só as **diferenças**, localmente ou via SSH:

```bash
rsync -aHAX --info=progress2 /home/ /mnt/backup/home/
rsync -aHAX --delete /home/ backup@servidor:/backups/home/
rsync -aHAXn --delete /home/ /mnt/backup/home/     # -n: simula (dry run)
```

| Opção | Significado |
|---|---|
| `-a` | Modo arquivo: recursivo, preserva permissões, datas, donos, links |
| `-H -A -X` | Preserva hard links, ACLs e atributos estendidos |
| `--delete` | Apaga no destino o que não existe mais na origem (espelho) |
| `--exclude` | Ignora arquivos (`--exclude='.cache/'`) |
| `-z` | Compacta durante a transferência (útil em links lentos) |

{{% notice note "A barra no final importa" %}}
`rsync -a /home/ destino/` copia o **conteúdo** de `/home`; `rsync -a /home destino/`
cria `destino/home`.
{{% /notice %}}

Um espelho com `--delete` **não** é um backup com histórico: se você apagar algo e o rsync
rodar, a cópia também some. Para histórico, use snapshots (`--link-dest`) ou o restic.

## restic

Backups **versionados**, **deduplicados** e **criptografados**, para disco local, servidor
SSH ou nuvem (S3, Backblaze B2, etc.):

```bash
sudo apt install restic
export RESTIC_REPOSITORY=/mnt/backup/restic     # ou sftp:backup@servidor:/restic, s3:...
export RESTIC_PASSWORD_FILE=/root/.restic-senha

restic init                                         # uma vez
restic backup /etc /home --exclude-caches --exclude='/home/*/.cache'
restic snapshots                                    # lista as versões
restic forget --keep-daily 7 --keep-weekly 4 --keep-monthly 12 --prune
restic check                                        # verifica a integridade
```

{{% notice warning "Guarde a senha" %}}
Sem a senha do repositório, **ninguém** recupera os dados — nem você. Guarde-a em um
gerenciador de senhas, fora do servidor.
{{% /notice %}}

## Bancos de dados

Copiar os arquivos de um banco em funcionamento gera backups inconsistentes. Faça um
**dump** antes do backup:

```bash
pg_dump -Fc minha_base > /var/backups/minha_base.dump          # PostgreSQL
mysqldump --single-transaction minha_base | zstd > /var/backups/minha_base.sql.zst  # MySQL/MariaDB
```

## Juntando tudo

`/usr/local/bin/backup.sh`, agendado por um timer do systemd (capítulo anterior):

```bash
#!/usr/bin/env bash
set -euo pipefail
export RESTIC_REPOSITORY=sftp:backup@servidor:/restic
export RESTIC_PASSWORD_FILE=/root/.restic-senha

dpkg --get-selections > /var/backups/pacotes.txt
pg_dump -Fc minha_base > /var/backups/minha_base.dump

restic backup /etc /home /srv /var/backups --exclude-caches
restic forget --keep-daily 7 --keep-weekly 4 --keep-monthly 12 --prune
```
