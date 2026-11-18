+++
title = "Entendendo o kernel"
weight = 1
description = "Versões, pacotes do kernel e como atualizá-lo."
+++

O kernel gerencia CPU, memória, dispositivos, sistemas de arquivos e rede, e oferece aos
programas as **chamadas de sistema** (*syscalls*). Programas nunca falam com o hardware
diretamente — sempre pedem ao kernel.

```bash
uname -r        # versão: 6.12.48-amd64
uname -a        # tudo: versão, arquitetura, data de compilação
cat /proc/version
```

## Versões

Novas versões saem a cada 9–10 semanas em [kernel.org](https://kernel.org). Algumas são
**LTS** (*long-term support*) e recebem correções por anos — são as que as distribuições
preferem. Uma versão `6.12.48` significa: série 6.12, 48ª atualização de correções.

## Kernel no Debian/Ubuntu

O kernel é um pacote como qualquer outro:

```bash
dpkg -l 'linux-image*' | grep ^ii    # kernels instalados
ls /boot/vmlinuz-*
sudo apt install linux-image-amd64   # metapacote: sempre o kernel mais recente da versão
```

As atualizações instalam o novo kernel **ao lado** do antigo; o GRUB passa a iniciar o novo
e mantém o anterior no submenu *Advanced options* — se algo der errado, basta escolher o
antigo. O `apt autoremove` remove kernels antigos, mantendo o atual e o anterior.

{{% notice tip "Precisa reiniciar?" %}}
O kernel novo só passa a valer após reiniciar. Para saber se há atualização pendente:
`ls /var/run/reboot-required` (Ubuntu) ou compare `uname -r` com o kernel mais novo em `/boot`.
O `needrestart` (`sudo apt install needrestart`) avisa também sobre serviços que precisam
ser reiniciados.
{{% /notice %}}

## Kernels alternativos

Ubuntu oferece kernels HWE (*hardware enablement*) mais novos para hardware recente; Debian
oferece kernels mais novos via **backports**:

```bash
sudo apt install -t trixie-backports linux-image-amd64
```
