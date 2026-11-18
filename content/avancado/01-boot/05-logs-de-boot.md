+++
title = "Verificando o boot"
weight = 5
description = "dmesg, journalctl -b e como investigar um boot com problemas."
+++

## Mensagens do kernel: dmesg

O kernel guarda suas mensagens (detecção de hardware, drivers, erros de disco) em um
buffer na memória:

```bash
sudo dmesg -T | less        # -T mostra data/hora legível
sudo dmesg -l err,warn      # só erros e avisos
sudo dmesg -w               # acompanha em tempo real (plugue um pendrive e veja)
```

## Logs do boot com journalctl

```bash
journalctl -b               # tudo desde o boot atual
journalctl -b -1            # boot anterior (útil depois de um travamento)
journalctl --list-boots     # boots registrados
journalctl -b -p err        # só mensagens de erro do boot atual
journalctl -k               # só mensagens do kernel (como o dmesg)
```

{{% notice note "Boots anteriores" %}}
Para consultar boots anteriores, o journal precisa ser **persistente** (gravado em
`/var/log/journal`). Se `journalctl --list-boots` mostrar só o boot atual, ative:
`sudo mkdir -p /var/log/journal && sudo systemctl restart systemd-journald`.
{{% /notice %}}

## Roteiro de investigação

1. **Algum serviço falhou?** `systemctl --failed`
2. **Por quê?** `systemctl status <serviço>` e `journalctl -u <serviço> -b`
3. **Hardware/driver?** `journalctl -k -p warn`
4. **Demorou demais?** `systemd-analyze blame` e `systemd-analyze critical-chain`
5. **Nem chega a subir?** Inicie com `systemd.unit=rescue.target` e consulte
   `journalctl -b -1 -p err` — erros no `/etc/fstab` são a causa mais comum.

## Exercícios

1. Descubra se a sua máquina usa UEFI ou BIOS e qual é o target padrão.
2. Liste os três serviços que mais atrasam o boot.
3. Mude o target padrão para `multi-user.target`, reinicie, faça login no modo texto e
   volte para o modo gráfico com `systemctl isolate graphical.target`.
4. Transforme um script seu em serviço com `Restart=on-failure`, mate o processo com
   `kill -9` e confirme no `systemctl status` que ele voltou sozinho.
