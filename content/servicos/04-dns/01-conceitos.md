+++
title = "Conceitos de DNS"
weight = 1
description = "Hierarquia, resolução recursiva e iterativa, zonas, delegação e tipos de servidor."
+++

## Hierarquia

O DNS é uma base de dados distribuída em árvore. Cada nível é responsável apenas pelo nível
abaixo:

```text
                       . (raiz)
            ┌──────────┼──────────┐
           br         com        org
            │          │
         com.br     google
            │          │
     sidneiweber     www
```

`www.sidneiweber.com.br.` é lido da direita para a esquerda: raiz → `br` → `com.br` →
`sidneiweber.com.br` → `www`. O ponto final representa a raiz.

## Como uma consulta é resolvida

```text
cliente ──▶ resolvedor (recursivo) ──▶ servidores raiz: "quem cuida de .br?"
                                   ──▶ servidores de .br: "quem cuida de com.br?"
                                   ──▶ ...: "quem cuida de sidneiweber.com.br?"
                                   ──▶ servidor autoritativo: "www = 185.199.108.153"
cliente ◀── resposta (guardada em cache pelo TTL) ──┘
```

- **Consulta recursiva**: o cliente pede a resposta final ao resolvedor ("resolva para mim").
- **Consulta iterativa**: o resolvedor pergunta a cada servidor, que responde com uma
  **indicação** de quem sabe mais.

```bash
dig +trace www.debian.org     # mostra cada passo
```

## Tipos de servidor

| Tipo | Função |
|---|---|
| **Recursivo / cache** | Resolve qualquer nome para os clientes e guarda as respostas (ex.: 1.1.1.1, o roteador de casa) |
| **Autoritativo** | Responde com autoridade pelas zonas que hospeda |
| **Primário (master)** | Autoritativo onde a zona é editada |
| **Secundário (slave)** | Autoritativo que copia a zona do primário (transferência de zona) |

## Zonas e delegação

Uma **zona** é a parte da árvore administrada por um conjunto de servidores. O dono de
`com.br` **delega** `sidneiweber.com.br` publicando registros `NS` que apontam para os
servidores autoritativos dessa zona.

| Zona | Exemplo | Resolve |
|---|---|---|
| **Direta** | `curso.lan` | Nome → IP |
| **Reversa** | `56.168.192.in-addr.arpa` | IP → nome |

## Tipos de registro

| Registro | Função | Exemplo |
|---|---|---|
| `SOA` | Dados da zona: servidor primário, e-mail do responsável, serial, tempos | — |
| `NS` | Servidores autoritativos da zona | `curso.lan. NS srv01.curso.lan.` |
| `A` / `AAAA` | Nome → IPv4 / IPv6 | `srv01 A 192.168.56.10` |
| `CNAME` | Apelido para outro nome | `www CNAME srv01` |
| `MX` | Servidor de e-mail do domínio (com prioridade) | `@ MX 10 mail` |
| `PTR` | IP → nome (zona reversa) | `10 PTR srv01.curso.lan.` |
| `TXT` | Texto livre (SPF, DKIM, verificação de domínio) | `@ TXT "v=spf1 mx -all"` |
| `SRV` | Localização de serviços (LDAP, SIP, AD) | `_ldap._tcp SRV 0 0 389 srv01` |
| `CAA` | Quais autoridades podem emitir certificados TLS | `@ CAA 0 issue "letsencrypt.org"` |

## TTL

Cada registro tem um **TTL** (em segundos): por quanto tempo os resolvedores podem guardá-lo
em cache. Antes de uma migração, reduza o TTL com antecedência para que a mudança se
propague rapidamente.
