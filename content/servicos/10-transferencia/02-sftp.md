+++
title = "Servidor SFTP com chroot"
weight = 2
description = "Usuários que só transferem arquivos, presos ao próprio diretório."
+++

O OpenSSH já é um servidor SFTP. O que falta é **restringir** alguns usuários: sem shell e
presos (*chroot*) a um diretório.

## Grupo e diretórios

```bash
sudo groupadd sftpusers
sudo mkdir -p /srv/sftp/fornecedor/upload
sudo useradd -M -d /upload -s /usr/sbin/nologin -G sftpusers fornecedor
sudo passwd fornecedor

# o diretório do chroot precisa ser do root e não pode ser gravável por outros
sudo chown root:root /srv/sftp/fornecedor
sudo chmod 755 /srv/sftp/fornecedor
# a pasta interna é do usuário
sudo chown fornecedor:fornecedor /srv/sftp/fornecedor/upload
```

## Configuração do SSH

`/etc/ssh/sshd_config.d/20-sftp.conf`:

```text
Match Group sftpusers
    ChrootDirectory /srv/sftp/%u
    ForceCommand internal-sftp
    AllowTcpForwarding no
    X11Forwarding no
    PasswordAuthentication yes
```

Se você aplicou `AllowGroups ssh-users` no capítulo de SSH, acrescente o grupo:
`AllowGroups ssh-users sftpusers`.

```bash
sudo sshd -t && sudo systemctl reload ssh
```

## Testando

```bash
sftp fornecedor@srv01
sftp> pwd          # Remote working directory: /upload
sftp> put arquivo.csv
sftp> cd /         # não sai do chroot
sftp> ls /etc      # não existe aqui dentro
```

```bash
ssh fornecedor@srv01      # recusado: "This service allows sftp connections only."
```

{{% notice note "Erro \"bad ownership or modes for chroot directory\"" %}}
Todos os diretórios do caminho do chroot (`/srv`, `/srv/sftp`, `/srv/sftp/fornecedor`)
precisam pertencer ao root e não ser graváveis pelo grupo ou outros.
{{% /notice %}}
