+++
title = "Acessando os compartilhamentos"
weight = 3
description = "Clientes Windows e Linux, montagem com cifs e impressoras."
+++

## Windows

No Explorador de Arquivos, digite na barra de endereço:

```text
\\srv01.curso.lan\publico
\\192.168.56.10\publico
```

Para mapear como unidade de rede, no Prompt de Comando:

```text
net use P: \\srv01.curso.lan\publico /user:CURSO\joao /persistent:yes
net use
```

## Linux: linha de comando

```bash
smbclient -L //srv01 -U joao                 # lista compartilhamentos
smbclient //srv01/publico -U joao            # sessão tipo FTP: ls, get, put
```

## Linux: montando

```bash
sudo apt install cifs-utils
sudo mkdir -p /mnt/publico
sudo mount -t cifs //srv01/publico /mnt/publico -o username=joao,uid=$(id -u),gid=$(id -g)
```

Permanente, com a senha fora do `fstab`:

```bash
sudo tee /root/.smb-joao <<'C'
username=joao
password=SenhaDoJoao
domain=CURSO
C
sudo chmod 600 /root/.smb-joao
```

```text
//srv01/publico  /mnt/publico  cifs  credentials=/root/.smb-joao,uid=1000,gid=1000,_netdev,nofail  0  0
```

Em desktops, basta abrir `smb://srv01/publico` no gerenciador de arquivos.

## Compartilhando impressoras

Com o CUPS instalado e as impressoras configuradas, acrescente ao `smb.conf`:

```ini
[global]
    printing = cups
    printcap name = cups
    load printers = yes

[printers]
    path = /var/spool/samba
    printable = yes
    guest ok = no
    browseable = yes
```

Clientes Windows modernos também podem adicionar a impressora via IPP
(`http://srv01:631/printers/Nome`), dispensando o Samba.

## Segurança Linux × Windows

| Linux | Windows | No Samba |
|---|---|---|
| Dono/grupo/outros (`rwx`) | ACLs NTFS detalhadas | `vfs objects = acl_xattr` armazena ACLs do Windows |
| UID/GID | SID | Mapeamento feito pelo `winbind` em domínios |
| Senhas em `/etc/shadow` | Hash NT | Banco do Samba (`pdbedit`) ou o AD |

## Diagnóstico

```bash
sudo smbstatus                  # quem está conectado e quais arquivos estão abertos
sudo tail -f /var/log/samba/log.*
testparm -s
```

## Exercícios

1. Crie os compartilhamentos `publico` e `financeiro` e os usuários `joao` (funcionarios) e
   `carla` (funcionarios + financeiro). Confirme que só a Carla acessa `financeiro`.
2. Monte `publico` no `cli01` pelo `fstab` com arquivo de credenciais.
3. Se tiver uma VM Windows, mapeie `publico` como unidade `P:`.
