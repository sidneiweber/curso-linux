+++
title = "Timers do systemd"
weight = 2
description = "Alternativa moderna ao cron, com logs, dependências e execução perdida."
+++

Um **timer** é uma unit que ativa um **serviço** em determinados momentos. Comparado ao cron:

- logs no journal (`journalctl -u`) e estado em `systemctl status`;
- `Persistent=true` executa a tarefa **perdida** se a máquina estava desligada no horário;
- `RandomizedDelaySec` espalha a execução (evita 100 servidores ao mesmo tempo);
- limites de CPU/memória, usuário, dependências — tudo que um serviço oferece.

```bash
systemctl list-timers --all
```

## Criando

`/etc/systemd/system/backup.service`:

```ini
[Unit]
Description=Backup diário

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
Nice=10
IOSchedulingClass=idle
```

`/etc/systemd/system/backup.timer`:

```ini
[Unit]
Description=Executa o backup todo dia às 02:30

[Timer]
OnCalendar=*-*-* 02:30:00
Persistent=true
RandomizedDelaySec=10m

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now backup.timer
systemctl list-timers backup.timer
sudo systemctl start backup.service    # executa agora, para testar
journalctl -u backup.service
```

## Expressões OnCalendar

| Expressão | Quando |
|---|---|
| `daily` / `weekly` / `monthly` | Meia-noite do dia / segunda / dia 1º |
| `Mon..Fri 08:00` | Dias úteis às 08:00 |
| `*-*-* *:00/15:00` | A cada 15 minutos |
| `*-*-01 00:00:00` | Todo dia 1º |

Teste a expressão antes:

```bash
systemd-analyze calendar 'Mon..Fri 08:00' --iterations=3
```

## Timers relativos

```ini
[Timer]
OnBootSec=5min          # 5 minutos depois do boot
OnUnitActiveSec=1h      # e depois a cada 1 hora
```
