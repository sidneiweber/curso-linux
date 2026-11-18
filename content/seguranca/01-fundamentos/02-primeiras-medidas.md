+++
title = "Primeiras medidas em um servidor"
weight = 2
description = "Atualizações automáticas, serviços desnecessários, AppArmor e checklist."
+++

## Atualizações automáticas de segurança

```bash
sudo apt install unattended-upgrades apt-listchanges
sudo dpkg-reconfigure -plow unattended-upgrades
```

Por padrão, instala apenas atualizações de **segurança**. Ajustes em
`/etc/apt/apt.conf.d/50unattended-upgrades`:

```text
Unattended-Upgrade::Mail "root";
Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";
Unattended-Upgrade::Automatic-Reboot "true";
Unattended-Upgrade::Automatic-Reboot-Time "04:00";
```

```bash
sudo unattended-upgrade --dry-run --debug
less /var/log/unattended-upgrades/unattended-upgrades.log
```

## Reduzindo a superfície de ataque

```bash
sudo ss -tulpn                                  # o que está escutando na rede?
systemctl list-units --type=service --state=running
apt list --installed | wc -l
```

Para cada serviço: **ele é necessário?** Se não, remova (`apt purge`). Se sim, ele precisa
escutar em todas as interfaces, ou só em `127.0.0.1`?

## AppArmor

O Debian e o Ubuntu vêm com o **AppArmor** ativo: perfis limitam quais arquivos, redes e
recursos cada programa pode usar.

```bash
sudo aa-status                     # perfis carregados: enforce x complain
sudo apt install apparmor-utils apparmor-profiles apparmor-profiles-extra
sudo aa-enforce /etc/apparmor.d/usr.sbin.named
journalctl -k | grep -i apparmor   # bloqueios ("DENIED")
```

(Na família Red Hat, o equivalente é o **SELinux**.)

## Ajustes do kernel

`/etc/sysctl.d/90-seguranca.conf`:

```ini
# rede
net.ipv4.conf.all.rp_filter = 1                # descarta pacotes com origem falsificada
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.tcp_syncookies = 1                     # proteção contra SYN flood
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.conf.all.log_martians = 1

# sistema
kernel.kptr_restrict = 2
kernel.dmesg_restrict = 1
fs.protected_symlinks = 1
fs.protected_hardlinks = 1
```

```bash
sudo sysctl --system
```

(Se o servidor é roteador, como o `srv01`, mantenha `ip_forward = 1`.)

## Checklist inicial

- [ ] Sistema atualizado e `unattended-upgrades` ativo
- [ ] SSH só com chaves, sem root (capítulo de SSH de Serviços de Rede)
- [ ] Usuários com `sudo` revisados; nenhuma conta sem senha
- [ ] Firewall ativo, negando por padrão
- [ ] Serviços desnecessários removidos; os demais escutando só onde precisam
- [ ] fail2ban no SSH e em serviços expostos
- [ ] Logs persistentes e, de preferência, enviados para outro servidor
- [ ] Backup automático **e testado**
- [ ] Horário sincronizado (`timedatectl`) — essencial para correlacionar logs
