+++
title = "IDS de rede com Suricata"
weight = 2
description = "Analisando o tráfego em busca de ataques conhecidos."
+++

Um **NIDS** (*Network Intrusion Detection System*) analisa o tráfego da rede e compara com
**assinaturas** de ataques conhecidos: varreduras, exploits, malware se comunicando,
protocolos anômalos.

| Ferramenta | Situação |
|---|---|
| **Snort** | O NIDS clássico, ainda mantido (Snort 3) |
| **Suricata** | Compatível com as regras do Snort, multithread, saída em JSON — o mais usado hoje em Linux |
| **Zeek** | Foco em registrar e analisar o comportamento da rede, não só assinaturas |

## Instalação no srv01

Como o `srv01` é o roteador do laboratório, todo o tráfego da LAN passa por ele.

```bash
sudo apt install suricata suricata-update jq
```

`/etc/suricata/suricata.yaml` — ajuste a rede protegida e a interface:

```yaml
vars:
  address-groups:
    HOME_NET: "[192.168.56.0/24]"

af-packet:
  - interface: enp0s8
```

```bash
sudo suricata-update                 # baixa as regras (Emerging Threats Open)
sudo suricata -T -c /etc/suricata/suricata.yaml    # testa a configuração
sudo systemctl restart suricata
```

## Alertas

```bash
sudo tail -f /var/log/suricata/fast.log
sudo jq 'select(.event_type=="alert") | {ts: .timestamp, src: .src_ip, dst: .dest_ip, alerta: .alert.signature}' /var/log/suricata/eve.json
```

Gere eventos de teste a partir do `cli01`:

```bash
curl http://testmynids.org/uid/index.html     # dispara a regra "GPL ATTACK_RESPONSE id check returned root"
sudo nmap -sS -T4 srv01                       # varredura
```

## IDS × IPS

- **IDS**: só **detecta** e alerta (o que configuramos).
- **IPS**: fica no caminho do tráfego e **bloqueia** (Suricata em modo inline com
  `nfqueue`). Mais proteção, mas um falso positivo derruba tráfego legítimo — comece como
  IDS e ajuste as regras antes.

Em ambientes maiores, os alertas do Suricata, do auditd e dos logs vão para um **SIEM**
(Wazuh, Security Onion, Elastic) que correlaciona eventos de várias fontes.
