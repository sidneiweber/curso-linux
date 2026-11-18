+++
title = "Prioridades"
weight = 2
description = "nice, renice, ionice e limites com systemd."
+++

## nice e renice (CPU)

O **nice** vai de **-20** (mais prioridade) a **19** (menos prioridade); o padrão é 0.
Quanto mais "gentil" (*nice*), mais o processo cede a CPU aos outros.

```bash
nice -n 10 tar czf backup.tar.gz /home   # inicia com prioridade baixa
renice -n 15 -p 1234                     # muda um processo em execução
sudo renice -n -5 -p 1234                # aumentar prioridade exige root
ps -o pid,ni,cmd -p 1234                 # coluna NI
```

No `top`, a tecla **`r`** muda o nice de um processo.

## ionice (disco)

Um backup pode deixar o sistema lento mesmo com nice baixo, porque disputa o **disco**:

```bash
ionice -c 3 rsync -a /home /backup       # classe idle: só usa o disco quando ninguém mais usa
ionice -c 2 -n 7 -p 1234                 # best-effort, prioridade mais baixa (0–7)
```

## Limites com systemd

Para serviços, a forma moderna é limitar recursos na própria unit (via *cgroups*):

```bash
sudo systemctl set-property meuapp.service CPUQuota=50% MemoryMax=512M
```

Ou rodar um comando avulso já limitado:

```bash
sudo systemd-run --scope -p CPUQuota=25% -p MemoryMax=1G ./script-pesado.sh
```

```bash
systemd-cgtop          # consumo por serviço/grupo, em tempo real
```

## Limites por usuário: ulimit

```bash
ulimit -a              # limites do shell atual
ulimit -n              # máximo de arquivos abertos
ulimit -n 4096         # aumenta (até o limite rígido)
```

Limites permanentes por usuário ficam em `/etc/security/limits.conf` (sessões de login) ou,
para serviços, em `LimitNOFILE=` na unit.
