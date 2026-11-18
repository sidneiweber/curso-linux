+++
title = "Discos e sistemas de arquivos"
type = "chapter"
weight = 2
+++

### Capítulo 2

# Discos e sistemas de arquivos

Particionar, formatar, montar e manter discos — incluindo swap, LVM e cotas.
No [Linux Básico]({{% relref "/basico/02-instalacao/04-discos" %}}) vimos como os discos
são nomeados; agora vamos mexer neles.

{{% notice warning "Cuidado" %}}
Os comandos deste capítulo **apagam dados** se usados no disco errado. Pratique em uma VM
com um **segundo disco virtual** (ex.: `/dev/vdb` ou `/dev/sdb`) e confira sempre com `lsblk`.
{{% /notice %}}

{{% children description="true" %}}
