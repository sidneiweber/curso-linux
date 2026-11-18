+++
title = "Por que IPv6"
weight = 1
description = "Esgotamento do IPv4, o que muda e o que melhora."
+++

O IPv4 tem cerca de **4,3 bilhões** de endereços — os blocos livres se esgotaram entre 2011
e 2019. A Internet sobreviveu com **NAT** e **CGNAT**, que fazem muitos clientes
compartilharem um IP público, ao custo de complexidade e de quebrar conexões de entrada.

O IPv6 usa **128 bits**: cerca de 3,4 × 10³⁸ endereços. Na prática, cada rede local recebe
um `/64` — mais endereços do que a Internet IPv4 inteira, ao quadrado.

## O que muda

| | IPv4 | IPv6 |
|---|---|---|
| Tamanho | 32 bits | 128 bits |
| Notação | `192.168.1.10` | `2001:db8:ab:1::10` |
| Configuração automática | DHCP | **SLAAC** (sem servidor) e/ou DHCPv6 |
| NAT | Onipresente | Desnecessário: todo dispositivo tem IP global |
| ARP | ARP | **NDP** (sobre ICMPv6) |
| Broadcast | Sim | Não existe — usa **multicast** |
| Cabeçalho | Variável, com checksum | Fixo (40 bytes), simplificado |
| Fragmentação | Roteadores podem fragmentar | Só a origem (exige ICMPv6 funcionando) |

## Novas funcionalidades

- **Autoconfiguração (SLAAC)**: o roteador anuncia o prefixo; cada host monta seu endereço.
- **Vários endereços por interface**: link-local, global, temporários (privacidade).
- **Fim do NAT**: conexões ponta a ponta voltam a ser simples — e o **firewall** passa a
  ser essencial, pois o dispositivo é diretamente endereçável.
- **IPsec** previsto desde o projeto do protocolo.

## Seu sistema tem IPv6?

```bash
ip -6 addr
ip -6 route
ping -6 -c 3 google.com
curl -6 https://ifconfig.co
```
