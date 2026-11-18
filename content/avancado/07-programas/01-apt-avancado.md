+++
title = "APT e dpkg a fundo"
weight = 1
description = "Repositórios, chaves, versões, pinning, hold e diagnóstico."
+++

## Repositórios

Debian 13 e Ubuntu 24.04+ usam o formato **deb822**, em `/etc/apt/sources.list.d/*.sources`:

```text
Types: deb
URIs: https://deb.debian.org/debian
Suites: trixie trixie-updates trixie-backports
Components: main contrib non-free non-free-firmware
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg
```

O formato antigo, de uma linha, ainda funciona em `/etc/apt/sources.list`:

```text
deb https://deb.debian.org/debian trixie main contrib
```

## Adicionando um repositório de terceiros

Exemplo com o repositório oficial do Docker:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo tee /etc/apt/keyrings/docker.asc > /dev/null
sudo tee /etc/apt/sources.list.d/docker.sources <<'EOS'
Types: deb
URIs: https://download.docker.com/linux/debian
Suites: trixie
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOS
sudo apt update
```

{{% notice warning "apt-key" %}}
Tutoriais antigos usam `apt-key add`, que confia na chave para **todos** os repositórios.
Esse comando foi removido; use `Signed-By` apontando para a chave de cada repositório.
{{% /notice %}}

## Versões

```bash
apt policy nginx                 # versão instalada, candidata e de onde vem cada uma
apt list --upgradable
apt show nginx                   # dependências, tamanho, descrição
sudo apt install nginx=1.26.3-3  # versão específica
sudo apt install -t trixie-backports nginx   # versão do backports
```

## Segurando uma versão

```bash
sudo apt-mark hold nginx         # não atualiza
apt-mark showhold
sudo apt-mark unhold nginx
```

Para regras mais elaboradas, use **pinning** em `/etc/apt/preferences.d/`.

## Investigando pacotes

```bash
dpkg -L nginx            # arquivos instalados pelo pacote
dpkg -S /usr/sbin/nginx  # qual pacote instalou este arquivo
apt-file search libssl.so.3   # em qual pacote está (mesmo não instalado) — apt install apt-file
apt-cache rdepends --installed libssl3  # quem depende deste pacote
sudo apt install --reinstall nginx
sudo dpkg --configure -a # termina instalações interrompidas
sudo apt -f install      # conserta dependências quebradas
```

## Logs

```bash
less /var/log/apt/history.log    # o que foi instalado/removido e quando
less /var/log/dpkg.log
```

## E os .rpm?

Distribuições da família Red Hat (Fedora, RHEL, Rocky, Alma) usam `.rpm` com `dnf`:

| Debian/Ubuntu | Fedora/RHEL |
|---|---|
| `apt install pacote` | `dnf install pacote` |
| `apt search termo` | `dnf search termo` |
| `dpkg -L pacote` | `rpm -ql pacote` |
| `dpkg -S /arquivo` | `rpm -qf /arquivo` |
| `dpkg -i arquivo.deb` | `rpm -i arquivo.rpm` / `dnf install ./arquivo.rpm` |
