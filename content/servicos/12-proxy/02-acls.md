+++
title = "Controle de acesso: ACLs"
weight = 2
description = "ACLs por rede, MAC, domínio, horário, expressões e navegador; ordem das regras."
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
| `arp` | `acl diretoria arp 08:00:27:aa:bb:cc` | Endereço MAC do cliente |
| `url_regex` | `acl palavras url_regex -i aposta cassino` | Expressão regular na URL inteira |
| `urlpath_regex` | `acl executaveis urlpath_regex -i \.(exe\|msi\|bat)$` | Expressão regular só no caminho (depois do domínio) |
| `browser` | `acl ie browser -i MSIE\|Trident` | Cabeçalho *User-Agent* do navegador |

Várias linhas `acl` com o **mesmo nome** se somam (**OU**), e o `!` antes de uma ACL
inverte a condição: `http_access deny bloqueados !ti` nega os bloqueados, exceto para a TI.

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

## Controle por MAC

A ACL `arp` identifica o computador pela **placa de rede**, não pelo IP. Isso resolve o
usuário que troca o próprio IP para fugir de uma regra:

```text
acl diretoria arp "/etc/squid/listas/diretoria.txt"
```

```text
# /etc/squid/listas/diretoria.txt: um MAC por linha
# pc da diretora
08:00:27:aa:bb:cc
```

{{% notice warning "Limites do controle por MAC" %}}
- Só funciona quando o cliente está no **mesmo segmento de rede** do Squid. Se houver um
  roteador no meio, o Squid só enxerga o MAC do roteador.
- O MAC também pode ser trocado (`ip link set enp0s3 address ...`). Ele dificulta, mas não
  substitui a autenticação de usuários (próxima página).
- Com as reservas do DHCP (capítulo 3), o IP de cada máquina já fica amarrado ao MAC, e uma
  ACL `src` por IP costuma bastar.
{{% /notice %}}

## Limite de tamanho de download

`reply_body_max_size` não é uma regra `http_access`: ele limita o tamanho das respostas para
os clientes que casarem com as ACLs da linha:

```text
reply_body_max_size 50 MB !ti     # quem não é da TI baixa no máximo 50 MB
```

## O que funciona com HTTPS

Com HTTPS, o Squid só vê o **domínio** e a porta (método `CONNECT`). Por isso:

| ACL / diretiva | HTTP | HTTPS (sem SSL Bump) |
|---|---|---|
| `src`, `arp`, `time`, `dstdomain`, `dstdom_regex` | ✔ | ✔ |
| `url_regex` | URL inteira | só `dominio:443` |
| `urlpath_regex` | ✔ | ✘ (o caminho é cifrado) |
| `browser` | ✔ | ✔ (os navegadores enviam o *User-Agent* no `CONNECT`) |
| `reply_body_max_size` | ✔ | ✘ (o conteúdo é cifrado) |

Bloquear "arquivos `.exe`" ou "downloads acima de 50 MB" só tem efeito em sites HTTP. Na
prática, o controle hoje é feito principalmente por **domínio**.

{{% notice note %}}
O *User-Agent* é informado pelo próprio navegador e pode ser alterado com facilidade. Use a
ACL `browser` para orientar os usuários, não como medida de segurança.
{{% /notice %}}
