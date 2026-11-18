+++
title = "Servidor FTP com vsftpd"
weight = 3
description = "Instalação, controle de acesso, modo passivo e FTPS."
+++

O **vsftpd** (*very secure FTP daemon*) é o servidor FTP padrão das principais distribuições.

```bash
sudo apt install vsftpd
```

## Configuração

`/etc/vsftpd.conf` (principais opções):

```ini
listen=YES
listen_ipv6=NO

# quem pode entrar
anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022

# prende cada usuário no próprio home
chroot_local_user=YES
allow_writeable_chroot=NO

# só usuários listados
userlist_enable=YES
userlist_file=/etc/vsftpd.userlist
userlist_deny=NO

# modo passivo: faixa de portas a liberar no firewall
pasv_enable=YES
pasv_min_port=40000
pasv_max_port=40100

# FTPS (TLS explícito)
ssl_enable=YES
rsa_cert_file=/etc/ssl/curso/cert.pem
rsa_private_key_file=/etc/ssl/curso/key.pem
force_local_logins_ssl=YES
force_local_data_ssl=YES
ssl_tlsv1_2=YES

xferlog_enable=YES
```

## Usuários

```bash
sudo useradd -m -s /usr/sbin/nologin ftpcliente
sudo passwd ftpcliente
echo ftpcliente | sudo tee -a /etc/vsftpd.userlist

# com chroot e allow_writeable_chroot=NO, o home não pode ser gravável:
sudo chmod a-w /home/ftpcliente
sudo mkdir /home/ftpcliente/arquivos && sudo chown ftpcliente: /home/ftpcliente/arquivos

# shells "nologin" precisam estar em /etc/shells para o PAM do vsftpd aceitar
grep -q nologin /etc/shells || echo /usr/sbin/nologin | sudo tee -a /etc/shells
sudo systemctl restart vsftpd
```

## Controle de acesso

| Arquivo | Função |
|---|---|
| `/etc/vsftpd.userlist` | Com `userlist_deny=NO`, **só** esses usuários entram |
| `/etc/ftpusers` | Usuários **sempre** bloqueados (root, contas de sistema) |

## Testando

```bash
sudo apt install lftp
lftp -u ftpcliente -e 'set ssl:verify-certificate no; ls; bye' srv01
```

No FileZilla, use **"FTP — exigir FTP sobre TLS explícito"**.

## Firewall

Libere a porta 21 **e** a faixa passiva (40000–40100). Com nftables:

```text
tcp dport { 21, 40000-40100 } accept
```

## Exercícios

1. Crie um usuário SFTP preso em `/srv/sftp/<usuário>` e confirme que ele não consegue
   abrir shell nem sair do diretório.
2. Configure o vsftpd com FTPS obrigatório e acesse com o FileZilla ou o `lftp`.
3. Capture com `tcpdump -A port 21` uma sessão FTP **sem** TLS e encontre a senha no tráfego.
