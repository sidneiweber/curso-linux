+++
title = "Instalando o sistema"
weight = 1
description = "Preparação, instalação em máquina virtual, criação de usuário e GRUB."
+++

## Preparação

1. Baixe a imagem ISO da distribuição (ex.: [Ubuntu](https://ubuntu.com/download/desktop) ou [Debian](https://www.debian.org/download)).
2. Escolha onde instalar:
   - **Máquina virtual** (recomendado para o curso): VirtualBox, GNOME Boxes, VMware, virt-manager;
   - **Pendrive bootável** para instalar em um computador real (use Balena Etcher, Ventoy ou `dd`).
3. Requisitos sugeridos para a VM: 2 CPUs, 4 GB de RAM, 25 GB de disco.

{{% notice warning %}}
Ao instalar em um computador real, **faça backup** dos seus dados. O particionamento
pode apagar o conteúdo do disco.
{{% /notice %}}

## Passo a passo (visão geral)

1. Inicie o computador/VM pela ISO;
2. Escolha **idioma**, **layout de teclado** (ex.: *Português Brasil ABNT2*) e **fuso horário**;
3. Configure a **rede** (opcional durante a instalação);
4. **Particionamento**: para iniciantes, use a opção *"Apagar disco e instalar"*;
5. Confirme e aguarde a cópia dos arquivos.

## Finalizando a instalação

### Criação do usuário

O instalador pede um **nome de usuário** e uma **senha**. Esse será seu usuário comum
do dia a dia. Tarefas administrativas são feitas com `sudo` (Ubuntu) ou com o usuário
**root** (Debian, se você definir senha para ele).

### GRUB

O **GRUB** é o *gerenciador de inicialização* (*bootloader*): o primeiro programa carregado
após a BIOS/UEFI. Ele carrega o kernel Linux e permite escolher entre sistemas
operacionais quando há mais de um instalado (*dual boot*).

Ao final, remova a mídia de instalação e reinicie. 🎉
