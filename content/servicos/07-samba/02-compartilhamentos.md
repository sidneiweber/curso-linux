+++
title = "Servidor de arquivos"
weight = 2
description = "Instalação, smb.conf, usuários do Samba e permissões."
+++

## Instalação

```bash
sudo apt install samba smbclient
```

## smb.conf

Guarde o original e crie um enxuto em `/etc/samba/smb.conf`:

```ini
[global]
    workgroup = CURSO
    server string = Servidor de arquivos srv01
    server role = standalone server
    interfaces = lo enp0s8
    bind interfaces only = yes
    server min protocol = SMB3
    map to guest = never
    log file = /var/log/samba/%m.log
    max log size = 1000

[publico]
    comment = Pasta pública (leitura para todos, escrita para o grupo)
    path = /srv/samba/publico
    read only = no
    valid users = @funcionarios
    write list = @funcionarios
    create mask = 0664
    directory mask = 2775
    force group = funcionarios

[financeiro]
    comment = Somente o setor financeiro
    path = /srv/samba/financeiro
    read only = no
    valid users = @financeiro
    create mask = 0660
    directory mask = 2770
    force group = financeiro
    browseable = no

[homes]
    comment = Pasta pessoal
    browseable = no
    read only = no
    valid users = %S
```

| Opção | Significado |
|---|---|
| `path` | Diretório compartilhado |
| `read only = no` | Permite escrita (equivalente a `writable = yes`) |
| `valid users` | Quem pode acessar (`@grupo` = membros do grupo) |
| `write list` | Quem pode escrever, mesmo que o share seja somente leitura |
| `create mask` / `directory mask` | Permissões de novos arquivos/diretórios |
| `force group` | Arquivos criados pertencem a este grupo |
| `browseable = no` | Não aparece na lista, mas acessível digitando o caminho |
| `[homes]` | Share especial: cada usuário vê a própria pasta pessoal |

## Diretórios e grupos

```bash
sudo groupadd funcionarios
sudo groupadd financeiro
sudo mkdir -p /srv/samba/{publico,financeiro}
sudo chgrp funcionarios /srv/samba/publico && sudo chmod 2775 /srv/samba/publico
sudo chgrp financeiro /srv/samba/financeiro && sudo chmod 2770 /srv/samba/financeiro
```

O bit **setgid** (`2` em `2775`) faz os arquivos novos herdarem o grupo do diretório.

{{% notice note "Duas camadas de permissão" %}}
O acesso final é a **interseção** das regras do Samba (`valid users`, `read only`) com as
**permissões do Linux** no diretório. Se uma das duas negar, o acesso é negado.
{{% /notice %}}

## Usuários

O Samba precisa de um usuário Linux **e** de uma senha própria do Samba:

```bash
sudo adduser --disabled-password --gecos "" joao   # usuário Linux sem senha de login
sudo usermod -aG funcionarios joao
sudo smbpasswd -a joao                              # senha do Samba
sudo pdbedit -L                                     # lista usuários do Samba
sudo smbpasswd -d joao                              # desativa
```

## Aplicando

```bash
testparm                      # valida e mostra a configuração efetiva
sudo systemctl restart smbd
sudo systemctl enable smbd
```
