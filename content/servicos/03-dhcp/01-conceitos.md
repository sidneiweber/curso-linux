+++
title = "Conceitos"
weight = 1
description = "IP estático x dinâmico, o processo DORA, concessões e reservas."
+++

## Estático ou dinâmico?

| | IP estático | IP dinâmico (DHCP) |
|---|---|---|
| Configuração | Manual, em cada máquina | Automática |
| Uso | Servidores, roteadores, impressoras | Estações, notebooks, celulares |
| Risco | Conflitos e erros de digitação | Depende do servidor DHCP |

Um meio-termo muito usado: **reservas** — o DHCP sempre entrega o mesmo IP para um
determinado MAC. A configuração fica centralizada, mas o endereço é previsível.

## Como funciona: DORA

O cliente ainda não tem IP, então tudo começa por **broadcast** (UDP, portas 67/68):

```text
cliente                                   servidor
   │ ── DHCPDISCOVER (há algum servidor?) ──▶ │
   │ ◀── DHCPOFFER (ofereço 192.168.56.101) ── │
   │ ── DHCPREQUEST (quero esse IP) ─────────▶ │
   │ ◀── DHCPACK (confirmado, por 12 h) ────── │
```

## Concessão (lease)

O IP é **emprestado** por um tempo. Na metade do prazo o cliente tenta renovar; se o
servidor sumir, ele continua usando o IP até o fim da concessão.

## Opções

Além do IP, o servidor entrega **opções**: máscara, gateway (`routers`), servidores DNS,
domínio de busca, servidor NTP, servidor de boot pela rede (PXE)…

## Servidores DHCP no Linux

| Servidor | Situação |
|---|---|
| **ISC DHCP** (`isc-dhcp-server`) | Clássico e muito documentado, mas **descontinuado** desde 2022 |
| **Kea** (`kea-dhcp4-server`) | Sucessor oficial do ISC DHCP, configurado em JSON — usaremos este |
| **dnsmasq** | DHCP + DNS simples em um só programa; ótimo para redes pequenas e laboratórios |

{{% notice warning "Um servidor DHCP por rede" %}}
Dois servidores DHCP respondendo na mesma rede causam caos: clientes recebem gateway e DNS
errados. No laboratório, a rede interna do VirtualBox **não** tem DHCP próprio — mas
verifique antes de ligar o seu.
{{% /notice %}}
