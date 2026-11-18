+++
title = "Compartilhando a Internet com NAT"
weight = 2
description = "ip_forward, masquerade com nftables e teste a partir do cliente."
+++

Objetivo: o `cli01`, que só está na rede interna, acessa a Internet através do `srv01`.

## 1. Habilitar o encaminhamento

Por padrão, o Linux **não** repassa pacotes entre interfaces:

```bash
echo 'net.ipv4.ip_forward = 1' | sudo tee /etc/sysctl.d/90-roteador.conf
sudo sysctl --system
```

## 2. NAT (masquerade) com nftables

O **nftables** é o firewall padrão do Debian (veremos em detalhes na trilha de Segurança).
Edite `/etc/nftables.conf`:

```text
#!/usr/sbin/nft -f
flush ruleset

define LAN = enp0s8
define WAN = enp0s3

table inet filtro {
    chain forward {
        type filter hook forward priority filter; policy drop;
        ct state established,related accept      # respostas
        iifname $LAN oifname $WAN accept         # LAN → Internet
    }
}

table ip nat {
    chain postrouting {
        type nat hook postrouting priority srcnat;
        oifname $WAN masquerade                  # troca o IP de origem pelo do srv01
    }
}
```

```bash
sudo nft -c -f /etc/nftables.conf     # valida a sintaxe
sudo systemctl enable --now nftables
sudo nft list ruleset
```

**Masquerade** é o NAT de origem que usa automaticamente o IP atual da interface de saída —
ideal quando esse IP vem de DHCP.

## 3. Testar no cliente

Enquanto ainda não temos DHCP, configure o `cli01` manualmente:

```bash
sudo ip addr add 192.168.56.20/24 dev enp0s3
sudo ip route add default via 192.168.56.10
echo 'nameserver 1.1.1.1' | sudo tee /etc/resolv.conf
ping -c 3 1.1.1.1
ping -c 3 debian.org
```

No `srv01`, observe a tradução acontecendo:

```bash
sudo tcpdump -ni enp0s8 icmp     # origem 192.168.56.20
sudo tcpdump -ni enp0s3 icmp     # origem 10.0.2.15 (o IP do srv01)
sudo conntrack -L                # tabela de conexões traduzidas (apt install conntrack)
```

## Encaminhamento de portas (DNAT)

Para expor um serviço interno — por exemplo, um site em `192.168.56.20:80` acessível pela
porta 8080 do `srv01`:

```text
table ip nat {
    chain prerouting {
        type nat hook prerouting priority dstnat;
        iifname $WAN tcp dport 8080 dnat to 192.168.56.20:80
    }
    # ... postrouting como antes
}
```

Lembre de permitir esse tráfego na chain `forward`
(`iifname $WAN oifname $LAN ip daddr 192.168.56.20 tcp dport 80 accept`).

## Exercícios

1. Configure o NAT e confirme que o `cli01` navega na Internet.
2. Capture com `tcpdump` nas duas interfaces e compare os endereços de origem.
3. Remova a linha `masquerade`, recarregue e explique por que o `ping` para de funcionar.
