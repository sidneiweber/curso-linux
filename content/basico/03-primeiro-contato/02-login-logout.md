+++
title = "Login e logout"
weight = 2
description = "Entrando e saindo do sistema, desligando e reiniciando."
+++

- **Login** é a entrada no sistema: você informa seu **usuário** e **senha**.
- **Logout** é a saída do sistema.

{{% notice note %}}
Ao digitar a senha no terminal **nada aparece na tela** — nem asteriscos. Isso é normal!
{{% /notice %}}

## Saindo do sistema

No modo texto, a saída pode ser feita por:

```bash
logout     # encerra o shell de login
exit       # encerra o shell atual
```

ou pelo atalho `Ctrl` + `D`.

## Desligando e reiniciando

```bash
sudo poweroff          # desliga
sudo reboot            # reinicia
sudo shutdown -h now   # desliga agora
sudo shutdown -r +10   # reinicia daqui a 10 minutos
```
