+++
title = "Permissões avançadas"
weight = 3
description = "SUID, SGID, sticky bit, ACLs e atributos (chattr)."
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

## Atributos de arquivos: chattr e lsattr

Os **atributos** valem nos sistemas de arquivos ext2/3/4 (e, em parte, em XFS e Btrfs) e
ficam acima das permissões: **nem o root** consegue apagar um arquivo imutável sem antes
remover o atributo.

| Atributo | Efeito |
|---|---|
| `i` | **imutável**: não pode ser alterado, apagado, renomeado nem receber links |
| `a` | **somente acréscimo** (*append*): só aceita gravação no final; ótimo para logs |
| `A` | não atualiza a data de último acesso (*atime*) |
| `d` | ignorado pelo backup com `dump` |
| `c` | compactação automática (só em alguns sistemas de arquivos, como Btrfs) |
| `S` | gravações síncronas, direto no disco |

```bash
sudo chattr +i /etc/resolv.conf    # torna imutável
lsattr /etc/resolv.conf            # ----i---------e------- /etc/resolv.conf
sudo rm /etc/resolv.conf           # rm: Operation not permitted
sudo chattr -i /etc/resolv.conf    # remove o atributo

sudo chattr +a /var/log/app.log    # só permite acrescentar
sudo chattr =A arquivo.txt         # = define exatamente estes atributos
lsattr -d /srv/equipe              # -d mostra o próprio diretório, não o conteúdo
```

{{% notice note %}}
Os atributos `s` (apagamento seguro) e `u` (recuperação), citados em materiais antigos, **não
são implementados** pelo ext4 atual. Para apagar um arquivo com segurança, use `shred -u arquivo`.
{{% /notice %}}

{{% notice tip %}}
Se um arquivo "não apaga de jeito nenhum", mesmo como root, confira com `lsattr`.
Atacantes às vezes usam `chattr +i` para proteger arquivos maliciosos.
{{% /notice %}}
