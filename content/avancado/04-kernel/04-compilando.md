+++
title = "Compilando um kernel"
weight = 4
description = "Baixando o código, configurando, compilando e instalando como pacote .deb."
+++

No dia a dia **não é necessário** compilar o kernel — o da distribuição é bem testado e
recebe correções de segurança. Compilar faz sentido para estudar, testar um recurso novo ou
aplicar um patch específico. Faça isso em uma VM.

## 1. Dependências e código-fonte

```bash
sudo apt install build-essential bc bison flex libssl-dev libelf-dev \
  libncurses-dev dwarves rsync debhelper
cd ~/src
wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.12.48.tar.xz
tar xf linux-6.12.48.tar.xz && cd linux-6.12.48
```

## 2. Configuração

O kernel tem milhares de opções. Comece pela configuração do kernel atual:

```bash
cp /boot/config-$(uname -r) .config
make olddefconfig        # aceita o padrão para as opções novas
make localmodconfig      # opcional: só módulos em uso agora (compila MUITO mais rápido)
make menuconfig          # ajustes interativos
```

No Debian, desative a exigência de chaves de assinatura da distribuição:

```bash
scripts/config --disable SYSTEM_TRUSTED_KEYS --disable SYSTEM_REVOCATION_KEYS
```

## 3. Compilação como pacote .deb

```bash
make -j"$(nproc)" bindeb-pkg LOCALVERSION=-curso
ls ../*.deb
```

Gerar pacotes é melhor que `make install`: o `apt`/`dpkg` passa a saber da existência do
kernel e você pode removê-lo depois de forma limpa.

## 4. Instalação

```bash
sudo apt install ../linux-image-6.12.48-curso_*.deb
sudo reboot
uname -r                 # 6.12.48-curso
```

O kernel antigo continua no GRUB, em *Advanced options*.

## Aplicando um patch

```bash
cd linux-6.12.48
patch -p1 --dry-run < ../correcao.patch   # testa sem alterar nada
patch -p1 < ../correcao.patch
```

## Removendo

```bash
sudo apt remove linux-image-6.12.48-curso
```

## Exercícios

1. Descubra qual módulo é o driver da sua placa de rede (`lspci -k`) e veja seus parâmetros
   com `modinfo`.
2. Habilite `net.ipv4.ip_forward` de forma permanente e confirme após reiniciar.
3. Compile um kernel com `localmodconfig` e `LOCALVERSION=-teste` em uma VM e inicie por ele.
