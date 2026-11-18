+++
title = "Informações em /proc e /sys"
weight = 2
description = "O que o kernel expõe sobre o hardware."
+++

As ferramentas da página anterior, no fundo, leem arquivos de `/proc` e `/sys`.
Conhecê-los ajuda em scripts e em sistemas mínimos, sem essas ferramentas instaladas.

| Arquivo | Conteúdo |
|---|---|
| `/proc/cpuinfo` | Detalhes de cada núcleo da CPU |
| `/proc/meminfo` | Uso detalhado da memória |
| `/proc/partitions` | Discos e partições conhecidos pelo kernel |
| `/proc/mounts` | Sistemas de arquivos montados |
| `/proc/interrupts` | Interrupções (IRQs) por dispositivo e CPU |
| `/proc/loadavg` | Load average |
| `/proc/uptime` | Segundos desde o boot |
| `/sys/class/net/` | Interfaces de rede |
| `/sys/block/` | Dispositivos de bloco (discos) |
| `/sys/class/dmi/id/` | Fabricante, modelo e número de série da máquina |

```bash
grep -c ^processor /proc/cpuinfo           # número de núcleos lógicos
grep MemTotal /proc/meminfo
cat /sys/class/dmi/id/product_name
cat /sys/class/net/*/address               # MAC das interfaces
cat /sys/block/sda/queue/rotational        # 1 = HD, 0 = SSD
```

{{% notice note "Arquivos que não existem" %}}
Nada em `/proc` e `/sys` está no disco: o kernel gera o conteúdo no momento da leitura.
Por isso `ls -l` mostra tamanho 0 — e mesmo assim `cat` mostra o conteúdo.
{{% /notice %}}
