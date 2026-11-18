+++
title = "Gerenciando serviços"
weight = 4
description = "systemctl, units e como criar o seu próprio serviço."
+++

Para o systemd, tudo é uma **unit**: serviços (`.service`), pontos de montagem (`.mount`),
agendamentos (`.timer`), sockets (`.socket`), targets (`.target`)…

## systemctl no dia a dia

```bash
systemctl status ssh             # estado, PID, memória e últimas linhas de log
sudo systemctl start ssh         # inicia agora
sudo systemctl stop ssh          # para agora
sudo systemctl restart ssh       # para e inicia
sudo systemctl reload ssh        # relê a configuração sem derrubar conexões
sudo systemctl enable ssh        # inicia automaticamente no boot
sudo systemctl enable --now ssh  # habilita e já inicia
sudo systemctl disable ssh       # não inicia mais no boot
sudo systemctl mask ssh          # impede que seja iniciado, até manualmente
```

```bash
systemctl list-units --type=service              # serviços carregados
systemctl list-units --type=service --state=failed
systemctl list-unit-files --type=service         # habilitados/desabilitados
systemctl is-active ssh; systemctl is-enabled ssh
```

## Onde ficam as units

| Diretório | Origem |
|---|---|
| `/usr/lib/systemd/system/` | Instaladas pelos pacotes — **não edite** |
| `/etc/systemd/system/` | Do administrador — têm prioridade |

Para ver ou ajustar uma unit sem mexer no arquivo do pacote:

```bash
systemctl cat ssh               # mostra a unit e seus complementos
sudo systemctl edit ssh         # cria um "drop-in" com só o que você quer mudar
sudo systemctl daemon-reload    # obrigatório após editar arquivos de unit à mão
```

## Criando um serviço

Um script qualquer vira serviço com poucas linhas. Crie `/etc/systemd/system/meuapp.service`:

```ini
[Unit]
Description=Minha aplicação de exemplo
After=network-online.target
Wants=network-online.target

[Service]
User=www-data
WorkingDirectory=/opt/meuapp
ExecStart=/usr/bin/python3 -m http.server 8080
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now meuapp
systemctl status meuapp
curl -I http://localhost:8080
```

- **`[Unit]`**: descrição e ordem/dependências (`After`, `Wants`, `Requires`).
- **`[Service]`**: o que executar, com qual usuário, e o que fazer se cair (`Restart`).
- **`[Install]`**: em qual target o serviço entra quando é habilitado.

{{% notice tip "Logs do serviço" %}}
Tudo o que o programa escreve na saída padrão vai para o journal:
`journalctl -u meuapp -f`. Veremos o journal em detalhes no capítulo de logs.
{{% /notice %}}
