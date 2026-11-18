+++
title = "Configuração manual com ip"
weight = 1
description = "Interfaces, endereços e rotas com o comando ip."
+++

O comando **`ip`** (pacote `iproute2`) substituiu `ifconfig`, `route` e `arp`. Tudo o que
ele altera é **temporário** — some ao reiniciar. É ideal para testar e diagnosticar.

## Interfaces

```bash
ip link                         # interfaces e estado (UP/DOWN), MAC, MTU
ip -br link                     # resumido
sudo ip link set enp3s0 up
sudo ip link set enp3s0 down
sudo ip link set enp3s0 mtu 9000
```

Nomes comuns: `lo` (loopback), `enp3s0`/`eno1` (cabo), `wlp2s0` (Wi-Fi), `docker0`/`br0`
(bridges), `wg0` (WireGuard).

## Endereços

```bash
ip addr                         # ou: ip a
ip -br addr                     # resumido
sudo ip addr add 192.168.1.50/24 dev enp3s0
sudo ip addr del 192.168.1.50/24 dev enp3s0
sudo ip addr flush dev enp3s0   # remove todos
```

## Rotas

```bash
ip route                        # tabela de roteamento
ip route get 8.8.8.8            # qual rota/interface seria usada
sudo ip route add default via 192.168.1.1
sudo ip route add 10.20.0.0/16 via 192.168.1.254   # rota estática para outra rede
sudo ip route del 10.20.0.0/16
```

```text
default via 192.168.1.1 dev enp3s0 proto dhcp metric 100
192.168.1.0/24 dev enp3s0 proto kernel scope link src 192.168.1.10
```

A primeira linha é o **gateway padrão**; a segunda diz que a rede local é alcançada
diretamente pela interface.

## Vizinhos (ARP/NDP)

```bash
ip neigh
sudo ip neigh flush dev enp3s0
```

## Equivalências

| Antigo (`net-tools`) | Atual (`iproute2`) |
|---|---|
| `ifconfig` | `ip addr`, `ip link` |
| `ifconfig eth0 up` | `ip link set eth0 up` |
| `route -n` | `ip route` |
| `route add default gw 192.168.1.1` | `ip route add default via 192.168.1.1` |
| `arp -n` | `ip neigh` |
| `netstat -tulpn` | `ss -tulpn` |
