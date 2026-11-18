+++
title = "Criando e gerenciando usuários"
weight = 2
description = "adduser, useradd, passwd, usermod e userdel."
+++

{{< tabs >}}
{{% tab title="adduser (interativo)" %}}
Disponível no Debian/Ubuntu; faz perguntas e já cria o diretório pessoal.
```bash
sudo adduser joao
```
{{% /tab %}}
{{% tab title="useradd (todas as distros)" %}}
```bash
sudo useradd -m -s /bin/bash -c "João Souza" joao
sudo passwd joao
```
- `-m` cria o diretório pessoal
- `-s` define o shell
- `-c` comentário (nome completo)
{{% /tab %}}
{{< /tabs >}}

## Senhas

```bash
passwd                 # altera a sua senha
sudo passwd joao       # define a senha de outro usuário
sudo passwd -l joao    # bloqueia a conta
sudo passwd -u joao    # desbloqueia
sudo chage -l joao     # informações de expiração da senha
```

## Modificando usuários

```bash
sudo usermod -c "João P. Souza" joao   # altera o comentário
sudo usermod -s /bin/zsh joao          # altera o shell
sudo usermod -l joaop joao             # renomeia o login
```

## Removendo usuários

```bash
sudo userdel joao        # remove o usuário, mantém o /home
sudo userdel -r joao     # remove também o diretório pessoal
```

## Trocando de usuário

```bash
su - joao       # vira o usuário joao (pede a senha dele)
sudo -i         # abre um shell como root
exit            # volta ao usuário anterior
```

## Modo gráfico

Nos ambientes de desktop, use **Configurações → Usuários** para criar e gerenciar contas.
