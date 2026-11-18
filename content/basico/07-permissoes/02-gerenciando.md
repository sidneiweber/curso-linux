+++
title = "Gerenciando permissões"
weight = 2
description = "chmod (simbólico e octal), chown, chgrp e umask."
+++

## chmod — modo simbólico

`chmod [quem][operação][permissões] arquivo`

- **quem**: `u` (dono), `g` (grupo), `o` (outros), `a` (todos)
- **operação**: `+` adiciona, `-` remove, `=` define exatamente

```bash
chmod u+x script.sh          # dono pode executar
chmod g-w relatorio.txt      # grupo perde escrita
chmod o= segredo.txt         # outros perdem tudo
chmod a+r publico.txt        # todos podem ler
chmod u=rwx,g=rx,o= pasta/
```

## chmod — modo octal

Cada permissão tem um valor: **r = 4**, **w = 2**, **x = 1**. Some os valores para cada categoria:

| Octal | Permissão |
|---|---|
| 7 | `rwx` |
| 6 | `rw-` |
| 5 | `r-x` |
| 4 | `r--` |
| 0 | `---` |

```bash
chmod 644 relatorio.txt   # rw-r--r--  (padrão para arquivos)
chmod 755 script.sh       # rwxr-xr-x  (programas e diretórios)
chmod 700 privado/        # rwx------  (só o dono)
chmod -R 750 projetos/    # -R aplica recursivamente
```

## chown e chgrp — mudando dono e grupo

```bash
sudo chown joao arquivo.txt            # muda o dono
sudo chown joao:equipe arquivo.txt     # muda dono e grupo
sudo chgrp equipe arquivo.txt          # muda só o grupo
sudo chown -R maria:maria /srv/site    # recursivo
```

## umask — permissões padrão

A `umask` define quais permissões são **retiradas** dos arquivos novos.

```bash
umask        # normalmente 0022 → arquivos 644, diretórios 755
```

## Exercício

Crie um script `ola.sh` com o conteúdo `echo "Olá, Linux!"`, torne-o executável apenas
para o dono e execute-o.

{{% expand "Ver solução" %}}
```bash
echo 'echo "Olá, Linux!"' > ola.sh
chmod 700 ola.sh     # ou chmod u+x ola.sh
./ola.sh
```
{{% /expand %}}
