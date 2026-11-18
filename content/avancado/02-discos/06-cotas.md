+++
title = "Cotas de disco"
weight = 6
description = "Limitando o espaço por usuário e grupo."
+++

Cotas limitam quanto espaço (e quantos arquivos) cada usuário ou grupo pode usar em um
sistema de arquivos — útil em servidores compartilhados.

- **Limite suave (soft)**: pode ser ultrapassado durante o **período de carência** (7 dias por padrão).
- **Limite rígido (hard)**: nunca pode ser ultrapassado.

## Habilitando no ext4

```bash
sudo apt install quota
```

No `/etc/fstab`, acrescente as opções à partição (ex.: `/home`):

```text
UUID=...  /home  ext4  defaults,usrquota,grpquota  0  2
```

```bash
sudo mount -o remount /home
sudo quotacheck -cugm /home     # cria os arquivos de cota
sudo quotaon -v /home
```

## Definindo limites

```bash
# usuário maria: soft 5 GiB, hard 6 GiB; 0 0 = sem limite de número de arquivos
sudo setquota -u maria 5G 6G 0 0 /home

sudo edquota -u maria        # edição interativa
sudo edquota -t              # período de carência
sudo edquota -p maria joao   # copia os limites de maria para joao
```

## Relatórios

```bash
quota -s                 # o usuário vê a própria cota
sudo repquota -s /home   # relatório de todos
```

{{% notice note "XFS" %}}
No XFS as cotas são ativadas com as opções `uquota`/`gquota` no `fstab` e gerenciadas com
`xfs_quota`, por exemplo: `sudo xfs_quota -x -c 'limit bsoft=5g bhard=6g maria' /home`.
{{% /notice %}}

## Exercícios

1. Em uma VM com um disco extra, crie uma tabela GPT com duas partições e formate uma em
   ext4 e outra em XFS.
2. Monte as duas pelo `/etc/fstab` usando UUID; valide com `findmnt --verify` e reinicie.
3. Crie um VG com dois discos, um LV de 1 GiB, preencha-o com arquivos e aumente-o para
   2 GiB sem desmontar.
4. Configure uma cota de 100 MiB para um usuário de teste e tente ultrapassá-la com
   `fallocate -l 150M`.
