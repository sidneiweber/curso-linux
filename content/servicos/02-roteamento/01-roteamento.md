+++
title = "Como o roteamento funciona"
weight = 1
description = "Tabela de rotas, a regra do prefixo mais específico, rotas estáticas."
+++

Para cada pacote que sai, o kernel consulta a **tabela de roteamento** e escolhe a rota
**mais específica** (o maior prefixo) que contém o destino.

```bash
ip route
```

```text
default via 10.0.2.2 dev enp0s3 proto dhcp metric 100
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15
192.168.56.0/24 dev enp0s8 proto kernel scope link src 192.168.56.10
```

| Destino | Rota escolhida | Por quê |
|---|---|---|
| `192.168.56.20` | `192.168.56.0/24` direto pela `enp0s8` | `/24` é mais específico que `/0` |
| `8.8.8.8` | `default via 10.0.2.2` | Só a rota padrão (`0.0.0.0/0`) contém esse endereço |

```bash
ip route get 192.168.56.20
ip route get 8.8.8.8
```

## Rotas estáticas

Se existir outra rede, `10.50.0.0/16`, alcançável por um roteador em `192.168.56.1`:

```bash
sudo ip route add 10.50.0.0/16 via 192.168.56.1     # temporário
```

Permanente, em `/etc/network/interfaces`, dentro do bloco da interface:

```text
    up   ip route add 10.50.0.0/16 via 192.168.56.1
    down ip route del 10.50.0.0/16 via 192.168.56.1
```

No Netplan, use a lista `routes:`; no NetworkManager,
`nmcli con mod <conexão> +ipv4.routes "10.50.0.0/16 192.168.56.1"`.

## Métrica

Quando há duas rotas para o mesmo destino com o mesmo prefixo, vence a de **menor métrica**
— útil para ter um link principal e um de reserva.

## Roteamento dinâmico

Em redes grandes, manter rotas estáticas à mão é inviável. Roteadores trocam rotas
automaticamente com protocolos de roteamento:

| Protocolo | Tipo | Uso |
|---|---|---|
| **RIP** | Vetor de distância (contagem de saltos, máx. 15) | Histórico, redes pequenas — pouco usado hoje |
| **OSPF** | Estado de enlace | Redes corporativas |
| **BGP** | Vetor de caminho | Entre provedores (a Internet), datacenters, Kubernetes |

No Linux, esses protocolos são implementados pelo **FRRouting** (`sudo apt install frr`),
configurado com uma linguagem semelhante à dos roteadores Cisco (`vtysh`).
