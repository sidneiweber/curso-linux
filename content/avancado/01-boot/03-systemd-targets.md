+++
title = "Targets (os antigos runlevels)"
weight = 3
description = "Targets do systemd, alvo padrão e modos de recuperação."
+++

No SysV init, o sistema funcionava em **níveis de execução** (*runlevels*). No systemd o
equivalente são os **targets**: grupos de units que representam um estado do sistema.

| Runlevel | Target | Estado |
|---|---|---|
| 0 | `poweroff.target` | Desligar |
| 1 | `rescue.target` | Monousuário, manutenção |
| 3 | `multi-user.target` | Multiusuário, modo texto, com rede |
| 5 | `graphical.target` | Multiusuário com interface gráfica |
| 6 | `reboot.target` | Reiniciar |
| — | `emergency.target` | Shell mínimo, raiz somente leitura |

## Consultando e mudando o target

```bash
systemctl get-default                        # target padrão no boot
sudo systemctl set-default multi-user.target # servidor: boot em modo texto
sudo systemctl set-default graphical.target  # volta ao modo gráfico

sudo systemctl isolate multi-user.target     # muda AGORA, sem reiniciar
systemctl list-units --type=target           # targets ativos
systemctl list-dependencies graphical.target # o que compõe um target
```

## Rescue e emergency

- **`rescue.target`**: monta os sistemas de arquivos, sobe o mínimo e pede a senha de root.
  Use para consertar configurações quebradas.
- **`emergency.target`**: ainda mais enxuto — a raiz é montada somente leitura e nada mais
  sobe. Use quando o próprio `/etc/fstab` está impedindo o boot.

Os dois podem ser escolhidos no GRUB acrescentando `systemd.unit=rescue.target` (ou
`emergency.target`) à linha `linux`.

## Desligar e reiniciar

```bash
sudo systemctl poweroff     # = poweroff, shutdown -h now
sudo systemctl reboot       # = reboot
sudo shutdown -h +10 "Manutenção em 10 minutos"   # agenda e avisa os usuários
sudo shutdown -c            # cancela
```
