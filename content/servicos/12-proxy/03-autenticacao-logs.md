+++
title = "Autenticação e relatórios"
weight = 3
description = "Login no proxy (arquivo ou LDAP), access.log e relatórios de acesso."
+++

## Autenticação básica

Com autenticação, os logs mostram **quem** acessou, não só o IP.

```bash
sudo apt install apache2-utils
sudo htpasswd -c /etc/squid/usuarios maria
sudo htpasswd /etc/squid/usuarios joao
```

No `squid.conf`, **antes** das suas ACLs:

```text
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/usuarios
auth_param basic realm Proxy do Curso
auth_param basic credentialsttl 8 hours

acl autenticados proxy_auth REQUIRED
```

E troque a liberação geral:

```text
http_access allow rede_interna autenticados
```

## Autenticação no LDAP

Para usar as contas do capítulo 8:

```text
auth_param basic program /usr/lib/squid/basic_ldap_auth -b "ou=pessoas,dc=curso,dc=lan" -f "uid=%s" -h srv01.curso.lan
```

Em redes com Active Directory, usa-se **Kerberos/NTLM** (`negotiate_kerberos_auth`) para
login transparente com a conta do Windows.

{{% notice warning "Senha em texto puro" %}}
Na autenticação *basic*, a senha vai do navegador ao proxy codificada, mas **sem
criptografia**. Mantenha o proxy acessível só pela rede interna — ou use Kerberos.
{{% /notice %}}

## access.log

```bash
sudo tail -f /var/log/squid/access.log
```

```text
1763466932.512    85 192.168.56.20 TCP_TUNNEL/200 6211 CONNECT debian.org:443 maria HIER_DIRECT/151.101.2.132 -
1763466940.004     0 192.168.56.20 TCP_DENIED/403 3954 CONNECT www.facebook.com:443 maria HIER_NONE/- text/html
```

Campos: data (Unix), duração (ms), cliente, resultado/código, bytes, método, destino, usuário.

```bash
# sites mais acessados
sudo awk '{print $7}' /var/log/squid/access.log | sed 's/:443$//' | sort | uniq -c | sort -rn | head
# bloqueios por usuário
sudo awk '$4 ~ /DENIED/ {print $8}' /var/log/squid/access.log | sort | uniq -c | sort -rn
```

## Relatórios

O clássico **SARG** gerava relatórios HTML a partir do `access.log`, mas está sem
manutenção. Alternativas atuais:

- **SquidAnalyzer** — relatórios HTML por usuário, site e período;
- enviar o `access.log` para **Grafana Loki** ou **OpenSearch** e montar painéis
  (veja o capítulo de logs do Linux Avançado).

{{% notice note "Privacidade" %}}
Registros de navegação são **dados pessoais** (LGPD). Defina por escrito a política de uso,
quem pode consultar os relatórios e por quanto tempo os logs são guardados.
{{% /notice %}}

## Exercícios

1. Configure o Squid para a rede interna e o `cli01` para navegar por ele.
2. Bloqueie uma lista de domínios para todos, exceto para o IP da TI.
3. Libere redes sociais apenas fora do horário comercial e teste mudando a hora da VM.
4. Ative autenticação no LDAP e confira o nome do usuário no `access.log`.
