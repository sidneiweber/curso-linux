+++
title = "Primeiros scripts"
weight = 7
description = "Variáveis, variáveis de ambiente, $?, $$, test, if, for e while."
+++

Um **shell script** é um arquivo de texto com uma sequência de comandos. Tudo que você digita
no terminal pode ir para um script e ser repetido quantas vezes quiser.

## O primeiro script

```bash
nano ola.sh
```

```bash
#!/bin/bash
# Meu primeiro script
echo "Olá, $USER! Hoje é $(date +%d/%m/%Y)."
```

A primeira linha (`#!/bin/bash`, o *shebang*) diz qual interpretador vai executar o arquivo.
Linhas com `#` são comentários.

```bash
chmod +x ola.sh    # dá permissão de execução (o mesmo que chmod 755)
./ola.sh
```

## Variáveis

```bash
nome="Maria"          # sem espaços em volta do =
echo "Olá, $nome"
hoje=$(date +%F)      # $( ) guarda a saída de um comando
echo "Backup de $hoje"
```

{{% notice tip %}}
Use sempre **aspas duplas** em volta de variáveis (`"$nome"`). Sem elas, um valor com espaços
vira vários argumentos. Aspas **simples** não expandem variáveis: `'$nome'` imprime `$nome`.
{{% /notice %}}

### Variáveis de ambiente

São definidas pelo sistema e herdadas pelos programas:

| Variável | Conteúdo |
|---|---|
| `PATH` | diretórios onde o shell procura os executáveis |
| `HOME` | diretório pessoal do usuário |
| `USER` | nome do usuário logado |
| `SHELL` | shell padrão do usuário |
| `TERM` | tipo de terminal |
| `PWD` | diretório atual |

```bash
echo $PATH
env                         # lista todas as variáveis de ambiente
export EDITOR=nano          # cria uma variável e a exporta para os programas filhos
```

### Variáveis especiais

| Variável | Conteúdo |
|---|---|
| `$?` | código de saída do último comando (**0 = sucesso**, outro valor = erro) |
| `$$` | PID do shell (ou script) atual |
| `$0` | nome do script |
| `$1`, `$2`... | argumentos passados ao script |
| `$#` | quantidade de argumentos |
| `$@` | todos os argumentos |

```bash
ls /etc > /dev/null; echo $?          # 0
ls /naoexiste 2> /dev/null; echo $?   # 2 (erro)
echo $$                               # PID do shell atual
```

## Testes: test e [ ]

O comando `test` (ou sua forma abreviada `[ ]`) avalia uma condição e devolve 0 (verdadeiro)
ou 1 (falso). **Os espaços dentro dos colchetes são obrigatórios.**

| Números | | Textos | |
|---|---|---|---|
| `-eq` | igual | `=` | igual |
| `-ne` | diferente | `!=` | diferente |
| `-gt` | maior | `-z` | texto vazio |
| `-lt` | menor | `-n` | texto não vazio |
| `-ge` | maior ou igual | | |
| `-le` | menor ou igual | | |

| Arquivos | |
|---|---|
| `-e` | existe |
| `-f` | é um arquivo comum |
| `-d` | é um diretório |
| `-s` | existe e não está vazio |
| `-r` / `-w` / `-x` | pode ser lido / alterado / executado |
| `-O` | pertence ao usuário atual |

Para combinar condições: `&&` (e), `||` (ou) e `!` (não).

```bash
[ -d /etc ] && echo "é um diretório"
[ 5 -gt 3 ] && echo "5 é maior que 3"
[ -f /etc/passwd ] && [ -r /etc/passwd ] && echo "existe e pode ser lido"
```

## if — decisões

```bash
#!/bin/bash
if [ $# -eq 0 ]; then
    echo "Uso: $0 arquivo"
    exit 1
fi

if [ -f "$1" ]; then
    echo "$1 tem $(wc -l < "$1") linhas."
elif [ -d "$1" ]; then
    echo "$1 é um diretório."
else
    echo "$1 não existe."
fi
```

## for e while — repetições

```bash
for usuario in maria joao ana; do
    echo "Criando a pasta de $usuario"
    mkdir -p "/tmp/$usuario"
done

for arq in *.log; do
    gzip "$arq"
done

contador=1
while [ $contador -le 5 ]; do
    echo "Volta $contador"
    contador=$((contador + 1))    # $(( )) faz contas
done
```

## Exercício

Crie o script `backup.sh` que:

1. Recebe um diretório como argumento e mostra uma mensagem de uso se nenhum for informado.
2. Sai com erro se o diretório não existir.
3. Cria em `/tmp` um arquivo `backup-NOME-AAAA-MM-DD.tar.gz` com o conteúdo do diretório.
4. Informa se o `tar` terminou com sucesso, usando `$?`.

{{% expand "Ver solução" %}}
```bash
#!/bin/bash
if [ $# -ne 1 ]; then
    echo "Uso: $0 diretorio"
    exit 1
fi

if [ ! -d "$1" ]; then
    echo "Erro: $1 não é um diretório."
    exit 2
fi

destino="/tmp/backup-$(basename "$1")-$(date +%F).tar.gz"
tar czf "$destino" "$1"

if [ $? -eq 0 ]; then
    echo "Backup criado em $destino"
else
    echo "Falha no backup!"
    exit 3
fi
```
{{% /expand %}}
