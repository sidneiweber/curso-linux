+++
title = "Entrada, saída e redirecionamento"
weight = 5
description = "stdin, stdout, stderr, >, >>, <, <<, | , && e ||."
+++

## Entrada e saída padrão

Todo processo tem três canais de comunicação:

| Canal | Descritor | Padrão |
|---|---|---|
| **stdin** — entrada padrão | `0` | teclado |
| **stdout** — saída padrão | `1` | tela |
| **stderr** — saída de erro | `2` | tela |

```mermaid
graph LR
  T[Teclado] -- "stdin (0)" --> C[Comando]
  C -- "stdout (1)" --> S[Tela]
  C -- "stderr (2)" --> S
```

## Caracteres de redirecionamento

| Operador | Efeito |
|---|---|
| `>` | envia a saída para um arquivo (**sobrescreve**) |
| `>>` | envia a saída para um arquivo (**acrescenta** no final) |
| `<` | usa um arquivo como entrada |
| `<<` | *here document*: entrada digitada até um marcador |
| `\|` | *pipe*: a saída de um comando vira entrada do próximo |

```bash
ls -l /etc > lista.txt          # cria/sobrescreve
date >> lista.txt               # acrescenta
sort < nomes.txt                # lê de arquivo
cat << FIM > mensagem.txt
Olá!
Este texto vai para o arquivo.
FIM
ps aux | grep bash | wc -l      # encadeando comandos
```

## Redirecionando erros

```bash
find / -name passwd 2> erros.txt        # só os erros vão para o arquivo
find / -name passwd 2> /dev/null        # descarta os erros
comando > saida.txt 2>&1                # saída e erros no mesmo arquivo
comando &> tudo.txt                     # forma curta no Bash
```

{{% notice info %}}
`/dev/null` é um "buraco negro": tudo que é enviado para ele é descartado.
{{% /notice %}}

## Encadeando com && e ||

Todo comando retorna um **código de saída**: `0` = sucesso, diferente de `0` = erro
(veja com `echo $?`).

| Operador | Executa o segundo comando… |
|---|---|
| `cmd1 ; cmd2` | sempre |
| `cmd1 && cmd2` | somente se `cmd1` **deu certo** |
| `cmd1 \|\| cmd2` | somente se `cmd1` **falhou** |

```bash
sudo apt update && sudo apt upgrade
mkdir teste || echo "Não foi possível criar o diretório"
cd /pasta/inexistente && ls || echo "falhou"
```

## tee — ver e salvar ao mesmo tempo

```bash
ls -l | tee lista.txt
```
