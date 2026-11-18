+++
title = "Instalação de programas"
weight = 1
description = "Repositórios, apt e lojas gráficas."
+++

## Conceitos

- **Pacote**: arquivo com o programa, seus metadados e scripts de instalação;
- **Repositório**: servidor com pacotes da distribuição (configurado em `/etc/apt/sources.list` e `/etc/apt/sources.list.d/`);
- **Dependências**: outros pacotes de que um programa precisa — o gerenciador as instala automaticamente.

| Família | Formato | Gerenciador |
|---|---|---|
| Debian, Ubuntu, Mint | `.deb` | `apt` / `dpkg` |
| Fedora, RHEL | `.rpm` | `dnf` / `rpm` |
| openSUSE | `.rpm` | `zypper` |
| Arch | `.pkg.tar.zst` | `pacman` |

## Modo texto com apt

```bash
sudo apt update              # atualiza a lista de pacotes disponíveis
sudo apt upgrade             # atualiza os pacotes instalados
sudo apt install htop vlc    # instala um ou mais pacotes
apt search editor            # procura pacotes
apt show htop                # detalhes de um pacote
```

{{% notice tip %}}
Sempre rode `sudo apt update` antes de instalar, para garantir que a lista de pacotes está atualizada.
{{% /notice %}}

## Modo gráfico

As distribuições trazem uma loja de aplicativos: **GNOME Software**, **Discover** (KDE),
**Central de Aplicativos** (Ubuntu), **Gerenciador de Programas** (Mint). Basta procurar e
clicar em **Instalar**.

## Formatos universais

Funcionam em qualquer distribuição e trazem as dependências embutidas:

```bash
flatpak install flathub org.gimp.GIMP   # Flatpak (Flathub)
sudo snap install code --classic        # Snap (Ubuntu)
```

Há também o **AppImage**: um único arquivo executável, basta dar permissão de execução e rodar.
