+++
title = "Permissões avançadas"
weight = 3
description = "SUID, SGID, sticky bit e ACLs."
+++

Além de `rwx`, existem três permissões especiais:

| Permissão | Octal | Em arquivos | Em diretórios |
|---|---|---|---|
| **SUID** | `4` | executa com as permissões do **dono** do arquivo | — |
| **SGID** | `2` | executa com as permissões do **grupo** do arquivo | arquivos novos herdam o **grupo** do diretório |
| **Sticky bit** | `1` | — | só o dono de um arquivo pode apagá-lo |

## SUID

O exemplo clássico é o `passwd`: um usuário comum consegue alterar sua senha, que fica em
`/etc/shadow` (pertencente ao root), porque o programa roda como root.

```bash
ls -l /usr/bin/passwd
# -rwsr-xr-x 1 root root ... /usr/bin/passwd   ← o "s" no lugar do x do dono
chmod u+s programa   # ou chmod 4755 programa
```

## SGID em diretórios

Ótimo para pastas compartilhadas por uma equipe:

```bash
sudo mkdir /srv/equipe
sudo chgrp desenvolvimento /srv/equipe
sudo chmod 2770 /srv/equipe     # ou chmod g+s
```

## Sticky bit

O `/tmp` é gravável por todos, mas ninguém pode apagar os arquivos dos outros:

```bash
ls -ld /tmp
# drwxrwxrwt ... /tmp   ← o "t" no final
chmod +t pasta_publica/   # ou chmod 1777
```

{{% notice warning %}}
Use SUID com muito cuidado: um programa SUID root com falhas pode permitir que qualquer
usuário obtenha acesso de administrador.
{{% /notice %}}

## ACLs — permissões para usuários específicos

Quando dono/grupo/outros não basta, as **ACLs** permitem dar permissões a usuários ou
grupos adicionais:

```bash
setfacl -m u:joao:rw relatorio.txt   # joao pode ler e escrever
getfacl relatorio.txt                # mostra as ACLs
setfacl -x u:joao relatorio.txt      # remove a ACL
```
