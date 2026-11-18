+++
title = "Monitorando logs"
weight = 4
description = "Filtrando eventos importantes, alertas simples e ferramentas."
+++

## Na linha de comando

```bash
journalctl -f -p warning                 # só avisos e erros, ao vivo
journalctl -u ssh --since today | grep -c 'Failed password'
sudo tail -F /var/log/nginx/error.log    # -F continua seguindo após a rotação
```

Alguns padrões que valem atenção:

| O que procurar | Onde |
|---|---|
| `Failed password`, `Invalid user` | `journalctl -u ssh` / `auth.log` |
| `Out of memory`, `I/O error`, `EXT4-fs error` | `journalctl -k` |
| Serviços em falha | `systemctl --failed` |
| `segfault` | `journalctl -k` |

## Resumo diário: logwatch

```bash
sudo apt install logwatch
sudo logwatch --detail med --range today --output stdout | less
```

Com um servidor de e-mail configurado, ele envia o relatório diariamente para o root.

## Reagindo a eventos: fail2ban

O **fail2ban** lê os logs e bloqueia no firewall os IPs que erram a senha muitas vezes:

```bash
sudo apt install fail2ban
sudo tee /etc/fail2ban/jail.local <<'J'
[sshd]
enabled  = true
backend  = systemd
maxretry = 5
bantime  = 1h
J
sudo systemctl restart fail2ban
sudo fail2ban-client status sshd
```

## Em escala

Com muitos servidores, os logs vão para uma plataforma central com busca e alertas:
**Grafana Loki**, **Elasticsearch/OpenSearch**, **Graylog**. Os coletores (Promtail/Alloy,
Fluent Bit, Vector) leem o journal e os arquivos de `/var/log` de cada máquina.

## Exercícios

1. Liste os erros do boot atual e descubra quais serviços os geraram.
2. Crie uma regra no rsyslog que grave `local5` em `/var/log/teste.log`, escreva nela com
   `logger` e configure a rotação diária mantendo 7 arquivos.
3. Instale o fail2ban, erre a senha do SSH algumas vezes a partir de outra VM e veja o IP
   ser bloqueado.
