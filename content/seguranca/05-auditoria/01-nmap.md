+++
title = "Varredura de rede com Nmap"
weight = 1
description = "Descobrindo hosts, portas, serviços e versões na sua rede."
+++

```bash
sudo apt install nmap
```

## Descoberta de hosts

```bash
sudo nmap -sn 192.168.56.0/24          # quem está ativo na rede (sem varrer portas)
```

## Portas

```bash
sudo nmap srv01                         # 1000 portas TCP mais comuns (SYN scan)
sudo nmap -p- srv01                     # todas as 65535 portas TCP
sudo nmap -sU --top-ports 50 srv01      # UDP (lento)
```

| Estado | Significado |
|---|---|
| `open` | Um serviço está aceitando conexões |
| `closed` | A máquina respondeu, mas não há serviço |
| `filtered` | Sem resposta — um firewall descartou |

## Serviços, versões e sistema

```bash
sudo nmap -sV srv01                     # identifica o programa e a versão
sudo nmap -O srv01                      # tenta adivinhar o sistema operacional
sudo nmap -A srv01                      # -sV -O + scripts padrão + traceroute
```

## Scripts (NSE)

O Nmap tem centenas de scripts para verificações específicas:

```bash
nmap --script ssl-enum-ciphers -p 443 srv01     # protocolos e cifras TLS
nmap --script ssh2-enum-algos -p 22 srv01       # algoritmos do SSH
nmap --script vuln srv01                        # verificações de vulnerabilidades conhecidas
nmap --script smb-protocols -p 445 srv01        # versões SMB habilitadas
```

## Usando na prática

1. Faça um inventário: `nmap -sV -oA inventario-$(date +%F) 192.168.56.0/24`
   (`-oA` grava em texto, XML e formato "grepável").
2. Repita periodicamente e compare: `ndiff inventario-antigo.xml inventario-novo.xml`
   mostra portas que abriram ou fecharam.
3. Varra também **de fora** (de outra rede/Internet) para ver o que um atacante vê.

{{% notice warning "Autorização" %}}
Varrer redes de terceiros sem autorização pode ser crime e costuma gerar bloqueios e
reclamações ao seu provedor. Para testar de fora, use um servidor seu, varrendo o seu IP.
{{% /notice %}}
