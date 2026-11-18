+++
title = "Configurando o servidor"
weight = 2
description = "sshd_config, desativando senhas e root, controle de acesso."
+++

A configuração do servidor fica em `/etc/ssh/sshd_config`. No Debian, prefira criar
arquivos em `/etc/ssh/sshd_config.d/`, que são lidos primeiro e sobrevivem às atualizações.

`/etc/ssh/sshd_config.d/10-curso.conf`:

```text
# Autenticação
PermitRootLogin no
PasswordAuthentication no
KbdInteractiveAuthentication no
PubkeyAuthentication yes

# Quem pode entrar
AllowGroups ssh-users

# Sessão
MaxAuthTries 3
LoginGraceTime 30
ClientAliveInterval 300
ClientAliveCountMax 2
X11Forwarding no
```

```bash
sudo groupadd ssh-users
sudo usermod -aG ssh-users maria
sudo sshd -t                     # valida a configuração
sudo systemctl reload ssh
```

{{% notice warning "Não se tranque para fora" %}}
Antes de desligar `PasswordAuthentication`, confirme que o login **por chave** funciona.
Mantenha uma sessão SSH aberta enquanto testa em outra janela — se algo der errado, você
ainda pode desfazer.
{{% /notice %}}

## Principais opções

| Opção | Efeito |
|---|---|
| `Port 22` | Porta. Trocar reduz ruído nos logs, mas **não** é segurança de verdade |
| `ListenAddress 192.168.56.10` | Escuta só nessa interface |
| `PermitRootLogin no` / `prohibit-password` | Bloqueia root / permite só com chave |
| `PasswordAuthentication no` | Só chaves |
| `AllowUsers` / `AllowGroups` / `DenyUsers` | Lista de quem pode entrar |
| `Match` | Regras condicionais (por usuário, grupo ou endereço) |

Exemplo com `Match`: senha permitida só a partir da rede interna:

```text
Match Address 192.168.56.0/24
    PasswordAuthentication yes
```

## Monitorando

```bash
journalctl -u ssh -f
journalctl -u ssh | grep -E 'Accepted|Failed'
who                              # quem está conectado agora
```

Combine com o **fail2ban** (visto no capítulo de logs do Linux Avançado) para bloquear
tentativas de força bruta.
