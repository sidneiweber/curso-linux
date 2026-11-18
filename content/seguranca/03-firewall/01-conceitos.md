+++
title = "Conceitos"
weight = 1
description = "Netfilter, tabelas, chains, hooks, regras, estado das conexões e iptables."
+++

## Caminho de um pacote

```text
                      ┌──────────────┐
  entra ─▶ prerouting ─▶ decisão de    ─▶ forward ─▶ postrouting ─▶ sai
                      │ roteamento   │                 ▲
                      └──────┬───────┘                 │
                             ▼                         │
                           input                    output
                             ▼                         ▲
                       processo local ─────────────────┘
```

| Hook | Pacotes |
|---|---|
| `prerouting` | Todos os que chegam, antes de decidir o destino (DNAT) |
| `input` | Destinados **à própria máquina** |
| `forward` | **Atravessando** a máquina (roteador) |
| `output` | Gerados pela própria máquina |
| `postrouting` | Todos os que saem (SNAT/masquerade) |

## Estrutura do nftables

- **Tabela** (*table*): agrupa chains; tem uma **família** — `ip` (IPv4), `ip6`, `inet`
  (IPv4 **e** IPv6 juntos), `arp`, `bridge`.
- **Chain**: lista de regras ligada a um **hook**, com **tipo** (`filter`, `nat`), prioridade
  e **política** padrão (`accept` ou `drop`).
- **Regra**: condição + ação (`accept`, `drop`, `reject`, `jump`, `log`, `counter`…).
- **Sets** e **maps**: listas de IPs/portas usadas nas regras, eficientes mesmo com milhares
  de itens.

## Estado das conexões (conntrack)

O Netfilter acompanha as conexões, o que permite regras **com estado**:

| Estado | Significado |
|---|---|
| `new` | Primeiro pacote de uma conexão |
| `established` | Pacote de uma conexão já aceita |
| `related` | Relacionado a uma conexão existente (ex.: ICMP de erro, FTP de dados) |
| `invalid` | Não pertence a nenhuma conexão conhecida — descarte |

A regra mais importante de qualquer firewall: **aceitar o que é `established,related`** e
decidir apenas sobre conexões **novas**.

## drop × reject

- `drop`: descarta em silêncio — o outro lado espera até o tempo esgotar.
- `reject`: responde com recusa — mais "educado" e rápido para diagnosticar.

Na Internet, `drop` é o mais usado; na rede interna, `reject` facilita a vida.

## E o iptables?

O iptables foi o padrão por 20 anos e aparece em muitos tutoriais. No Debian atual, o
comando `iptables` é o **iptables-nft**: aceita a sintaxe antiga, mas grava regras no
nftables. Equivalências:

| iptables | nftables |
|---|---|
| `iptables -L -n -v` | `nft list ruleset` |
| `-A INPUT -p tcp --dport 22 -j ACCEPT` | `add rule inet filtro input tcp dport 22 accept` |
| `-m state --state ESTABLISHED,RELATED` | `ct state established,related` |
| `-j MASQUERADE` | `masquerade` |
| `ip6tables` separado | família `inet` cobre os dois |
| `iptables-save` | `nft list ruleset > arquivo` |

Para converter regras antigas: `iptables-restore-translate -f regras.v4`.
