+++
title = "Multiusuário"
weight = 3
description = "Vários usuários usando o sistema ao mesmo tempo."
+++

O Linux é **multiusuário**: diversos usuários podem estar logados **ao mesmo tempo**,
seja em consoles diferentes, pelo ambiente gráfico ou remotamente via **SSH**.

Cada usuário tem seu próprio diretório pessoal, arquivos, permissões e processos.

## Quem está logado?

```bash
whoami   # qual é o meu usuário
who      # quem está logado
w        # quem está logado e o que está fazendo
```

{{% notice tip "Experimente" %}}
Abra um console virtual com `Ctrl` + `Alt` + `F3`, faça login com seu usuário e,
de volta ao terminal gráfico, execute `who`. Você aparecerá duas vezes!
{{% /notice %}}
