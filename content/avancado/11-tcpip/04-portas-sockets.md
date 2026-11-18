+++
title = "Portas e sockets"
weight = 4
description = "Portas conhecidas, sockets, estados TCP e o comando ss."
+++

O IP leva o pacote até a **máquina**; a **porta** (0–65535) indica a qual **programa** ele se
destina. A combinação `IP:porta` de origem e destino + protocolo identifica um **socket** —
uma conexão única.

```text
192.168.1.10:51234  ⇄  93.184.215.14:443   (TCP)
 cliente, porta efêmera     servidor web HTTPS
```

## Portas conhecidas

| Porta | Serviço | Porta | Serviço |
|---|---|---|---|
| 22/tcp | SSH | 143/tcp, 993/tcp | IMAP, IMAPS |
| 25/tcp, 587/tcp | SMTP, submissão | 389/tcp, 636/tcp | LDAP, LDAPS |
| 53/udp+tcp | DNS | 445/tcp | SMB (Samba) |
| 67–68/udp | DHCP | 631/tcp | IPP (CUPS) |
| 80/tcp, 443/tcp | HTTP, HTTPS | 2049/tcp | NFS |
| 123/udp | NTP | 3306 / 5432 | MySQL / PostgreSQL |

Portas **abaixo de 1024** só podem ser abertas por root (ou com a *capability*
`CAP_NET_BIND_SERVICE`). A lista local fica em `/etc/services`.

## ss: quem está escutando e conectado

```bash
sudo ss -tulpn          # t=TCP u=UDP l=escutando p=processo n=números
ss -tn                  # conexões TCP estabelecidas
ss -tn state established '( dport = :443 )'
ss -s                   # resumo
```

```text
Netid State  Local Address:Port  Peer Address:Port Process
tcp   LISTEN 0.0.0.0:22          0.0.0.0:*         users:(("sshd",pid=812,fd=3))
tcp   LISTEN 127.0.0.1:5432      0.0.0.0:*         users:(("postgres",pid=901,fd=6))
```

Repare no endereço local: `0.0.0.0` (ou `*`/`[::]`) aceita conexões de **qualquer** interface;
`127.0.0.1` só da própria máquina — o banco acima não está exposto à rede.

{{% notice note "netstat" %}}
O `netstat` (pacote `net-tools`) está obsoleto; `ss` é o substituto e aceita opções
parecidas (`netstat -tulpn` → `ss -tulpn`).
{{% /notice %}}

## Testando portas

```bash
nc -zv servidor 22            # a porta está aberta?
curl -v telnet://servidor:25  # conversa "crua" com o serviço
sudo apt install nmap
nmap -sT servidor             # varre as portas mais comuns (só em máquinas suas!)
```

## Exercícios

1. Calcule rede, broadcast e faixa de hosts de `172.20.14.77/20` e confira com `ipcalc`.
2. Divida `192.168.50.0/24` em 8 sub-redes iguais.
3. Liste as portas abertas na sua máquina e identifique quais estão expostas à rede.
4. Capture com `tcpdump` o handshake de um `curl https://example.com`.
