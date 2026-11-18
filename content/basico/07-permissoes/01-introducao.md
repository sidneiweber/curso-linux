+++
title = "Introdução às permissões"
weight = 1
description = "Lendo a saída de ls -l: tipo, permissões, dono e grupo."
+++

```bash
$ ls -l
-rw-r--r-- 1 maria equipe 1200 nov 17 10:00 relatorio.txt
drwxr-x--- 2 maria equipe 4096 nov 17 10:05 projetos
```

Vamos decompor `-rw-r--r--`:

| Posição | `-` | `rw-` | `r--` | `r--` |
|---|---|---|---|---|
| Significado | tipo | **dono** (u) | **grupo** (g) | **outros** (o) |

**Tipo**: `-` arquivo comum, `d` diretório, `l` link simbólico, `b`/`c` dispositivos.

## O que cada permissão significa

| Permissão | Em arquivos | Em diretórios |
|---|---|---|
| **r** (read) | ler o conteúdo | listar o conteúdo (`ls`) |
| **w** (write) | alterar o conteúdo | criar, renomear e apagar arquivos dentro dele |
| **x** (execute) | executar como programa | entrar no diretório (`cd`) |

## Dono e grupo

Na linha acima, o dono é `maria` e o grupo é `equipe`. Quando um usuário tenta acessar
o arquivo, o sistema verifica, **nessa ordem**: ele é o dono? Pertence ao grupo? Senão,
vale a permissão de "outros".

{{% notice info %}}
O usuário **root** ignora as permissões e pode acessar qualquer arquivo.
{{% /notice %}}
