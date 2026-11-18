+++
title = "CUPS e IPP"
weight = 1
description = "Instalação, IPP Everywhere, interface web e lpadmin."
+++

## Conceitos

- **CUPS** (*Common UNIX Printing System*): servidor de impressão que recebe os trabalhos,
  coloca em **filas**, converte para o formato da impressora e envia.
- **IPP** (*Internet Printing Protocol*): protocolo de impressão sobre HTTP, porta **631**.
- **IPP Everywhere / AirPrint / driverless**: impressoras modernas anunciam-se na rede
  (mDNS/DNS-SD) e informam suas capacidades — o CUPS as configura sem driver.

```bash
sudo apt install cups cups-browsed
sudo usermod -aG lpadmin $USER     # permite administrar impressoras sem root
```

## Interface web

Acesse **http://localhost:631**. Em *Administration → Add Printer* é possível adicionar
impressoras locais, de rede e compartilhadas, com login do usuário do grupo `lpadmin`.

## Pela linha de comando

```bash
lpinfo -v                     # dispositivos/URIs encontrados (USB, rede)
lpinfo --make-and-model '' -m | head   # drivers disponíveis

# impressora de rede sem driver (IPP Everywhere)
sudo lpadmin -p Escritorio -E -v ipp://192.168.1.50/ipp/print -m everywhere
sudo lpadmin -d Escritorio    # define como padrão

lpstat -p -d                  # impressoras e a padrão
sudo lpadmin -x Escritorio    # remove
```

## Impressoras USB

Ao conectar, a maioria é reconhecida automaticamente (pelo `ipp-usb`, que expõe a
impressora USB como se fosse IPP). Modelos antigos podem precisar de drivers:

```bash
sudo apt install printer-driver-all   # pacote com drivers de vários fabricantes
sudo apt install hplip                # HP
```

## Compartilhando e usando impressoras remotas

No servidor:

```bash
sudo cupsctl --share-printers --remote-any
sudo lpadmin -p Escritorio -o printer-is-shared=true
```

Nos clientes Linux, impressoras compartilhadas aparecem sozinhas (via `cups-browsed`).
Clientes Windows e macOS podem adicionar `http://servidor:631/printers/Escritorio`.

{{% notice note "Impressoras em servidores Windows" %}}
Para usar uma impressora compartilhada por um Windows (SMB), use a URI
`smb://usuario:senha@servidor/Impressora` (pacote `smbclient`). Se o Windows oferecer IPP,
prefira IPP.
{{% /notice %}}
