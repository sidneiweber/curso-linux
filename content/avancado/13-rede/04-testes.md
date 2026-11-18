+++
title = "Testando e diagnosticando"
weight = 4
description = "ping, traceroute, mtr, dig, curl, ssh e um roteiro de diagnóstico."
+++

## Conectividade

```bash
ping -c 4 192.168.1.1          # gateway responde?
ping -c 4 1.1.1.1              # Internet por IP?
ping -c 4 debian.org           # e por nome (DNS)?
traceroute debian.org          # caminho salto a salto (apt install traceroute)
mtr debian.org                 # ping + traceroute contínuo, mostra perda por salto
tracepath debian.org           # sem root, descobre também o MTU
```

## DNS

```bash
sudo apt install dnsutils      # dig, host, nslookup
dig debian.org                 # resposta completa
dig +short debian.org A
dig +short debian.org MX
dig @9.9.9.9 debian.org        # pergunta a um servidor específico
dig -x 1.1.1.1                 # DNS reverso
host debian.org
```

## Aplicação

```bash
curl -I https://debian.org               # só os cabeçalhos
curl -v https://debian.org -o /dev/null  # handshake TLS e cabeçalhos
curl -4 / curl -6                        # força IPv4 ou IPv6
wget https://exemplo.com/arquivo.iso     # downloads
nc -zv servidor 5432                     # porta TCP aberta?
```

## Acesso remoto e transferência

```bash
ssh usuario@servidor
scp arquivo.txt usuario@servidor:/tmp/
rsync -avz pasta/ usuario@servidor:/srv/pasta/
sftp usuario@servidor
```

{{% notice note "telnet e ftp" %}}
`telnet` e `ftp` transmitem tudo — inclusive senhas — **em texto puro**. Use `ssh`,
`scp`/`sftp`/`rsync`. O `telnet` sobrevive apenas como "teste de porta" (prefira `nc`).
{{% /notice %}}

## Roteiro: "a rede não funciona"

| Passo | Comando | Se falhar |
|---|---|---|
| 1. A interface está UP e com link? | `ip -br link` | Cabo, Wi-Fi, driver (`journalctl -k`) |
| 2. Tem IP? | `ip -br addr` | DHCP: `journalctl -u NetworkManager` |
| 3. Tem rota padrão? | `ip route` | Configurar gateway |
| 4. Alcança o gateway? | `ping <gateway>` | Rede local, VLAN, firewall |
| 5. Alcança a Internet por IP? | `ping 1.1.1.1` | Roteador, provedor |
| 6. Resolve nomes? | `dig debian.org` | DNS: `resolvectl status` |
| 7. O serviço responde? | `curl -v`, `nc -zv` | Firewall, serviço parado, porta errada |

## Exercícios

1. Configure um IP fixo na sua VM com a ferramenta que ela usa, reinicie e confirme.
2. Adicione um segundo IP temporário com `ip addr add` e teste-o a partir de outra VM.
3. Aponte `teste.local` para `127.0.0.1` no `/etc/hosts` e confirme com `getent hosts`.
4. Simule uma falha removendo a rota padrão e use o roteiro acima para diagnosticá-la.
