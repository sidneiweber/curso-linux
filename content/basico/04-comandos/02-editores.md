+++
title = "Editores de texto"
weight = 2
description = "Editando arquivos no terminal com nano e vi/vim."
+++

Como no Linux quase toda configuração está em arquivos de texto, saber editá-los no
terminal é fundamental.

## nano — simples e amigável

```bash
nano notas.txt
```

Os atalhos ficam no rodapé (`^` significa `Ctrl`):

| Atalho | Ação |
|---|---|
| `Ctrl` + `O` | salvar |
| `Ctrl` + `X` | sair |
| `Ctrl` + `W` | buscar |
| `Ctrl` + `K` | recortar linha |
| `Ctrl` + `U` | colar |

## vi / vim — poderoso e presente em todo lugar

O **vi** está disponível em praticamente qualquer sistema Unix. Ele trabalha com **modos**:

- **Normal** (ao abrir): comandos e navegação;
- **Inserção**: digitar texto — entre com `i`, saia com `Esc`;
- **Comando**: começa com `:` no modo normal.

| Comando | Ação |
|---|---|
| `i` | entra no modo de inserção |
| `Esc` | volta ao modo normal |
| `:w` | salva |
| `:q` | sai |
| `:wq` ou `ZZ` | salva e sai |
| `:q!` | sai **sem** salvar |
| `dd` | apaga (recorta) a linha |
| `yy` / `p` | copia a linha / cola |
| `u` | desfaz |
| `/texto` | busca "texto" (`n` vai para a próxima ocorrência) |

{{% notice tip %}}
Rode `vimtutor` no terminal para um tutorial interativo de 30 minutos.
{{% /notice %}}
