+++
title = "cron"
weight = 1
description = "crontab, sintaxe, diretórios do sistema e controle de acesso."
+++

O daemon **cron** verifica a cada minuto se há tarefas a executar.

## crontab do usuário

```bash
crontab -e      # edita as tarefas do seu usuário
crontab -l      # lista
crontab -r      # apaga TODAS (cuidado: fica ao lado do -e no teclado)
sudo crontab -u maria -l
```

## Sintaxe

```text
┌───────── minuto (0–59)
│ ┌─────── hora (0–23)
│ │ ┌───── dia do mês (1–31)
│ │ │ ┌─── mês (1–12)
│ │ │ │ ┌─ dia da semana (0–7; 0 e 7 = domingo)
│ │ │ │ │
* * * * *  comando
```

| Expressão | Quando |
|---|---|
| `30 2 * * *` | Todo dia às 02:30 |
| `0 */4 * * *` | A cada 4 horas |
| `*/15 * * * *` | A cada 15 minutos |
| `0 8 * * 1-5` | Dias úteis às 08:00 |
| `0 0 1 * *` | Todo dia 1º, à meia-noite |
| `0 9,18 * * *` | Às 09:00 e às 18:00 |
| `@reboot` | Ao iniciar o sistema |
| `@daily`, `@weekly`, `@monthly` | Atalhos |

O site [crontab.guru](https://crontab.guru) ajuda a montar e conferir expressões.

## Exemplo

```text
MAILTO=""
PATH=/usr/local/bin:/usr/bin:/bin

30 2 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1
```

{{% notice warning "As três armadilhas do cron" %}}
1. **PATH reduzido**: o cron não carrega o seu `.bashrc`. Use caminhos completos ou defina `PATH`.
2. **Saída**: sem redirecionar, a saída é enviada por e-mail (ou perdida). Redirecione para um log.
3. **`%`** tem significado especial na crontab; escape como `\%` (ex.: em `date +\%F`).
{{% /notice %}}

## Tarefas do sistema

| Local | Formato |
|---|---|
| `/etc/crontab` e `/etc/cron.d/*` | Igual à crontab, mas com **campo de usuário** antes do comando |
| `/etc/cron.hourly/`, `daily/`, `weekly/`, `monthly/` | Scripts executáveis, sem extensão no nome |

```text
# /etc/cron.d/limpeza
0 3 * * 0  root  find /tmp -type f -mtime +7 -delete
```

## Controle de acesso

- Se existir `/etc/cron.allow`, **só** os usuários listados podem usar `crontab`.
- Senão, quem estiver em `/etc/cron.deny` é bloqueado.

## Monitorando

```bash
journalctl -u cron --since today     # execuções (no Debian o serviço se chama cron)
```

Para saber se uma tarefa **falhou silenciosamente**, faça o script registrar sucesso e erro,
ou use um serviço de *heartbeat* (healthchecks.io, Uptime Kuma) chamado ao final do script.
