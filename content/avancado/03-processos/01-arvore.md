+++
title = "A árvore de processos"
weight = 1
description = "PID, PPID, estados, /proc e processos zumbis."
+++

Todo processo é criado por outro (o **pai**) — exceto o PID 1, o systemd. O resultado é uma árvore:

```bash
pstree -p | less        # árvore com PIDs
ps -ef --forest         # o mesmo, com mais detalhes
ps -o pid,ppid,user,stat,cmd -p $$   # o seu próprio shell ($$ = PID do shell)
```

## Estados de um processo

A coluna `STAT` do `ps` mostra o estado:

| Estado | Significado |
|---|---|
| `R` | Executando ou pronto para executar |
| `S` | Dormindo, esperando algo (a maioria) |
| `D` | Dormindo **sem poder ser interrompido** (geralmente esperando disco/rede) |
| `T` | Parado (`Ctrl+Z` ou `SIGSTOP`) |
| `Z` | **Zumbi**: terminou, mas o pai ainda não leu o código de saída |

Sufixos comuns: `s` (líder de sessão), `+` (em primeiro plano), `<` / `N` (prioridade alta/baixa).

{{% notice note "Zumbis" %}}
Um zumbi não consome CPU nem memória, só uma entrada na tabela de processos. Não adianta
dar `kill` nele — ele já morreu. Quem resolve é o **pai** (ou encerrar o pai, para que o
systemd "adote" e limpe o zumbi). Muitos zumbis indicam um bug no programa pai.
{{% /notice %}}

## /proc: o processo por dentro

Cada processo tem um diretório `/proc/<PID>` com informações geradas pelo kernel:

```bash
ls /proc/$$
cat /proc/$$/status       # estado, memória, usuário
tr '\0' ' ' < /proc/$$/cmdline; echo   # linha de comando
ls -l /proc/$$/cwd        # diretório atual
ls -l /proc/$$/fd         # arquivos abertos
cat /proc/$$/environ | tr '\0' '\n'    # variáveis de ambiente
```

## Encontrando processos

```bash
pgrep -a ssh              # PID e comando de quem tem "ssh" no nome
pgrep -u maria            # processos do usuário maria
pidof sshd                # PIDs exatos de um programa
ps aux --sort=-%mem | head    # os que mais usam memória
ps aux --sort=-%cpu | head    # os que mais usam CPU
```
