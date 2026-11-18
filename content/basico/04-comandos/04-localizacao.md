+++
title = "Localizando arquivos"
weight = 4
description = "find, locate, which e grep."
+++

## find — busca em tempo real

Sintaxe: `find [onde] [critérios] [ação]`

```bash
find / -name bash              # procura pelo nome exato
find /home -iname "*.PDF"      # ignora maiúsculas/minúsculas
find / -type l                 # apenas links simbólicos
find /etc -type d              # apenas diretórios
find /home -mtime -30          # modificados nos últimos 30 dias
find /home -atime -10          # acessados nos últimos 10 dias
find / -size +100M             # maiores que 100 MB
find ~ -name "*.tmp" -delete   # encontra e apaga
find / -user maria 2>/dev/null # arquivos da maria, ocultando erros
```

## locate — busca rápida em banco de dados

`locate` consulta um índice, por isso é muito mais rápido que o `find` — mas só encontra
arquivos que existiam na última atualização do banco.

```bash
sudo apt install plocate   # se não estiver instalado
locate sshd_config
sudo updatedb              # atualiza o banco de dados
```

## which e whereis — onde está o comando

```bash
which ls       # /usr/bin/ls
whereis ls     # binário, fontes e manuais
```

## grep — procurar texto dentro de arquivos

```bash
grep root /etc/passwd
grep -i erro /var/log/syslog      # ignora maiúsculas/minúsculas
grep -r "Listen" /etc/apache2/    # recursivo
grep -n -v "^#" /etc/fstab        # -n numera, -v inverte (linhas que NÃO começam com #)
```
