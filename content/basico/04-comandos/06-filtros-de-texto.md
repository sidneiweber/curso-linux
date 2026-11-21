+++
title = "Filtros de texto"
weight = 6
description = "head, tail, cut, tr, wc, nl, tac, sort, uniq, expand, join, split e xargs."
+++

**Filtros** são comandos que leem texto (de um arquivo ou da entrada padrão), transformam e
escrevem o resultado na saída. Combinados com o `|` (pipe), viram ferramentas poderosas.

Para os exemplos, crie um arquivo de teste:

```bash
cat > tabela.txt
maria;financeiro;3500
joao;ti;4200
ana;ti;5100
pedro;rh;2900
```

Digite as linhas e termine com **`Ctrl` + `D`**.

## head e tail — início e fim

```bash
head tabela.txt            # 10 primeiras linhas (padrão)
head -n 2 tabela.txt       # 2 primeiras linhas
tail tabela.txt            # 10 últimas linhas
tail -n 1 tabela.txt       # última linha
tail -f /var/log/syslog    # acompanha o arquivo em tempo real (Ctrl+C para sair)
```

## tac — de trás para frente

O contrário do `cat`: mostra o arquivo da **última para a primeira** linha.

```bash
tac tabela.txt
```

## nl — numerar linhas

```bash
nl tabela.txt
#      1  maria;financeiro;3500
#      2  joao;ti;4200
#      ...
```

## wc — contar

```bash
wc tabela.txt        # linhas, palavras e bytes
wc -l tabela.txt     # só linhas
wc -w tabela.txt     # só palavras
wc -c tabela.txt     # só bytes (-m conta caracteres)
ls /etc | wc -l      # quantos itens existem em /etc
```

## cut — recortar colunas

`-d` define o delimitador e `-f` os campos:

```bash
cut -d';' -f1 tabela.txt       # só os nomes
cut -d';' -f1,3 tabela.txt     # nome e salário
cut -d: -f1,7 /etc/passwd      # usuário e shell de cada conta
cut -c1-3 tabela.txt           # 3 primeiros caracteres de cada linha
```

## tr — trocar ou apagar caracteres

O `tr` só lê da entrada padrão, por isso usamos `|` ou `<`:

```bash
tr 'a-z' 'A-Z' < tabela.txt    # converte para maiúsculas
tr ';' '\t' < tabela.txt       # troca ; por tabulação
tr -d 'a' < tabela.txt         # apaga todas as letras "a"
echo "muitos     espaços" | tr -s ' '   # -s espreme repetições: "muitos espaços"
```

## sort e uniq — ordenar e remover repetidos

```bash
sort tabela.txt                    # ordem alfabética
sort -t';' -k3 -n tabela.txt       # ordena pela 3ª coluna, numericamente
sort -t';' -k3 -nr tabela.txt      # idem, do maior para o menor
cut -d';' -f2 tabela.txt | sort | uniq -c   # quantas pessoas por setor
```

{{% notice tip %}}
O `uniq` só remove repetições **consecutivas**, por isso quase sempre vem depois de um `sort`.
{{% /notice %}}

## expand — tabulações em espaços

```bash
expand arquivo.txt          # cada tabulação vira 8 espaços
expand -t 4 arquivo.txt     # cada tabulação vira 4 espaços
unexpand -a arquivo.txt     # o caminho inverso
```

## join — juntar arquivos por um campo comum

Funciona como um "JOIN" de banco de dados: une as linhas de dois arquivos que tenham o mesmo
valor em um campo. Os dois arquivos precisam estar **ordenados** por esse campo.

```bash
cat > setores.txt
financeiro;Andar 2
rh;Andar 1
ti;Andar 3
```

```bash
sort -t';' -k2 tabela.txt > func_ordenado.txt
join -t';' -1 2 -2 1 func_ordenado.txt setores.txt
# financeiro;maria;3500;Andar 2
# rh;pedro;2900;Andar 1
# ti;ana;5100;Andar 3
# ti;joao;4200;Andar 3
```

`-1 2` usa o campo 2 do primeiro arquivo; `-2 1` usa o campo 1 do segundo.

## split — dividir arquivos grandes

```bash
split -l 1000 dados.csv parte_      # a cada 1000 linhas: parte_aa, parte_ab, ...
split -b 100M video.mp4 video_      # a cada 100 MB
cat video_* > video.mp4             # junta de novo
```

## xargs — transformar saída em argumentos

Muitos comandos (como `rm` ou `cp`) não leem a entrada padrão; eles esperam **argumentos**.
O `xargs` faz essa ponte, executando o comando com cada item recebido:

```bash
find /tmp -name "*.tmp" | xargs rm                 # apaga todos os .tmp encontrados
find . -name "*.log" -print0 | xargs -0 gzip       # -print0/-0: seguro com espaços no nome
cat servidores.txt | xargs -I{} ping -c1 {}        # {} é substituído por cada linha
```

## Exercício

Usando `/etc/passwd` (campos separados por `:`):

1. Mostre apenas os nomes de usuário, em ordem alfabética e numerados.
2. Conte quantas contas usam o shell `/bin/bash`.
3. Liste os shells existentes e quantas contas usam cada um.
4. Mostre os nomes de usuário em maiúsculas.

{{% expand "Ver solução" %}}
```bash
cut -d: -f1 /etc/passwd | sort | nl
grep -c ':/bin/bash$' /etc/passwd          # ou: grep '/bin/bash$' /etc/passwd | wc -l
cut -d: -f7 /etc/passwd | sort | uniq -c
cut -d: -f1 /etc/passwd | tr 'a-z' 'A-Z'
```
{{% /expand %}}
