+++
title = "Processos e recursos do sistema"
weight = 3
description = "ps, top, kill, killall, w, df e du."
+++

Todo programa em execução é um **processo**, identificado por um número: o **PID**.

## ps — fotografia dos processos

```bash
ps            # processos do terminal atual
ps aux        # todos os processos do sistema
ps aux | grep firefox
```

## top / htop — monitoramento em tempo real

```bash
top     # q para sair, k para matar um processo, M ordena por memória
htop    # versão mais amigável (pode ser necessário instalar)
```

## kill e killall — enviando sinais

`kill` envia **sinais** aos processos. Os mais usados:

| Sinal | Número | Efeito |
|---|---|---|
| `SIGTERM` | 15 | pede para o processo terminar (padrão) |
| `SIGKILL` | 9 | encerra imediatamente, sem chance de defesa |
| `SIGHUP` | 1 | reinicia/recarrega configuração (muitos serviços) |
| `SIGSTOP` / `SIGCONT` | 19 / 18 | pausa / continua |

```bash
kill 1234          # envia SIGTERM ao PID 1234
kill -9 1234       # força o encerramento
kill -l            # lista todos os sinais
killall firefox    # envia sinal para todos os processos com esse nome
pkill -u maria     # processos do usuário maria
```

## Primeiro e segundo plano

```bash
sleep 300 &   # executa em segundo plano
jobs          # lista tarefas do terminal
fg %1         # traz a tarefa 1 para o primeiro plano
# Ctrl+Z suspende o processo atual; bg o continua em segundo plano
```

## Recursos do sistema

```bash
w              # usuários logados, uptime e carga
uptime         # há quanto tempo o sistema está ligado
free -h        # uso de memória
df -h          # espaço em disco por partição
du -sh ~/*     # tamanho de cada item do diretório pessoal
```
