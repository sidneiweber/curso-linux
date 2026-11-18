+++
title = "Endereçamento IPv4"
weight = 2
description = "Endereços, máscaras, CIDR, sub-redes, faixas privadas e NAT."
+++

Um endereço IPv4 tem **32 bits**, escritos em quatro octetos decimais: `192.168.1.10`.
Ele tem duas partes: a **rede** e o **host**. A **máscara** diz onde uma termina e a outra começa.

```text
IP       192.168.1.10   11000000.10101000.00000001.00001010
Máscara  255.255.255.0  11111111.11111111.11111111.00000000
                        └──────── rede (24 bits) ──────┘└host┘
```

## Notação CIDR

Em vez da máscara, indica-se quantos bits são de rede: `192.168.1.10/24`.

| CIDR | Máscara | Endereços | Hosts utilizáveis |
|---|---|---|---|
| `/8` | 255.0.0.0 | 16.777.216 | 16.777.214 |
| `/16` | 255.255.0.0 | 65.536 | 65.534 |
| `/24` | 255.255.255.0 | 256 | 254 |
| `/26` | 255.255.255.192 | 64 | 62 |
| `/28` | 255.255.255.240 | 16 | 14 |
| `/30` | 255.255.255.252 | 4 | 2 |
| `/32` | 255.255.255.255 | 1 | um único host |

Em cada sub-rede, o **primeiro** endereço identifica a rede e o **último** é o *broadcast* —
por isso são 2 a menos utilizáveis.

```bash
sudo apt install ipcalc
ipcalc 192.168.1.10/26
# Network:   192.168.1.0/26
# HostMin:   192.168.1.1
# HostMax:   192.168.1.62
# Broadcast: 192.168.1.63
```

## Dividindo uma rede

A rede `10.0.0.0/24` dividida em 4 sub-redes `/26`:

| Sub-rede | Hosts | Broadcast |
|---|---|---|
| `10.0.0.0/26` | .1 – .62 | .63 |
| `10.0.0.64/26` | .65 – .126 | .127 |
| `10.0.0.128/26` | .129 – .190 | .191 |
| `10.0.0.192/26` | .193 – .254 | .255 |

## Faixas especiais

| Faixa | Uso |
|---|---|
| `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16` | **Privadas** (RFC 1918): redes internas, não roteadas na Internet |
| `127.0.0.0/8` | Loopback (a própria máquina) |
| `169.254.0.0/16` | Link-local: autoconfigurado quando o DHCP falha |
| `100.64.0.0/10` | CGNAT (provedores) |
| `0.0.0.0` | "Qualquer endereço" (em `ss`, escutando em todas as interfaces) |

## Gateway e NAT

Para falar com um host **fora** da sua rede, o pacote vai para o **gateway padrão** (roteador).
Como os IPs privados não circulam na Internet, o roteador faz **NAT**: troca o IP de origem
privado pelo seu IP público e guarda a associação para devolver as respostas.

## Classes (histórico)

Antes do CIDR, as redes eram divididas em **classes**: A (`/8`, 1–126), B (`/16`, 128–191)
e C (`/24`, 192–223). O termo ainda aparece ("uma classe C"), mas não é mais usado no roteamento.
