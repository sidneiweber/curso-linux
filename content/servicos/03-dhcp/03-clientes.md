+++
title = "Configurando os clientes"
weight = 3
description = "Clientes Linux e Windows, renovação e diagnóstico."
+++

## Linux

Com NetworkManager (desktops), DHCP é o padrão — basta conectar. Para forçar uma renovação:

```bash
nmcli con down "Wired connection 1" && nmcli con up "Wired connection 1"
```

Com ifupdown (`/etc/network/interfaces`):

```text
auto enp0s3
iface enp0s3 inet dhcp
```

```bash
sudo dhclient -r enp0s3 && sudo dhclient -v enp0s3   # libera e pede de novo
```

Conferindo o que foi recebido:

```bash
ip -br addr
ip route
resolvectl status       # ou: cat /etc/resolv.conf
```

## Windows

```text
ipconfig /release
ipconfig /renew
ipconfig /all
```

## Diagnóstico

No servidor, observe a conversa DORA:

```bash
sudo tcpdump -ni enp0s8 -v port 67 or port 68
```

| Sintoma | Causa provável |
|---|---|
| Cliente recebe `169.254.x.x` | Nenhum servidor respondeu: serviço parado, interface errada, firewall |
| Cliente recebe IP de outra faixa | Outro servidor DHCP na rede |
| `DHCPOFFER` sai, mas o cliente não aceita | Firewall do servidor bloqueando UDP 67 |
| Pool esgotado | Aumente a faixa ou reduza o tempo de concessão |

## Exercícios

1. Configure o Kea e faça o `cli01` receber IP, gateway e DNS automaticamente.
2. Crie uma reserva para o `cli01` com o IP `192.168.56.20` e confirme após renovar.
3. Pare o Kea, renove o cliente e observe o endereço `169.254.x.x`.
