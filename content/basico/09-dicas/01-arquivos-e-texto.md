+++
title = "Arquivos e texto"
weight = 1
description = "split, rm --, tee, wc, sort, find -exec, wget, convert e mais."
+++

## Dividir um arquivo em várias partes

Útil para enviar arquivos grandes em pedaços. O `split` divide e o `cat` junta de novo:

```bash
split -b 100M video.mp4 video_parte_     # gera video_parte_aa, video_parte_ab, ...
cat video_parte_* > video.mp4            # reagrupa
```

Também é possível dividir por número de linhas: `split -l 1000 dados.csv dados_`.

## Apagar arquivos que começam com `-`

Um arquivo chamado `-audio2012.wav` confunde o `rm`, que acha que é uma opção.
Use `--` para indicar o fim das opções (ou um caminho com `./`):

```bash
rm -- -audio2012.wav
rm ./-audio2012.wav
```

## Ver a saída na tela e salvar em arquivo ao mesmo tempo

```bash
ls | tee conteudo.txt
ls | tee -a conteudo.txt     # -a acrescenta em vez de sobrescrever
```

## Contar linhas, palavras e caracteres

```bash
wc arquivo.txt       # linhas, palavras, bytes
wc -l arquivo.txt    # só linhas
```

## Ordenar o conteúdo de um arquivo

```bash
sort agenda.txt
sort -r agenda.txt          # ordem inversa
sort -n numeros.txt         # ordenação numérica
sort agenda.txt | uniq      # remove linhas repetidas
```

## Listar arquivos ordenados por data

```bash
ls -ltr      # mais recentes por último
ls -lt       # mais recentes primeiro
ls -lS       # ordenado por tamanho
```

## Pesquisar palavras em um arquivo / tipo de arquivo

```bash
grep o_que_procuro arquivo.txt
file nome-do-arquivo
```

## Executar uma ação em todos os arquivos encontrados

Exemplo: apagar todos os arquivos objeto (`*.o`) nos diretórios dos alunos:

```bash
find /home/universidade/alunos -name "*.o" -type f -delete
# forma clássica, que funciona com qualquer comando:
find /home/universidade/alunos -name "*.o" -type f -exec rm -f {} \;
```

{{% notice warning %}}
Antes de apagar, rode o mesmo `find` **sem** o `-delete`/`-exec` para conferir a lista.
{{% /notice %}}

## Baixar arquivos e sites

```bash
wget https://exemplo.com/arquivo.iso
wget -c https://exemplo.com/arquivo.iso      # -c continua um download interrompido
wget -r -np -k https://exemplo.com/docs/     # cópia para leitura offline
```

## Converter imagens

Com o **ImageMagick** (`sudo apt install imagemagick`):

```bash
convert imagem.png imagem.jpg            # ImageMagick 6 (Debian/Ubuntu)
magick imagem.png imagem.jpg             # ImageMagick 7
convert foto.jpg -resize 50% menor.jpg
convert -list format                     # formatos suportados
```

## Arquivos com `~` no final

Arquivos como `relatorio.txt~` são **cópias de backup** criadas automaticamente por
alguns editores de texto. Podem ser apagados com segurança.
