+++
title = "Nome da máquina e cliente DNS"
weight = 3
description = "hostname, /etc/hosts, resolv.conf, systemd-resolved e nsswitch."
+++

## Nome da máquina

```bash
hostnamectl                               # nome, SO, kernel, tipo de máquina
sudo hostnamectl set-hostname srv-web01
```

Atualize também a linha correspondente em `/etc/hosts`:

```text
127.0.0.1   localhost
127.0.1.1   srv-web01.empresa.local srv-web01
```

## Como um nome vira IP

O arquivo `/etc/nsswitch.conf` define a ordem de consulta:

```text
hosts: files mdns4_minimal [NOTFOUND=return] dns
```

1. **files**: `/etc/hosts` — bom para apelidos e testes (ex.: apontar um domínio para um
   servidor novo antes de mudar o DNS).
2. **mdns**: nomes `.local` na rede (Avahi).
3. **dns**: os servidores DNS configurados.

```bash
getent hosts srv-web01     # resolve seguindo o nsswitch (como os programas fazem)
```

## Servidores DNS: /etc/resolv.conf

```text
nameserver 1.1.1.1
nameserver 9.9.9.9
search empresa.local
```

`search` permite usar nomes curtos: `ping intranet` tenta `intranet.empresa.local`.

## systemd-resolved

No Ubuntu (e no Debian, se instalado), o `/etc/resolv.conf` aponta para um resolvedor local,
`127.0.0.53`, gerenciado pelo **systemd-resolved** — que recebe os servidores DNS do
NetworkManager/Netplan, faz cache e suporta DNS por interface (útil com VPNs).

```bash
ls -l /etc/resolv.conf          # link para /run/systemd/resolve/stub-resolv.conf
resolvectl status               # servidores DNS de cada interface
resolvectl query example.com
sudo resolvectl flush-caches
```

{{% notice warning "Não edite resolv.conf à toa" %}}
Se o `/etc/resolv.conf` for um link gerenciado (resolved, NetworkManager, resolvconf), as
alterações manuais serão sobrescritas. Configure o DNS na ferramenta de rede em uso.
{{% /notice %}}
