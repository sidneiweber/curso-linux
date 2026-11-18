+++
title = "Criando seus próprios comandos"
weight = 3
description = "Scripts no PATH, aliases e desinstalação de programas compilados."
+++

## Com um script

1. Crie o diretório de scripts pessoais (já faz parte do `PATH` no Debian/Ubuntu):

   ```bash
   mkdir -p ~/.local/bin
   ```

2. Crie o arquivo do comando:

   ```bash
   nano ~/.local/bin/listar
   ```

3. Escreva os comandos, começando pela linha *shebang*:

   ```bash
   #!/bin/bash
   ls -lh --color=auto
   echo 'O comando foi executado com sucesso'
   ```

4. Torne-o executável:

   ```bash
   chmod +x ~/.local/bin/listar
   ```

5. Pronto! Execute de qualquer lugar digitando apenas `listar`.
   (Se não funcionar, faça logout/login ou rode `source ~/.profile`.)

{{% notice tip %}}
Para um comando disponível a **todos os usuários**, coloque o script em
`/usr/local/bin` (com `sudo`). Evite `/bin` e `/usr/bin`: esses diretórios são do
gerenciador de pacotes.
{{% /notice %}}

## Com um alias

Para atalhos simples, um *alias* é suficiente. Adicione ao final do `~/.bashrc`:

```bash
alias ll='ls -lh --color=auto'
alias atualizar='sudo apt update && sudo apt upgrade'
```

Depois rode `source ~/.bashrc`.

## Desinstalar programas compilados (configure / make / make install)

Programas instalados a partir do código-fonte não são controlados pelo `apt`. Para removê-los,
verifique no `Makefile` se existe o alvo `uninstall` e, no diretório do código-fonte, execute:

```bash
sudo make uninstall
```

{{% notice tip %}}
Para facilitar a remoção no futuro, instale com `sudo checkinstall` em vez de
`sudo make install`: ele gera um pacote `.deb` que pode ser removido com `apt remove`.
{{% /notice %}}
