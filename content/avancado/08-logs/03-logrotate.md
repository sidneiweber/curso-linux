+++
title = "Rotação de logs: logrotate"
weight = 3
description = "Como os logs são compactados e apagados, e como configurar para sua aplicação."
+++

Sem controle, os logs crescem até encher o disco. O **logrotate** periodicamente renomeia o
log atual (`app.log` → `app.log.1`), compacta os antigos e apaga os mais velhos.
Ele roda uma vez por dia pelo `logrotate.timer` do systemd.

```bash
systemctl list-timers logrotate.timer
ls /etc/logrotate.d/
cat /etc/logrotate.d/apt
```

## Configurando para uma aplicação

`/etc/logrotate.d/minha-app`:

```text
/var/log/minha-app/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 www-data adm
    sharedscripts
    postrotate
        systemctl reload minha-app > /dev/null 2>&1 || true
    endscript
}
```

| Diretiva | Efeito |
|---|---|
| `daily` / `weekly` / `monthly` / `size 100M` | Quando rotacionar |
| `rotate 14` | Quantos arquivos antigos manter |
| `compress` / `delaycompress` | Compacta, mas deixa o mais recente (`.1`) sem compactar |
| `missingok` / `notifempty` | Não reclama se não existir; não rotaciona arquivo vazio |
| `create 0640 dono grupo` | Cria o novo log com essas permissões |
| `copytruncate` | Copia e zera o original — para apps que não sabem reabrir o log |
| `postrotate … endscript` | Comando após rotacionar (avisar o programa) |

## Testando

```bash
sudo logrotate -d /etc/logrotate.d/minha-app   # simula, sem alterar nada
sudo logrotate -f /etc/logrotate.d/minha-app   # força a rotação agora
```
