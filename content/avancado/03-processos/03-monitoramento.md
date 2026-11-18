+++
title = "Monitoramento e diagnóstico"
weight = 3
description = "load average, vmstat, iostat, lsof, ss e strace."
+++

## Load average

```bash
uptime
# 10:42:01 up 3 days,  2 users,  load average: 0.52, 0.71, 0.80
```

São as médias de **1, 5 e 15 minutos** do número de processos rodando ou esperando
(estados `R` e `D`). Compare com o número de CPUs (`nproc`): load 4 em uma máquina com
4 núcleos significa ocupação total; acima disso, há fila.

## htop, btop e atop

```bash
sudo apt install htop btop
htop       # F6 ordena, F5 árvore, F9 envia sinal, F4 filtra
btop       # CPU, memória, disco e rede em uma tela
```

## vmstat e iostat

```bash
vmstat 2 5       # a cada 2s, 5 vezes: processos, memória, swap, I/O, CPU
```

Colunas para observar: `r` (fila de CPU), `si`/`so` (swap entrando/saindo — se for
constante, falta RAM) e `wa` (CPU esperando disco).

```bash
sudo apt install sysstat
iostat -xz 2     # uso de cada disco; %util perto de 100 = disco saturado
pidstat 2        # CPU por processo ao longo do tempo
```

## Memória

```bash
free -h
```

A coluna que importa é **available**. O Linux usa RAM "sobrando" como cache de disco
(`buff/cache`), e libera esse cache quando os programas precisam — memória "livre" baixa
**não** é um problema.

{{% notice note "OOM killer" %}}
Quando a memória realmente acaba, o kernel escolhe um processo e o encerra (o *OOM killer*).
Para ver se isso aconteceu: `journalctl -k | grep -i 'out of memory'`.
{{% /notice %}}

## Arquivos e conexões abertos

```bash
sudo lsof -p 1234           # tudo o que o processo 1234 tem aberto
sudo lsof /var/log/syslog   # quem está usando este arquivo
sudo lsof +L1               # arquivos apagados que ainda ocupam espaço
sudo ss -tulpn              # portas TCP/UDP em escuta e quem as abriu
```

{{% notice tip "Disco cheio, mas du não encontra nada?" %}}
Um arquivo apagado continua ocupando espaço enquanto algum processo o mantiver aberto
(comum com logs). `sudo lsof +L1` mostra quem é; reinicie esse processo para liberar o espaço.
{{% /notice %}}

## strace: o que o processo está fazendo?

```bash
sudo apt install strace
strace -f -e trace=openat ls /tmp      # quais arquivos o ls abre
sudo strace -p 1234                    # acompanha um processo travado
```

## Exercícios

1. Rode `yes > /dev/null &` três vezes e observe o load average subir; depois use `renice`
   em um deles e veja a diferença no `top`.
2. Limite um `stress --cpu 2` a 30% de CPU com `systemd-run` e confirme no `systemd-cgtop`.
3. Abra um arquivo grande com `less`, apague-o em outro terminal e encontre-o com `lsof +L1`.
