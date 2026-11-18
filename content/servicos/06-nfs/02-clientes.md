+++
title = "Clientes NFS"
weight = 2
description = "Montagem manual, fstab e montagem sob demanda com systemd."
+++

```bash
sudo apt install nfs-common
showmount -e srv01            # lista exportações (NFSv3; em v4 puro pode não responder)
```

## Montagem manual

```bash
sudo mkdir -p /mnt/publico
sudo mount -t nfs4 srv01:/srv/nfs/publico /mnt/publico
df -hT /mnt/publico
touch /mnt/publico/teste-$(hostname)
```

## Montagem no boot

`/etc/fstab`:

```text
srv01:/srv/nfs/publico  /mnt/publico  nfs4  defaults,_netdev,nofail  0  0
```

`_netdev` avisa que o sistema depende da rede; `nofail` evita travar o boot se o servidor
estiver fora do ar.

## Montagem sob demanda

Montagens NFS permanentes travam processos (e às vezes o desligamento) quando o servidor cai.
Com **automount** do systemd, a pasta só é montada quando alguém a acessa, e é desmontada
após um tempo ocioso:

```text
srv01:/srv/nfs/publico  /mnt/publico  nfs4  noauto,x-systemd.automount,x-systemd.idle-timeout=10min,_netdev  0  0
```

```bash
sudo systemctl daemon-reload
sudo systemctl restart remote-fs.target
ls /mnt/publico      # monta neste momento
```

(Para mapas mais elaborados, como montar o home de cada usuário, existe o `autofs`.)

## Diagnóstico

```bash
nfsstat -m                    # montagens e opções efetivas no cliente
sudo nfsstat -s               # estatísticas no servidor
rpcinfo -p srv01              # serviços RPC (NFSv3)
```

| Sintoma | Causa provável |
|---|---|
| `access denied by server` | IP do cliente fora da regra em `/etc/exports`, ou faltou `exportfs -ra` |
| `Permission denied` ao gravar | Permissões/UID no servidor; `root_squash` para o root |
| Arquivos com dono "estranho" | UIDs diferentes entre cliente e servidor |
| Comandos travam na pasta | Servidor inacessível; use automount |

## Exercícios

1. Exporte `/srv/nfs/publico` para a rede e monte-o no `cli01` pelo `fstab`.
2. Como root no `cli01`, crie um arquivo na pasta e verifique, no servidor, quem é o dono.
3. Crie o usuário `ana` com UIDs diferentes no servidor e no cliente e observe o problema.
