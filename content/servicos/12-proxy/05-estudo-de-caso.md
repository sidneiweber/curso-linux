+++
title = "Estudo de caso: política de acesso"
weight = 5
description = "Transformando uma política de uso da Internet da empresa em regras do Squid."
+++

Uma empresa definiu, por escrito, a seguinte **política de acesso à Internet**:

1. Somente os computadores da rede interna navegam.
2. Fora do horário de expediente, todos os computadores da rede interna navegam.
3. No expediente, só os computadores **autorizados** navegam.
4. Alguns computadores **nunca** navegam (bloqueados 24 h).
5. Navegadores obsoletos (Internet Explorer) são proibidos.
6. Exceto em alguns computadores que ainda precisam deles para um sistema legado.
7. Arquivos de vídeo, áudio e executáveis são proibidos.
8. Sites e palavras impróprios são proibidos.
9. Downloads acima de 5 MB são proibidos.
10. Os computadores da administração têm acesso total.

Os computadores são identificados pelo **MAC** (veja os limites em
[Controle de acesso: ACLs]({{% relref "02-acls" %}})).

{{% notice info "Créditos" %}}
Política adaptada do roteiro **"Instalando servidor Debian — Memento"**, de Taylor Lopes
(2006). O original usava Squid 2.5, `dhcp3-server` e iptables; aqui ele foi atualizado para
o Squid, o Kea e o nftables atuais do laboratório.
{{% /notice %}}

## As listas

Cada grupo de computadores ou de termos fica em um arquivo, um item por linha:

```bash
sudo mkdir /etc/squid/listas
cd /etc/squid/listas
sudo touch admin.txt autorizados.txt bloqueados.txt legado.txt sites.txt palavras.txt
```

| Arquivo | Conteúdo | Exemplo |
|---|---|---|
| `admin.txt` | MACs com acesso total (política 10) | `08:00:27:11:11:11` |
| `autorizados.txt` | MACs que navegam no expediente (3) | `08:00:27:aa:bb:cc` |
| `bloqueados.txt` | MACs que nunca navegam (4) | `08:00:27:44:44:44` |
| `legado.txt` | MACs que podem usar o navegador antigo (6) | `08:00:27:88:88:88` |
| `sites.txt` | domínios proibidos (8) | `.apostas-exemplo.com` |
| `palavras.txt` | palavras proibidas na URL (8) | `cassino` |

{{% notice tip %}}
Coloque os comentários em **linhas próprias** (`# pc da recepção`), acima de cada MAC.
Assim fica fácil saber de quem é cada endereço.
{{% /notice %}}

## As regras

No `squid.conf`, abaixo de `INSERT YOUR OWN RULE(S) HERE`:

```text
# --- Quem é quem ---
acl rede_interna src 192.168.56.0/24
acl admin        arp "/etc/squid/listas/admin.txt"
acl autorizados  arp "/etc/squid/listas/autorizados.txt"
acl bloqueados   arp "/etc/squid/listas/bloqueados.txt"
acl legado       arp "/etc/squid/listas/legado.txt"

# --- O quê ---
acl sites      dstdomain -i "/etc/squid/listas/sites.txt"
acl palavras   url_regex -i "/etc/squid/listas/palavras.txt"
acl midia      urlpath_regex -i \.(mp3|wav|wma|mp4|avi|mpe?g|mov|asf)$
acl executavel urlpath_regex -i \.(exe|msi|bat|scr|com|pps)$
acl ie         browser -i MSIE|Trident

# --- Quando (expediente com intervalo de almoço) ---
acl expediente time MTWHF 08:00-12:00
acl expediente time MTWHF 13:30-18:00

# 10. administração: acesso total
http_access allow admin

# 4. bloqueados: nunca navegam
http_access deny bloqueados

# 7 e 8. conteúdo proibido
http_access deny sites
http_access deny palavras
http_access deny midia
http_access deny executavel

# 5 e 6. navegador obsoleto, exceto nas máquinas do sistema legado
http_access deny ie !legado

# 3. no expediente, só os autorizados
http_access deny expediente !autorizados

# 1 e 2. o restante da rede interna navega
http_access allow rede_interna

# (o "http_access deny all" padrão nega todo o resto)

# 9. downloads de no máximo 5 MB, exceto para a administração
reply_body_max_size 5 MB !admin
```

```bash
sudo squid -k parse && sudo squid -k reconfigure
```

Repare em como a **ordem** implementa a política: a administração é liberada antes de
qualquer bloqueio; os bloqueios vêm antes da liberação geral. Basta uma regra fora de ordem
para a política inteira mudar.

## O que a política não alcança

Lembre da tabela [O que funciona com HTTPS]({{% relref "02-acls#o-que-funciona-com-https" %}}).
Com HTTPS, as políticas **7** e **9** e a parte de *palavras* da política **8** só valem para
sites HTTP. Uma política escrita hoje deveria ser revista com isso em mente:

- bloqueio por **domínio** e por **categoria** (listas prontas, como as do UT1 da
  Universidade de Toulouse) em vez de palavras na URL;
- antivírus e controle de execução **nos computadores**, em vez de bloquear `.exe` no proxy;
- **autenticação de usuários** em vez de (ou junto com) o MAC, para os relatórios mostrarem
  pessoas, não placas de rede.

{{% notice note "LGPD" %}}
Divulgue a política aos funcionários (por exemplo, na intranet) **antes** de aplicá-la e
registre quem tem acesso aos relatórios de navegação.
{{% /notice %}}

## Exercícios

1. Cadastre o MAC do `cli01` em `autorizados.txt`, mude a hora do `srv01` para dentro e para
   fora do expediente e confira o resultado no `access.log`.
2. Mova o `cli01` para `bloqueados.txt` e confirme que ele não navega em nenhum horário.
3. Inverta a ordem das regras `allow admin` e `deny bloqueados`, coloque o mesmo MAC nas duas
   listas e explique o que acontece.
4. Simule um navegador antigo com
   `curl -x http://srv01:3128 -A "Mozilla/4.0 (compatible; MSIE 6.0)" -I http://neverssl.com`.
