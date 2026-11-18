+++
title = "Grupos"
weight = 3
description = "Grupo primário e secundários, /etc/group, groupadd e gerenciamento de membros."
+++

Grupos facilitam dar permissões para vários usuários de uma vez.

- **Grupo primário**: atribuído aos arquivos que o usuário cria (campo 4 do `/etc/passwd`);
- **Grupos secundários**: grupos extras dos quais o usuário faz parte.

## /etc/group

```text
desenvolvimento:x:1001:maria,joao
```

`nome : senha : GID : membros`

## Criando e removendo grupos

```bash
sudo groupadd desenvolvimento
sudo groupmod -n dev desenvolvimento   # renomeia
sudo groupdel dev
```

## Gerenciando membros

```bash
groups maria                              # grupos da maria
sudo usermod -aG desenvolvimento maria    # adiciona (sem o -a, substitui os grupos!)
sudo gpasswd -a joao desenvolvimento      # adiciona
sudo gpasswd -d joao desenvolvimento      # remove do grupo
sudo usermod -aG sudo maria               # dá permissão de administrador
```

{{% notice note %}}
O usuário precisa **fazer logout e login novamente** para que a mudança de grupos tenha efeito
(ou use `newgrp nome_do_grupo`).
{{% /notice %}}

## Exercício

Crie o grupo `financeiro`, os usuários `ana` e `bruno`, adicione os dois ao grupo e
confira o resultado.

{{% expand "Ver solução" %}}
```bash
sudo groupadd financeiro
sudo adduser ana
sudo adduser bruno
sudo usermod -aG financeiro ana
sudo usermod -aG financeiro bruno
getent group financeiro
```
{{% /expand %}}
