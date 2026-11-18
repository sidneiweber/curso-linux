+++
title = "Endereçamento IPv6"
weight = 2
description = "Notação, abreviação, prefixos e tipos de endereço."
+++

## Notação

Oito grupos de 4 dígitos hexadecimais, separados por `:`.

```text
2001:0db8:0000:0000:0000:ff00:0042:8329
```

Duas regras de abreviação:

1. Zeros à esquerda em cada grupo podem ser omitidos: `0db8` → `db8`, `0042` → `42`.
2. **Uma** sequência de grupos zerados pode virar `::` (só uma vez por endereço).

```text
2001:db8::ff00:42:8329
```

Em URLs, o endereço vai entre colchetes: `http://[2001:db8::1]:8080/`.

## Prefixos

Como no CIDR: `2001:db8:ab:1::/64`. A estrutura típica:

```text
2001:db8:ab  :  1   :  a1b2:c3ff:fed4:e5f6
└ prefixo do ┘ sub-  └── identificador da interface ──┘
  provedor    rede         (64 bits)
  (/48 ou /56)
```

Um provedor costuma entregar um `/56` ou `/48` ao cliente, que divide em várias redes `/64`.

## Tipos de endereço

| Prefixo | Tipo | Equivalente IPv4 |
|---|---|---|
| `::1/128` | Loopback | `127.0.0.1` |
| `::/128` | Não especificado | `0.0.0.0` |
| `fe80::/10` | **Link-local**: toda interface tem um; só vale no enlace local | `169.254.0.0/16` |
| `2000::/3` | **Global unicast**: endereços públicos | IPs públicos |
| `fc00::/7` (na prática `fd00::/8`) | **ULA**: endereços privados | `10.0.0.0/8` etc. |
| `ff00::/8` | Multicast (`ff02::1` = todos os nós do enlace) | broadcast |
| `2001:db8::/32` | Reservado para documentação | `192.0.2.0/24` |

```bash
ip -6 addr show dev eth0
# inet6 2001:db8:ab:1:a1b2:c3ff:fed4:e5f6/64 scope global dynamic
# inet6 fe80::a1b2:c3ff:fed4:e5f6/64 scope link
```

{{% notice tip "Link-local precisa da interface" %}}
Como todo enlace usa `fe80::/64`, é preciso dizer por qual interface sair:
`ping fe80::1%eth0`.
{{% /notice %}}

## SLAAC na prática

1. A interface cria seu endereço link-local.
2. Pergunta aos roteadores (*Router Solicitation*, via ICMPv6).
3. O roteador responde com o prefixo (*Router Advertisement*).
4. O host gera os 64 bits finais (aleatórios ou estáveis) e verifica se não há duplicata.

```bash
ip -6 route show default     # o gateway padrão aparece como fe80::... do roteador
```
