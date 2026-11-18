+++
title = "Imprimindo e gerenciando filas"
weight = 2
description = "lp, lpstat, cancel, opções de impressão e logs."
+++

## Imprimindo

```bash
lp documento.pdf                       # na impressora padrão
lp -d Escritorio -n 2 documento.pdf    # 2 cópias em outra impressora
lp -o sides=two-sided-long-edge -o page-ranges=1-4 relatorio.pdf
lp -o media=A4 -o fit-to-page imagem.png
lpoptions -p Escritorio -l             # opções suportadas pela impressora
```

## Fila

```bash
lpstat -o                  # trabalhos na fila
lpstat -t                  # tudo: impressoras, filas, trabalhos
cancel 42                  # cancela o trabalho 42
cancel -a Escritorio       # cancela todos da impressora
```

## Pausar e retomar

```bash
sudo cupsdisable Escritorio   # para de imprimir (a fila continua aceitando trabalhos)
sudo cupsenable Escritorio
sudo cupsreject Escritorio    # recusa novos trabalhos
sudo cupsaccept Escritorio
```

## Quando nada sai

1. `lpstat -t` — a impressora está *disabled*? Use `cupsenable`.
2. `systemctl status cups`
3. Logs: `/var/log/cups/error_log` (detalhe com `sudo cupsctl --debug-logging`).
4. A impressora responde na rede? `ping` e `ipptool -tv ipp://IP/ipp/print get-printer-attributes.test`.

## Exercícios

1. Instale o CUPS, adicione a impressora virtual de PDF (`sudo apt install printer-driver-cups-pdf`)
   e imprima um arquivo de texto com `lp`.
2. Pause a fila, envie três trabalhos, liste-os, cancele um e retome a fila.
