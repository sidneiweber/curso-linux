+++
title = "Estrutura de diretórios"
weight = 2
description = "A árvore de diretórios do Linux (FHS)."
+++

No Linux não existem "unidades" como `C:` ou `D:`. Tudo fica em uma **única árvore**
que começa na **raiz** `/`. Essa organização segue o padrão **FHS** (*Filesystem Hierarchy Standard*).

| Diretório | Conteúdo |
|---|---|
| `/` | Diretório raiz — o topo da árvore. |
| `/bin` | Binários (comandos) essenciais dos usuários: `ls`, `cp`, `cat`… |
| `/sbin` | Binários administrativos: `fdisk`, `reboot`, `ip`… |
| `/boot` | Arquivos de inicialização: kernel, initrd, GRUB. |
| `/dev` | Arquivos que representam **dispositivos** (discos, terminais, USB). |
| `/etc` | **Arquivos de configuração** do sistema. |
| `/home` | Diretórios pessoais dos usuários comuns (`/home/maria`). |
| `/root` | Diretório pessoal do usuário **root**. |
| `/lib` | Bibliotecas compartilhadas e módulos do kernel. |
| `/media` | Ponto de montagem automática de mídias removíveis (pendrives, CDs). |
| `/mnt` | Ponto de montagem temporário, usado manualmente. |
| `/opt` | Programas adicionais, fora dos pacotes oficiais. |
| `/tmp` | Arquivos temporários (normalmente apagados ao reiniciar). |
| `/usr` | Programas, bibliotecas e documentação comuns aos usuários. |
| `/var` | Dados variáveis: **logs** (`/var/log`), cache, filas de impressão. |
| `/proc` | Diretório **virtual** gerado pelo kernel com informações do sistema e processos. |
| `/sys` | Diretório virtual com informações de dispositivos e drivers. |

{{% notice tip "Experimente" %}}
```bash
ls /
cat /proc/cpuinfo
ls /home
```
{{% /notice %}}

{{% notice info %}}
Em distribuições modernas, `/bin`, `/sbin` e `/lib` costumam ser apenas *links* para
`/usr/bin`, `/usr/sbin` e `/usr/lib` (*usr merge*).
{{% /notice %}}
