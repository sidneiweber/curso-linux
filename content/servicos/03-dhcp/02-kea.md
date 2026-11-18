+++
title = "Servidor DHCP com Kea"
weight = 2
description = "Instalação, configuração da subnet, opções, reservas e leases."
+++

## Instalação

```bash
sudo apt install kea-dhcp4-server
```

## Configuração

O arquivo é `/etc/kea/kea-dhcp4.conf` (JSON, que aceita comentários). Guarde o original
como referência e crie um novo:

```bash
sudo mv /etc/kea/kea-dhcp4.conf /etc/kea/kea-dhcp4.conf.orig
sudo vim /etc/kea/kea-dhcp4.conf
```

```json
{
  "Dhcp4": {
    "interfaces-config": {
      "interfaces": [ "enp0s8" ]
    },
    "lease-database": {
      "type": "memfile",
      "persist": true,
      "name": "/var/lib/kea/kea-leases4.csv"
    },
    "valid-lifetime": 43200,
    "renew-timer": 21600,
    "rebind-timer": 37800,

    "option-data": [
      { "name": "domain-name-servers", "data": "192.168.56.10" },
      { "name": "domain-name", "data": "curso.lan" },
      { "name": "domain-search", "data": "curso.lan" }
    ],

    "subnet4": [
      {
        "id": 1,
        "subnet": "192.168.56.0/24",
        "pools": [ { "pool": "192.168.56.100 - 192.168.56.199" } ],
        "option-data": [
          { "name": "routers", "data": "192.168.56.10" }
        ],
        "reservations": [
          {
            "hw-address": "08:00:27:aa:bb:cc",
            "ip-address": "192.168.56.20",
            "hostname": "cli01"
          }
        ]
      }
    ],

    "loggers": [
      {
        "name": "kea-dhcp4",
        "output-options": [ { "output": "stdout" } ],
        "severity": "INFO"
      }
    ]
  }
}
```

- **interfaces**: só atende na rede interna — nunca na interface da Internet.
- **pool**: a faixa distribuída dinamicamente. Deixe fora dela os IPs fixos (`.1`–`.99`).
- **reservations**: troque o MAC pelo do `cli01` (`ip link` no cliente).
- O DNS aponta para o próprio `srv01`, que configuraremos no próximo capítulo. Até lá,
  use `1.1.1.1` se quiser navegar pelo cliente.

```bash
sudo kea-dhcp4 -t /etc/kea/kea-dhcp4.conf     # valida
sudo systemctl restart kea-dhcp4-server
sudo systemctl enable kea-dhcp4-server
journalctl -u kea-dhcp4-server -f
```

## Concessões

```bash
sudo cat /var/lib/kea/kea-leases4.csv
```

Cada linha registra IP, MAC, validade e nome do cliente.

{{% notice note "Alternativa rápida: dnsmasq" %}}
Em redes pequenas, o **dnsmasq** faz DHCP **e** DNS com poucas linhas em
`/etc/dnsmasq.d/lan.conf`:

```text
interface=enp0s8
dhcp-range=192.168.56.100,192.168.56.199,12h
dhcp-option=option:router,192.168.56.10
dhcp-host=08:00:27:aa:bb:cc,cli01,192.168.56.20
domain=curso.lan
```
{{% /notice %}}
