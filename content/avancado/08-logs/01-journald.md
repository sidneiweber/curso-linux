+++
title = "journald e journalctl"
weight = 1
description = "Consultando, filtrando e configurando o journal."
+++

O **systemd-journald** coleta mensagens do kernel, dos serviços (tudo o que escrevem na
saída padrão) e do syslog, e guarda em formato binário indexado — o que permite filtros
rápidos e precisos.

## Consultando

```bash
journalctl                    # tudo (mais antigo primeiro); use / para buscar
journalctl -r                 # mais recentes primeiro
journalctl -f                 # acompanha em tempo real (como tail -f)
journalctl -n 50              # últimas 50 linhas
journalctl -e                 # abre no final
```

## Filtrando

```bash
journalctl -u ssh                         # por serviço
journalctl -u nginx -u php8.4-fpm         # vários
journalctl -p err                         # prioridade err ou mais grave
journalctl --since "2026-11-18 08:00" --until "1 hour ago"
journalctl --since today
journalctl -b -1                          # boot anterior
journalctl _UID=1000                      # mensagens de um usuário
journalctl /usr/sbin/sshd                 # por executável
journalctl -k                             # kernel
journalctl -t sudo                        # por identificador (tag)
```

## Formatos de saída

```bash
journalctl -u ssh -o short-iso           # data ISO
journalctl -u ssh -o json-pretty | less  # todos os campos
journalctl -u ssh -o cat                 # só a mensagem
```

## Escrevendo no log

```bash
logger "Backup iniciado"
logger -t backup -p user.err "Falha no backup"
journalctl -t backup
```

## Espaço e retenção

```bash
journalctl --disk-usage
sudo journalctl --vacuum-size=500M      # reduz para 500 MB
sudo journalctl --vacuum-time=30d       # apaga o que tiver mais de 30 dias
```

Limites permanentes em `/etc/systemd/journald.conf.d/limites.conf`:

```ini
[Journal]
Storage=persistent
SystemMaxUse=1G
MaxRetentionSec=3month
```

```bash
sudo systemctl restart systemd-journald
```

{{% notice tip "Quem pode ler os logs" %}}
Usuários comuns só veem as próprias mensagens. Para ver todas sem `sudo`, adicione o usuário
ao grupo **`systemd-journal`** (ou `adm`, no Debian/Ubuntu).
{{% /notice %}}
