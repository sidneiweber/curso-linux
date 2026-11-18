+++
title = "Controle de acesso: ACLs"
weight = 2
description = "ACLs por rede, domínio, horário e expressões; ordem das regras."
+++

O controle do Squid tem duas partes:

1. **`acl`** define condições com nome;
2. **`http_access`** permite ou nega combinando ACLs.

As regras `http_access` são avaliadas **de cima para baixo** e **a primeira que casar
decide**. Na mesma linha, as ACLs são combinadas com **E**.

## Tipos de ACL

| Tipo | Exemplo | Casa com |
|---|---|---|
| `src` | `acl ti src 192.168.56.30-192.168.56.39` | IP do cliente |
| `dstdomain` | `acl redes_sociais dstdomain .facebook.com .instagram.com` | Domínio do site (o ponto inicial inclui subdomínios) |
| `dstdom_regex` | `acl apostas dstdom_regex -i bet` | Expressão regular no domínio |
| `time` | `acl expediente time MTWHF 08:00-18:00` | Dias (S M T W H F A) e horário |
| `port` | `acl SSL_ports port 443` | Porta de destino |
| `proxy_auth` | `acl autenticados proxy_auth REQUIRED` | Usuário autenticado |

## Exemplo completo

No `/etc/squid/squid.conf`, abaixo de `INSERT YOUR OWN RULE(S) HERE`:

```text
acl rede_interna src 192.168.56.0/24
acl ti src 192.168.56.30-192.168.56.39
acl expediente time MTWHF 08:00-18:00

acl bloqueados dstdomain "/etc/squid/bloqueados.txt"
acl redes_sociais dstdomain .facebook.com .instagram.com .tiktok.com .x.com

# a TI acessa tudo
http_access allow ti

# sites bloqueados para todos os demais
http_access deny bloqueados

# redes sociais só fora do expediente
http_access deny redes_sociais expediente

# o resto da rede interna pode navegar
http_access allow rede_interna

# (o "http_access deny all" que já existe logo abaixo nega todo o resto)
```

Página de erro em português (em qualquer ponto do arquivo):

```text
error_directory /usr/share/squid-langpack/pt-br
```

`/etc/squid/bloqueados.txt` (um por linha):

```text
.exemplo-malware.com
.jogos-online.com
```

```bash
sudo squid -k parse && sudo squid -k reconfigure   # recarrega sem derrubar conexões
```

{{% notice tip "Ordem é tudo" %}}
Se `http_access allow rede_interna` vier **antes** de `deny bloqueados`, ninguém é bloqueado
— a primeira regra já liberou. Coloque exceções e bloqueios antes das liberações gerais.
{{% /notice %}}
