+++
title = "Transição e convivência com IPv4"
weight = 3
description = "Pilha dupla, NAT64/DNS64, túneis e cuidados de segurança."
+++

IPv4 e IPv6 **não** são compatíveis entre si — um host só IPv6 não conversa diretamente
com um só IPv4. Por isso existem mecanismos de transição.

## Pilha dupla (dual stack)

O mais comum: a máquina tem **os dois** protocolos ativos. Quando um nome resolve para os
dois (registros `A` e `AAAA` no DNS), os programas tentam IPv6 primeiro e caem para IPv4
rapidamente se falhar (*Happy Eyeballs*).

```bash
dig +short A google.com
dig +short AAAA google.com
```

## NAT64 / DNS64

Redes **só IPv6** (comuns em operadoras móveis e em nuvens) acessam sites só IPv4 por meio de:

- **DNS64**: inventa um registro `AAAA` (prefixo `64:ff9b::/96` + o IPv4) para nomes sem IPv6;
- **NAT64**: um gateway traduz os pacotes IPv6 desse prefixo para IPv4.

## Túneis

Encapsulam IPv6 dentro de IPv4 (6in4, túneis de *brokers* como o da Hurricane Electric) ou
vice-versa. Hoje são usados principalmente para ter IPv6 onde o provedor ainda não oferece.

## Segurança

{{% notice warning "Firewall para os dois protocolos" %}}
Um erro clássico: configurar regras de firewall só para IPv4 enquanto o servidor tem um
IPv6 global — todas as portas ficam abertas por IPv6. Ferramentas como o **nftables**
(família `inet`), o `ufw` e o `firewalld` tratam os dois juntos; confira com
`ss -tulpn` e testando de fora com `nmap -6`.
{{% /notice %}}

Também filtre com cuidado o **ICMPv6**: ele é essencial (NDP, SLAAC, Path MTU) e não pode
ser bloqueado por inteiro.

## Desativando (se realmente precisar)

```bash
echo 'net.ipv6.conf.all.disable_ipv6 = 1' | sudo tee /etc/sysctl.d/90-sem-ipv6.conf
sudo sysctl --system
```

Prefira corrigir o problema a desativar o IPv6 — ele não vai embora.

## Exercícios

1. Liste os endereços IPv6 da sua máquina e classifique cada um (link-local, global, ULA).
2. Escreva por extenso `2001:db8::1:0:0:1` e abrevie `fd00:0000:0000:0001:0000:0000:0000:00a0`.
3. Verifique se o seu provedor entrega IPv6 com `curl -6 https://ifconfig.co`.
