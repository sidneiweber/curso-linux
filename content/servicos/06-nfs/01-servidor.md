+++
title = "Servidor NFS"
weight = 1
description = "Conceitos, versões, /etc/exports e opções de exportação."
+++

## Conceitos

- O servidor **exporta** diretórios; os clientes os **montam**.
- O controle de acesso é feito por **endereço do cliente** e, no arquivo, pelas permissões
  **UID/GID** de sempre — o NFS confia no UID informado pelo cliente.
- **NFSv4** (padrão atual) usa apenas a porta **2049/TCP**, o que simplifica o firewall.
  As versões 2/3 dependiam do `rpcbind` e de várias portas.

{{% notice warning "UIDs iguais" %}}
Como o NFS compara números, o usuário `maria` precisa ter o **mesmo UID** no servidor e nos
clientes — ou ela verá arquivos de outra pessoa. Em redes maiores, isso é resolvido
centralizando os usuários no **LDAP** (capítulo 8). Para segurança de verdade, NFS com
**Kerberos** (`sec=krb5p`).
{{% /notice %}}

## Instalação

```bash
sudo apt install nfs-kernel-server
```

## Exportando diretórios

```bash
sudo mkdir -p /srv/nfs/publico /srv/nfs/home
sudo chown nobody:nogroup /srv/nfs/publico
sudo chmod 1777 /srv/nfs/publico
```

`/etc/exports`:

```text
/srv/nfs/publico   192.168.56.0/24(rw,sync,no_subtree_check)
/srv/nfs/home      192.168.56.0/24(rw,sync,no_subtree_check,root_squash)
/srv/nfs/iso       192.168.56.20(ro,sync,no_subtree_check)
```

| Opção | Significado |
|---|---|
| `rw` / `ro` | Leitura e escrita / somente leitura |
| `sync` | Confirma a escrita só depois de gravar no disco (seguro; `async` é mais rápido e arriscado) |
| `root_squash` | O root do cliente vira `nobody` no servidor (**padrão** — mantenha) |
| `no_root_squash` | O root do cliente é root no servidor — perigoso |
| `all_squash` | Todos os usuários viram `nobody` (bom para pastas públicas) |
| `no_subtree_check` | Desativa uma verificação que causa problemas; recomendado |

{{% notice warning "Espaço importa" %}}
`192.168.56.20(rw)` e `192.168.56.20 (rw)` são **diferentes**: com espaço, a opção `rw` vale
para o **mundo inteiro**, e o host listado recebe as opções padrão.
{{% /notice %}}

```bash
sudo exportfs -ra        # aplica /etc/exports
sudo exportfs -v         # mostra o que está exportado e com quais opções
sudo systemctl enable --now nfs-server
```
