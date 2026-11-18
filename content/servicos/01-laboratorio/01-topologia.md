+++
title = "Topologia e máquinas"
weight = 1
description = "As VMs, a rede do laboratório e os endereços usados nos exemplos."
+++

## Topologia

```text
                 Internet
                    │
             (NAT do VirtualBox)
                    │ enp0s3 (DHCP)
             ┌──────┴──────┐
             │    srv01    │  Debian 13 — servidor
             │ 192.168.56.10│
             └──────┬──────┘
                    │ enp0s8
   ─────────────────┴──────────────────  rede interna 192.168.56.0/24
          │                        │
   ┌──────┴──────┐          ┌──────┴──────┐
   │    cli01    │          │    cli02    │  (opcional)
   │192.168.56.20│          │ Windows 10/11│
   └─────────────┘          └─────────────┘
```

| Máquina | Sistema | Endereço | Papel |
|---|---|---|---|
| `srv01` | Debian 13 (sem interface gráfica) | `192.168.56.10` | Roteador, DHCP, DNS e demais serviços |
| `cli01` | Debian 13 ou Ubuntu Desktop | DHCP (reserva `.20`) | Cliente Linux |
| `cli02` | Windows (opcional) | DHCP | Cliente Windows (Samba) |

- **Domínio:** `curso.lan`
- **Rede interna:** `192.168.56.0/24`, gateway `192.168.56.10` (o próprio `srv01`)

{{% notice tip "Por que .lan?" %}}
Não use domínios que existem de verdade (nem `.local`, reservado para mDNS). Para redes
internas de produção, o recomendado é um subdomínio de um domínio seu, como
`interno.empresa.com.br`. No laboratório, `curso.lan` é suficiente.
{{% /notice %}}

## Criando as VMs (VirtualBox)

1. **srv01**: 1–2 vCPU, 2 GB de RAM, 20 GB de disco.
   - Adaptador 1: **NAT** (acesso à Internet para instalar pacotes).
   - Adaptador 2: **Rede interna**, nome `curso`.
   - Instale o Debian com apenas **SSH server** e **utilitários padrão do sistema**.
2. **cli01**: 2 GB de RAM; **somente** o adaptador de **Rede interna** `curso`.
   O acesso à Internet virá através do `srv01`, que será o roteador.

No GNOME Boxes, virt-manager/KVM ou Proxmox, a ideia é a mesma: uma rede **isolada** entre
as VMs e o servidor com uma segunda interface para a Internet.
