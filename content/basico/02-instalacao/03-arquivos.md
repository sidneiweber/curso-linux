+++
title = "Arquivos no Linux"
weight = 3
description = "Nomes, extensões, arquivos ocultos e noções de permissões."
+++

## Nomes de arquivos

- Podem ter até **255 caracteres**;
- São **case sensitive**: `Relatorio.txt`, `relatorio.txt` e `RELATORIO.TXT` são três arquivos diferentes;
- Evite espaços e acentos — prefira `meu_arquivo.txt` ou `meu-arquivo.txt`;
- O único caractere proibido é a `/` (separador de diretórios).

## Extensões

São de **uso opcional**. O tipo de um arquivo é determinado pelo seu **conteúdo**, e não
pela extensão. Use o comando `file` para descobrir o tipo:

```bash
file /etc/passwd
# /etc/passwd: ASCII text
file /bin/ls
# /bin/ls: ELF 64-bit LSB pie executable, x86-64 ...
```

## Arquivos ocultos

Arquivos e diretórios cujo nome **começa com ponto (`.`)** são ocultos. Normalmente
guardam configurações pessoais, como `~/.bashrc`. Para vê-los:

```bash
ls -a
```

## Permissões (introdução)

Cada arquivo ou diretório possui permissões de **leitura (r)**, **escrita (w)** e
**execução (x)**, aplicadas a três categorias:

- **Proprietário** (*user*) do arquivo;
- **Grupo** ao qual o arquivo pertence;
- **Outros** usuários do sistema.

Veremos isso em detalhes no [capítulo de permissões]({{% relref "/basico/07-permissoes" %}}).
