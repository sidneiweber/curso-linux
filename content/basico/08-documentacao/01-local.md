+++
title = "Documentação local"
weight = 1
description = "man, --help, info, apropos e /usr/share/doc."
+++

## --help

A maioria dos comandos mostra um resumo das opções:

```bash
ls --help
cp --help | less
```

## man — as páginas de manual

```bash
man ls
man 5 passwd     # seção 5: formato do arquivo /etc/passwd
```

Navegação: `↑`/`↓` ou `Espaço` para rolar, `/texto` para buscar, `q` para sair.

| Seção | Conteúdo |
|---|---|
| 1 | comandos de usuário |
| 5 | formatos de arquivos de configuração |
| 8 | comandos administrativos |

{{% notice tip %}}
Para ter as páginas em português: `sudo apt install manpages-pt-br`.
{{% /notice %}}

## Procurando o comando certo

```bash
apropos "copy files"   # busca nas descrições dos manuais (mesmo que man -k)
whatis ls              # descrição em uma linha
type cd                # diz se é comando interno do shell, alias ou programa
help cd                # ajuda dos comandos internos do Bash
```

## Outros recursos locais

```bash
info coreutils         # documentação no formato info (GNU)
ls /usr/share/doc/     # documentação instalada com os pacotes
```
