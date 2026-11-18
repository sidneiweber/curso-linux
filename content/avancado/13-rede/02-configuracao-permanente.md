+++
title = "Configuração permanente"
weight = 2
description = "NetworkManager (nmcli), Netplan e /etc/network/interfaces."
+++

Cada família de distribuição tem sua ferramenta. Descubra qual a sua máquina usa:

```bash
systemctl is-active NetworkManager systemd-networkd networking
ls /etc/netplan/ 2>/dev/null
```

| Onde | Ferramenta |
|---|---|
| Desktops (qualquer distro), Fedora/RHEL | **NetworkManager** (`nmcli`) |
| Ubuntu Server | **Netplan** (que gera config para `systemd-networkd` ou NetworkManager) |
| Debian Server (instalação mínima) | **ifupdown** (`/etc/network/interfaces`) |

## NetworkManager: nmcli

```bash
nmcli device                     # interfaces e conexões
nmcli connection show            # perfis de conexão

# IP fixo
sudo nmcli con mod "Wired connection 1" \
  ipv4.method manual \
  ipv4.addresses 192.168.1.50/24 \
  ipv4.gateway 192.168.1.1 \
  ipv4.dns "1.1.1.1 9.9.9.9"
sudo nmcli con up "Wired connection 1"

# voltar ao DHCP
sudo nmcli con mod "Wired connection 1" ipv4.method auto ipv4.addresses "" ipv4.gateway "" ipv4.dns ""

nmcli device wifi list
nmcli device wifi connect MinhaRede --ask
```

Também há a interface em modo texto **`nmtui`**.

## Netplan (Ubuntu)

`/etc/netplan/50-cloud-init.yaml` (ou um novo arquivo, ex.: `01-rede.yaml`):

```yaml
network:
  version: 2
  ethernets:
    enp3s0:
      dhcp4: false
      addresses: [192.168.1.50/24]
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [1.1.1.1, 9.9.9.9]
        search: [empresa.local]
```

```bash
sudo chmod 600 /etc/netplan/*.yaml
sudo netplan try       # aplica e desfaz em 120 s se você não confirmar
sudo netplan apply
```

{{% notice tip "netplan try" %}}
Em servidores remotos, use sempre `netplan try`: se a nova configuração derrubar o seu
SSH, ela é revertida sozinha.
{{% /notice %}}

## ifupdown (Debian)

`/etc/network/interfaces`:

```text
auto lo
iface lo inet loopback

auto enp3s0
iface enp3s0 inet static
    address 192.168.1.50/24
    gateway 192.168.1.1
    dns-nameservers 1.1.1.1 9.9.9.9

# para DHCP:
# iface enp3s0 inet dhcp
```

```bash
sudo ifdown enp3s0 && sudo ifup enp3s0
# ou
sudo systemctl restart networking
```

(`dns-nameservers` exige o pacote `resolvconf`; sem ele, configure o DNS em `/etc/resolv.conf`.)
