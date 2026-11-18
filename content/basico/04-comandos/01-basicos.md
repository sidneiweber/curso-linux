+++
title = "Comandos básicos"
weight = 1
description = "Navegação e manipulação de arquivos: ls, cd, mkdir, cp, mv, rm, cat, file."
+++

## Dicas antes de começar

- **`Tab`** completa nomes de comandos e arquivos. Aperte duas vezes para ver as opções.
- **`↑` / `↓`** navegam pelo histórico de comandos; `history` mostra o histórico completo.
- **`Ctrl` + `C`** interrompe o comando em execução; **`Ctrl` + `L`** (ou `clear`) limpa a tela.

## Caminhos especiais

| Símbolo | Significado |
|---|---|
| `.` | diretório atual |
| `..` | diretório superior (pai) |
| `~` | diretório pessoal do usuário |
| `-` | diretório anterior (usado com `cd`) |
| `/` | raiz do sistema |

Caminhos **absolutos** começam com `/` (`/home/maria/docs`); caminhos **relativos**
partem do diretório atual (`docs/notas.txt`).

## pwd e cd — onde estou e para onde vou

```bash
pwd             # mostra o diretório atual
cd /etc         # vai para /etc
cd ..           # sobe um nível
cd              # volta para o diretório pessoal (mesmo que cd ~)
cd -            # volta ao diretório anterior
```

## ls — listar

```bash
ls              # lista o diretório atual
ls -l           # formato longo: permissões, dono, tamanho, data
ls -a           # inclui arquivos ocultos
ls -h           # tamanhos legíveis (K, M, G) — use com -l
ls -R           # lista recursivamente os subdiretórios
ls -lah /var/log
```

## mkdir — criar diretórios

```bash
mkdir projetos
mkdir -p curso/linux/aula1   # -p cria os diretórios intermediários
```

## touch — criar arquivo vazio

```bash
touch notas.txt
```

## cp — copiar

```bash
cp notas.txt copia.txt
cp -i notas.txt copia.txt    # -i pergunta antes de sobrescrever
cp -v *.txt backup/          # -v mostra o que está sendo feito
cp -R projetos/ backup/      # -R copia diretórios recursivamente
```

## mv — mover ou renomear

```bash
mv copia.txt antigo.txt      # renomeia
mv antigo.txt backup/        # move para outro diretório
```

## rm — remover

```bash
rm antigo.txt
rm -r backup/        # remove diretório e todo o conteúdo
rm -f arquivo        # força, sem perguntar
rmdir vazio/         # remove apenas diretórios vazios
```

{{% notice warning "Cuidado!" %}}
No terminal **não existe lixeira**. Um `rm -rf` no lugar errado apaga tudo sem volta.
Na dúvida, use `rm -i`.
{{% /notice %}}

## cat e file — ver conteúdo e tipo

```bash
cat /etc/os-release      # exibe o conteúdo do arquivo
cat > lista.txt          # cria arquivo com o que for digitado (Ctrl+D para terminar)
less /var/log/syslog     # visualiza arquivos grandes com rolagem (q para sair)
head -n 5 arquivo        # primeiras 5 linhas
tail -f arquivo          # últimas linhas, acompanhando em tempo real
file foto.jpg            # identifica o tipo do arquivo
```

## Exercício

1. Crie a estrutura `~/curso/aula1/exercicios` com um único comando.
2. Entre em `~/curso/aula1` e crie os arquivos `a.txt` e `b.txt`.
3. Copie `a.txt` para `exercicios/` e renomeie `b.txt` para `c.txt`.
4. Liste tudo recursivamente a partir de `~/curso`.
5. Apague o diretório `~/curso`.

{{% expand "Ver solução" %}}
```bash
mkdir -p ~/curso/aula1/exercicios
cd ~/curso/aula1
touch a.txt b.txt
cp a.txt exercicios/
mv b.txt c.txt
ls -R ~/curso
rm -r ~/curso
```
{{% /expand %}}
