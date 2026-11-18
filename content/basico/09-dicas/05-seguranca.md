+++
title = "Dicas para servidores e segurança"
weight = 5
description = "Permissões de webserver, ping, sysctl, SSH, serviços, firewall e manutenção."
+++

{{% notice warning %}}
Estas dicas alteram configurações do sistema. Teste em uma máquina virtual antes de
aplicar em um servidor de produção.
{{% /notice %}}

## Permissões em um servidor web

| Item | Permissão | Octal |
|---|---|---|
| Diretórios | `rwxr-xr-x` | `755` |
| Arquivos HTML, CSS, imagens | `rw-r--r--` | `644` |
| Scripts CGI | `rwxr-xr-x` | `755` |

```bash
sudo chown -R www-data:www-data /var/www/site
sudo find /var/www/site -type d -exec chmod 755 {} \;
sudo find /var/www/site -type f -exec chmod 644 {} \;
```

{{% notice note %}}
No Debian/Ubuntu o servidor web roda com o usuário `www-data` (antigamente, `nobody`).
{{% /notice %}}

## Parâmetros do kernel com sysctl

O `sysctl` altera, em tempo de execução, os parâmetros em `/proc/sys`. Para tornar a
mudança permanente, crie um arquivo em `/etc/sysctl.d/`.

### Ignorar ping

```bash
sudo sysctl -w net.ipv4.icmp_echo_ignore_all=1   # ignora
sudo sysctl -w net.ipv4.icmp_echo_ignore_all=0   # volta a responder
```

### Registrar pacotes estranhos e desativar *source routing*

Pacotes "marcianos" (com endereços forjados/impossíveis) podem ser registrados no log.
Já o *IP source routing* permite que o remetente defina o caminho do pacote, o que pode
ser usado para interceptar tráfego — o ideal é desativá-lo.

```bash
sudo tee /etc/sysctl.d/90-seguranca.conf << 'FIM'
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0
FIM
sudo sysctl --system     # aplica todos os arquivos de configuração
```

## Acesso remoto do root

Antigamente se editava `/etc/securetty` para impedir o root de entrar via **Telnet**
(protocolo sem criptografia, que **não deve mais ser usado**). Hoje o acesso remoto é
feito por **SSH**; para bloquear o login direto do root, edite `/etc/ssh/sshd_config`:

```text
PermitRootLogin no
```

```bash
sudo systemctl restart ssh
```

## Desabilitar serviços não utilizados

Menos serviços rodando = menos pontos de ataque. Antigamente isso era feito comentando
linhas em `/etc/inetd.conf`; hoje usamos o `systemd`:

```bash
systemctl list-units --type=service --state=running   # serviços em execução
sudo ss -tulpn                                        # o que está escutando na rede
sudo systemctl disable --now cups                     # desativa e para um serviço
```

Remova também contas desnecessárias, como a do FTP anônimo, se existir:
`sudo userdel -r ftp`.

## Firewall (sucessor do TCP Wrappers)

Os arquivos `/etc/hosts.allow` e `/etc/hosts.deny` (TCP Wrappers) foram descontinuados na
maioria das distribuições. A ideia continua a mesma — **negar tudo e liberar apenas o
necessário** — mas agora com um firewall:

```bash
sudo apt install ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow from 192.168.0.0/24 to any port 22   # SSH só da rede local
sudo ufw allow 80,443/tcp                            # site
sudo ufw enable
sudo ufw status verbose
```

Para ir além: **fail2ban** (bloqueia tentativas repetidas de login), **AIDE** ou
**Tripwire** (detectam alterações em arquivos do sistema) e monitoramento de logs com
`journalctl`.

## Proteger o modo de recuperação (single user)

Com acesso físico, qualquer pessoa poderia iniciar o sistema em modo de manutenção.
Nas distribuições com systemd, o modo de recuperação já **pede a senha do root**
(antes era preciso configurar `sulogin` no `/etc/inittab`). Para proteger também a
edição das opções de boot, defina uma **senha no GRUB**.

## Esconder a versão do Apache

Por padrão o Apache revela sua versão nos cabeçalhos e páginas de erro. No Debian/Ubuntu,
edite `/etc/apache2/conf-available/security.conf`:

```apacheconf
ServerTokens Prod
ServerSignature Off
```

| `ServerTokens` | Cabeçalho exibido |
|---|---|
| `Prod` | `Apache` |
| `Major` | `Apache/2` |
| `Minor` | `Apache/2.4` |
| `Min` | `Apache/2.4.62` |
| `OS` | `Apache/2.4.62 (Debian)` |
| `Full` (padrão) | `Apache/2.4.62 (Debian) PHP/8.2 ...` |

```bash
sudo systemctl reload apache2
```

## Impedir logins durante a manutenção

Crie o arquivo `/etc/nologin` com uma mensagem. Usuários comuns (não-root) não
conseguirão entrar e verão o aviso:

```bash
echo "Servidor em manutenção. Volte às 18h." | sudo tee /etc/nologin
# ... manutenção ...
sudo rm /etc/nologin
```

## Em caso de invasão

1. **Isole** a máquina da rede, mas não a desligue de imediato (a memória contém evidências);
2. **Preserve os logs** (`/var/log`, `journalctl`);
3. Descubra o responsável pelo IP de origem com `whois <IP>` e procure o contato de
   **abuse** (normalmente `abuse@provedor`);
4. Reinstale o sistema a partir de uma mídia confiável e restaure os dados de backup.
