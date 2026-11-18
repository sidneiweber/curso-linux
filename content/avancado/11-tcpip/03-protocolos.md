+++
title = "Protocolos: IP, ICMP, ARP, TCP e UDP"
weight = 3
description = "O que cada protocolo faz e como observá-lo."
+++

## IP

Entrega pacotes de um endereço a outro, **sem garantia**: pacotes podem se perder, chegar
duplicados ou fora de ordem. Cada pacote tem um **TTL**, decrementado a cada roteador —
quando chega a zero, é descartado (evita loops infinitos).

## ICMP

Mensagens de controle e erro do IP: *destino inalcançável*, *TTL excedido*, *echo* (o `ping`).
O `traceroute` explora o TTL: envia pacotes com TTL 1, 2, 3… e cada roteador que descarta
um deles se revela com um ICMP *TTL excedido*.

{{% notice warning "Não bloqueie todo ICMP" %}}
Bloquear o ICMP inteiro no firewall quebra o *Path MTU Discovery* e causa conexões que
"travam" misteriosamente. Bloqueie, no máximo, o *echo request* vindo de fora.
{{% /notice %}}

## ARP

Na rede local, os quadros Ethernet usam endereços **MAC**. O ARP descobre o MAC de um IP
("quem tem 192.168.1.1? Responda para 192.168.1.10"):

```bash
ip neigh          # tabela ARP (vizinhos conhecidos)
```

## TCP e UDP

| | TCP | UDP |
|---|---|---|
| Conexão | Estabelecida antes (*three-way handshake*) | Não há; cada datagrama é independente |
| Confiabilidade | Garante entrega, ordem e controle de fluxo | Sem garantias |
| Overhead | Maior | Mínimo |
| Usos | HTTP, SSH, SMTP, bancos de dados | DNS, DHCP, VoIP, jogos, streaming, QUIC/HTTP3 |

O handshake do TCP:

```text
cliente  ── SYN ──────────▶  servidor
cliente  ◀─ SYN+ACK ──────   servidor
cliente  ── ACK ──────────▶  servidor   (conexão ESTABLISHED)
```

## Vendo os pacotes

```bash
sudo apt install tcpdump
sudo tcpdump -i any -n icmp                 # ping
sudo tcpdump -i any -n port 53              # consultas DNS
sudo tcpdump -i any -n 'tcp[tcpflags] & tcp-syn != 0'   # inícios de conexão TCP
sudo tcpdump -i eth0 -w captura.pcap        # grava para abrir no Wireshark
```
