+++
title = "Firewall do servidor"
weight = 2
description = "Política de negar por padrão, liberando só os serviços necessários."
+++

O `srv01` do laboratório é roteador e servidor ao mesmo tempo. Vamos escrever o firewall
completo em `/etc/nftables.conf`.

```text
#!/usr/sbin/nft -f
flush ruleset

define WAN = enp0s3
define LAN = enp0s8
define REDE_LAN = 192.168.56.0/24

table inet filtro {

    set admins {
        type ipv4_addr
        elements = { 192.168.56.20 }
    }

    chain input {
        type filter hook input priority filter; policy drop;

        ct state established,related accept
        ct state invalid drop
        iif lo accept

        # ICMP/ICMPv6 essenciais (ping limitado)
        ip protocol icmp icmp type echo-request limit rate 5/second accept
        ip protocol icmp icmp type echo-request drop          # excesso de ping
        ip protocol icmp accept
        meta l4proto ipv6-icmp accept

        # SSH só dos administradores
        tcp dport 22 ip saddr @admins accept

        # serviços para a rede interna
        iifname $LAN udp dport { 53, 67 } accept
        iifname $LAN tcp dport { 53, 80, 443, 445, 2049, 3128, 389, 587, 993 } accept

        # registra o resto antes de descartar (limitado, para não inundar o log)
        limit rate 10/minute log prefix "nft-input-drop: " level info
    }

    chain forward {
        type filter hook forward priority filter; policy drop;

        ct state established,related accept
        ct state invalid drop

        # LAN só sai para a Internet por DNS e pelo proxy; acesso web direto bloqueado
        iifname $LAN oifname $WAN udp dport 123 accept      # NTP
        iifname $LAN oifname $WAN ip saddr @admins accept   # admins saem direto
    }

    chain output {
        type filter hook output priority filter; policy accept;
    }
}

table ip nat {
    chain postrouting {
        type nat hook postrouting priority srcnat;
        oifname $WAN masquerade
    }
}
```

```bash
sudo nft -c -f /etc/nftables.conf      # verifica sem aplicar
sudo nft -f /etc/nftables.conf
sudo systemctl enable nftables
sudo nft list ruleset
```

{{% notice warning "Não se tranque para fora" %}}
Ao mexer no firewall de uma máquina remota, agende um "desfazer" antes de aplicar:

```bash
echo 'nft flush ruleset' | sudo at now + 5 minutes
sudo nft -f /etc/nftables.conf
# deu certo? cancele: atq; sudo atrm <número>
```
{{% /notice %}}

## Manipulando em tempo real

```bash
sudo nft list chain inet filtro input -a           # mostra os "handles" das regras
sudo nft add element inet filtro admins '{ 192.168.56.30 }'
sudo nft insert rule inet filtro input tcp dport 8080 accept
sudo nft delete rule inet filtro input handle 17
journalctl -k | grep nft-input-drop                # pacotes descartados
```

Alterações com `nft add/insert/delete` são **temporárias**: leve-as para o
`/etc/nftables.conf`.

## Alternativas mais simples

| Ferramenta | Para quem |
|---|---|
| **ufw** (Ubuntu) | Servidores simples: `ufw allow 22/tcp`, `ufw enable` |
| **firewalld** (Fedora/RHEL) | Zonas e serviços; `firewall-cmd --add-service=http --permanent` |

As duas geram regras nftables por baixo. Não misture ferramentas na mesma máquina.
