+++
title = "Regras contra ataques comuns"
weight = 3
description = "Força bruta, varreduras, SYN flood, spoofing e bloqueio por listas."
+++

## Força bruta: limitar novas conexões

Cada IP pode abrir no máximo 4 conexões SSH novas por minuto; quem passar disso vai para
uma lista de bloqueio por 1 hora:

```text
table inet filtro {
    set bloqueados {
        type ipv4_addr
        flags dynamic, timeout
        timeout 1h
    }

    chain input {
        # ... (antes da regra que libera o SSH)
        ip saddr @bloqueados drop
        tcp dport 22 ct state new meter ssh_limite { ip saddr limit rate over 4/minute } \
            add @bloqueados { ip saddr } drop
        tcp dport 22 accept
    }
}
```

```bash
sudo nft list set inet filtro bloqueados
```

O **fail2ban** é complementar: ele lê os logs e bloqueia por **falha de autenticação**,
não só por volume de conexões.

## Varreduras de portas

Uma política `drop` já torna varreduras lentas e pouco informativas. Pacotes TCP com
combinações de flags impossíveis (usados por varreduras "stealth") podem ser descartados:

```text
tcp flags & (fin|syn|rst|ack) == fin|syn drop
tcp flags & (fin|syn|rst|psh|ack|urg) == 0x0 drop      # NULL scan
tcp flags & (fin|psh|urg) == fin|psh|urg drop          # XMAS scan
```

## SYN flood

O ataque abre milhares de conexões pela metade para esgotar a memória do servidor.
A proteção principal está no kernel (já ativada no capítulo 1):

```ini
net.ipv4.tcp_syncookies = 1
```

E, no firewall, um limite de conexões novas por origem:

```text
tcp flags syn ct state new meter syn_limite { ip saddr limit rate over 50/second } drop
```

## IP spoofing

Pacotes chegando pela Internet com origem de rede interna (ou privada) são falsificados:

```text
iifname $WAN ip saddr { 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16, 127.0.0.0/8 } drop
```

Junto com `net.ipv4.conf.all.rp_filter = 1`, que descarta pacotes cuja origem não seria
alcançável pela interface de onde vieram.

{{% notice note "Laboratório" %}}
No VirtualBox, a "Internet" do `srv01` é a rede NAT `10.0.2.0/24` — a regra acima a
bloquearia. Ela é para servidores com IP público na interface externa.
{{% /notice %}}

## Bloqueio por país ou por listas

Sets aceitam milhares de redes com desempenho constante. Listas públicas de IPs maliciosos
(ex.: Spamhaus DROP, FireHOL) podem ser carregadas periodicamente:

```text
set lista_negra { type ipv4_addr; flags interval; }
chain input { ip saddr @lista_negra drop }
```

```bash
curl -s https://www.spamhaus.org/drop/drop.txt | grep -oE '^[0-9./]+' | paste -sd, | \
  xargs -I{} sudo nft add element inet filtro lista_negra '{ {} }'
```

## DDoS volumétrico

Ataques que saturam o **link** não podem ser resolvidos no servidor: o tráfego já ocupou a
banda antes de chegar ao firewall. A proteção é feita antes, no provedor ou em serviços
como Cloudflare.

## Exercícios

1. Aplique o firewall do `srv01` e confirme, com `nmap` a partir do `cli01`, que só as portas
   liberadas aparecem.
2. Implemente o limite de conexões SSH e teste com um laço:
   `for i in $(seq 10); do ssh -o ConnectTimeout=2 srv01 true; done`.
3. Bloqueie o acesso web direto da LAN, obrigando o uso do proxy Squid.
