+++
title = "Arquivos e túneis"
weight = 3
description = "scp, sftp, rsync, encaminhamento de portas e SSH como proxy."
+++

## Copiando arquivos

```bash
scp relatorio.pdf srv01:/tmp/                 # local → remoto
scp srv01:/var/log/syslog .                   # remoto → local
scp -r projeto/ srv01:~/                      # diretório
rsync -avz --progress projeto/ srv01:~/projeto/   # incremental, retoma de onde parou
sftp srv01                                    # sessão interativa: ls, cd, get, put
sshfs srv01:/srv/dados ~/remoto               # monta a pasta remota (apt install sshfs)
```

Para usuários que só devem **transferir arquivos**, sem shell, veja o capítulo
[Transferência de arquivos]({{% relref "/servicos/10-transferencia" %}}).

## Encaminhamento de portas (túneis)

**Local** (`-L`): acessar algo que só o servidor alcança. Ex.: o PostgreSQL do `srv01`
escuta apenas em `127.0.0.1:5432`:

```bash
ssh -L 15432:localhost:5432 srv01
# no seu computador: psql -h localhost -p 15432
```

**Remoto** (`-R`): expor um serviço da sua máquina no servidor:

```bash
ssh -R 8080:localhost:3000 srv01
# no srv01: curl localhost:8080 chega na porta 3000 do seu computador
```

**Dinâmico** (`-D`): um proxy SOCKS — o navegador sai para a Internet "pelo" servidor:

```bash
ssh -D 1080 srv01
# configure o navegador com proxy SOCKS5 localhost:1080
```

Use `-N` para só abrir o túnel, sem shell, e `-f` para ir para o segundo plano.

## Executando comandos remotos

```bash
ssh srv01 'df -h /; uptime'
ssh srv01 'sudo journalctl -u named -n 20'
tar czf - pasta | ssh srv01 'tar xzf - -C /tmp'   # copia via pipe
```

## Exercícios

1. Gere uma chave ed25519, copie-a para o `srv01` e desative o login por senha.
2. Permita SSH somente ao grupo `ssh-users` e teste com um usuário fora do grupo.
3. Crie um túnel local para acessar, do seu computador, um serviço que escuta só em
   `127.0.0.1` no `srv01` (ex.: `python3 -m http.server --bind 127.0.0.1 8000`).
