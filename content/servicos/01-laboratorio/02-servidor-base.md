+++
title = "Preparando o servidor"
weight = 2
description = "IP fixo, nome, pacotes básicos e acesso SSH ao srv01."
+++

## IP fixo na interface interna

Descubra os nomes das interfaces (`ip -br link`) e edite `/etc/network/interfaces`:

```text
auto lo
iface lo inet loopback

# Internet (NAT do VirtualBox)
auto enp0s3
iface enp0s3 inet dhcp

# Rede interna do laboratório
auto enp0s8
iface enp0s8 inet static
    address 192.168.56.10/24
```

```bash
sudo systemctl restart networking
ip -br addr
```

## Nome da máquina

```bash
sudo hostnamectl set-hostname srv01
```

`/etc/hosts`:

```text
127.0.0.1       localhost
192.168.56.10   srv01.curso.lan srv01
```

## Pacotes úteis

```bash
sudo apt update && sudo apt full-upgrade
sudo apt install vim curl wget dnsutils tcpdump nmap net-tools bash-completion tree
```

## Acessando pelo SSH

Trabalhar no console da VM é desconfortável. Do seu computador (ou do `cli01`):

```bash
ssh usuario@192.168.56.10
```

O SSH é detalhado no capítulo [Acesso remoto com SSH]({{% relref "/servicos/05-ssh" %}}).

{{% notice warning "Snapshot!" %}}
Com o servidor base pronto, desligue a VM e tire um snapshot chamado **base**. Antes de
cada capítulo, você pode voltar a ele e começar do zero.
{{% /notice %}}
