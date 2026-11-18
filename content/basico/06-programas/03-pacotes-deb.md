+++
title = "E os programas .exe?"
weight = 3
description = "Pacotes .deb baixados manualmente e programas do Windows."
+++

Arquivos `.exe` são executáveis do **Windows** e **não rodam nativamente** no Linux.
O equivalente mais próximo, nas distros Debian/Ubuntu, é o pacote **`.deb`**.

## Instalando um .deb baixado

Alguns programas (Google Chrome, Zoom, etc.) oferecem um `.deb` para download:

```bash
sudo apt install ./google-chrome-stable_current_amd64.deb
```

{{% notice info %}}
Prefira `apt install ./arquivo.deb` em vez de `dpkg -i arquivo.deb`: o `apt` resolve
as dependências automaticamente.
{{% /notice %}}

## E se eu realmente precisar de um programa do Windows?

1. Procure uma **alternativa livre** (ex.: LibreOffice, GIMP, Inkscape) — o site
   [alternativeto.net](https://alternativeto.net) ajuda;
2. Use o **Wine** (ou ferramentas baseadas nele, como Bottles e Proton/Steam) para executar
   alguns programas `.exe`;
3. Use uma **máquina virtual** com Windows.

{{% notice warning %}}
Instale programas apenas de **fontes confiáveis**. Pacotes de terceiros rodam com
permissão de administrador durante a instalação.
{{% /notice %}}
