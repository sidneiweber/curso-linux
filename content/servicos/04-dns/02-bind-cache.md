+++
title = "BIND 9: servidor recursivo"
weight = 2
description = "Instalação, recursão restrita à rede interna e encaminhadores."
+++

O **BIND 9** é o servidor DNS mais tradicional e completo. (Alternativas: **Unbound** só
recursivo; **PowerDNS** e **Knot** autoritativos.)

```bash
sudo apt install bind9 bind9-utils bind9-doc
```

## Arquivos

| Arquivo | Conteúdo |
|---|---|
| `/etc/bind/named.conf` | Arquivo principal — só inclui os demais |
| `/etc/bind/named.conf.options` | Opções globais (recursão, encaminhadores, acesso) |
| `/etc/bind/named.conf.local` | **Suas** zonas |
| `/var/lib/bind/` | Arquivos de zona que o BIND altera (zonas dinâmicas, secundárias) |

## Opções: recursão só para a rede interna

`/etc/bind/named.conf.options`:

```text
acl "internas" {
    127.0.0.0/8;
    ::1;
    192.168.56.0/24;
};

options {
    directory "/var/cache/bind";

    listen-on { 127.0.0.1; 192.168.56.10; };
    listen-on-v6 { ::1; };

    recursion yes;
    allow-recursion { internas; };
    allow-query     { internas; };
    allow-transfer  { none; };

    // opcional: em vez de consultar a raiz, encaminhar para outro resolvedor
    // forwarders { 1.1.1.1; 9.9.9.9; };

    dnssec-validation auto;
};
```

{{% notice warning "Resolvedor aberto" %}}
Nunca deixe `recursion yes` para qualquer origem em um servidor exposto à Internet: ele vira
um **resolvedor aberto**, usado em ataques de amplificação (DDoS). Sempre restrinja com
`allow-recursion`.
{{% /notice %}}

```bash
sudo named-checkconf
sudo systemctl restart named      # o serviço chama-se named (bind9 é um apelido)
sudo systemctl enable named
sudo ss -tulpn | grep :53
```

## Testando

```bash
dig @127.0.0.1 debian.org
dig @127.0.0.1 debian.org     # a 2ª vez é instantânea: veio do cache ("Query time: 0 msec")
```

No `cli01` (que recebe `192.168.56.10` como DNS via DHCP):

```bash
dig debian.org
resolvectl status
```

```bash
sudo rndc status
sudo rndc flush         # limpa o cache
sudo rndc dumpdb -cache && sudo less /var/cache/bind/named_dump.db
```
