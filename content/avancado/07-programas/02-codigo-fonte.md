+++
title = "Instalando a partir do código-fonte"
weight = 2
description = "configure, make, make install, CMake e onde instalar."
+++

Quando o programa não está nos repositórios (ou você precisa de uma versão/opção específica),
compile-o. Antes, verifique alternativas: **backports**, repositório oficial do projeto,
**Flatpak** (desktop) ou **containers** (servidores).

## Ferramentas

```bash
sudo apt install build-essential pkg-config autoconf automake cmake
```

## O roteiro clássico (autotools)

```bash
wget https://ftp.gnu.org/gnu/hello/hello-2.12.1.tar.gz
tar xf hello-2.12.1.tar.gz && cd hello-2.12.1
./configure --prefix=/usr/local    # verifica dependências e prepara o Makefile
make -j"$(nproc)"                  # compila
sudo make install                  # instala em /usr/local
hello
```

Se o `./configure` reclamar de uma biblioteca, instale o pacote `-dev` correspondente
(ex.: falta `openssl` → `sudo apt install libssl-dev`). O `apt build-dep pacote` instala
tudo o que é necessário para compilar a versão da distribuição.

## CMake e Meson

Projetos mais novos usam outros sistemas de build, mas a ideia é a mesma:

```bash
cmake -B build -DCMAKE_INSTALL_PREFIX=/usr/local && cmake --build build -j && sudo cmake --install build
meson setup build --prefix=/usr/local && ninja -C build && sudo ninja -C build install
```

## Onde instalar

| Prefixo | Uso |
|---|---|
| `/usr` | **Somente** pacotes da distribuição — não instale nada aqui à mão |
| `/usr/local` | Software compilado pelo administrador |
| `/opt/<programa>` | Programas "autocontidos" de terceiros |
| `~/.local` | Só para o seu usuário, sem `sudo` |

## Desinstalando

```bash
sudo make uninstall    # se o projeto oferecer
```

Muitos não oferecem — e o `dpkg` não sabe da existência desses arquivos. Alternativas:
instalar em `/opt/programa` (para remover, basta apagar a pasta) ou usar o **checkinstall**,
que gera um `.deb` a partir do `make install`:

```bash
sudo apt install checkinstall
sudo checkinstall --pkgname=hello --pkgversion=2.12.1
```
