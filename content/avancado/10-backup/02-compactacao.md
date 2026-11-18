+++
title = "Compactação e tar"
weight = 2
description = "gzip, xz, zstd e tar, incluindo backups incrementais."
+++

## Compactadores

| Ferramenta | Extensão | Características |
|---|---|---|
| `gzip` | `.gz` | Universal, rápido, compressão média |
| `bzip2` | `.bz2` | Mais lento, pouco usado hoje |
| `xz` | `.xz` | Melhor compressão, lento para compactar |
| `zstd` | `.zst` | Ótimo equilíbrio: muito rápido e boa compressão — **recomendado** |
| `zip` | `.zip` | Compatível com Windows |

```bash
zstd -T0 arquivo.sql          # -T0 usa todos os núcleos
zstd -d arquivo.sql.zst
gzip -k arquivo.log           # -k mantém o original
xz -9 -T0 imagem.raw
```

## tar

O **tar** junta vários arquivos em um só (preservando permissões, donos e links) e pode
compactar ao mesmo tempo:

```bash
tar -cavf etc-$(date +%F).tar.zst /etc     # -a escolhe o compactador pela extensão
tar -tvf etc-2026-11-18.tar.zst            # lista o conteúdo
tar -xvf etc-2026-11-18.tar.zst -C /tmp/restaura          # extrai em outro lugar
tar -xvf etc-2026-11-18.tar.zst -C /tmp/restaura etc/ssh  # só um diretório
```

| Opção | Significado |
|---|---|
| `c` / `x` / `t` | Criar / extrair / listar |
| `f arquivo` | Nome do arquivo tar |
| `v` | Mostra os arquivos |
| `z` / `J` / `--zstd` / `a` | gzip / xz / zstd / automático pela extensão |
| `-C dir` | Muda para o diretório antes de agir |
| `--exclude=padrão` | Ignora arquivos |
| `-p` | Preserva permissões ao extrair (padrão para root) |

## Backup incremental com tar

```bash
# domingo: completo (o arquivo .snar guarda o "estado" dos arquivos)
tar --listed-incremental=/backup/home.snar -cazf /backup/home-full.tar.gz /home

# demais dias: só o que mudou desde o último
tar --listed-incremental=/backup/home.snar -cazf /backup/home-$(date +%F).tar.gz /home
```

Para restaurar, extraia o completo e depois **cada** incremental, em ordem, com
`--listed-incremental=/dev/null`.
