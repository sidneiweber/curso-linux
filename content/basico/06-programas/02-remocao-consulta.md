+++
title = "Remoção e consulta"
weight = 2
description = "Removendo pacotes e consultando o que está instalado."
+++

## Removendo

```bash
sudo apt remove vlc       # remove o programa, mantém configurações
sudo apt purge vlc        # remove programa e arquivos de configuração
sudo apt autoremove       # remove dependências que não são mais necessárias
sudo apt clean            # limpa o cache de pacotes baixados
```

## Consultando

```bash
apt list --installed              # todos os pacotes instalados
apt list --installed | grep vim
apt list --upgradable             # pacotes com atualização disponível
dpkg -l | grep firefox            # consulta via dpkg
dpkg -L htop                      # arquivos instalados por um pacote
dpkg -S /usr/bin/htop             # a qual pacote pertence um arquivo
```
