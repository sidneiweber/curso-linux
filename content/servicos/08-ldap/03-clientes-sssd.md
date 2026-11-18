+++
title = "Login nas estações com SSSD"
weight = 3
description = "Configurando clientes Linux para autenticar no LDAP."
+++

O **SSSD** conecta o Linux ao diretório: os comandos (`id`, `ls -l`, `login`, `ssh`) passam a
enxergar os usuários do LDAP, com cache para funcionar mesmo se o servidor cair.

## No cliente

```bash
sudo apt install sssd-ldap ldap-utils libnss-sss libpam-sss
```

`/etc/sssd/sssd.conf`:

```ini
[sssd]
services = nss, pam
domains = curso.lan

[domain/curso.lan]
id_provider = ldap
auth_provider = ldap
ldap_uri = ldap://srv01.curso.lan
ldap_search_base = dc=curso,dc=lan
ldap_id_use_start_tls = true
ldap_tls_reqcert = demand
ldap_tls_cacert = /etc/ssl/certs/ca-curso.pem
cache_credentials = true
enumerate = false
```

```bash
sudo chmod 600 /etc/sssd/sssd.conf
sudo systemctl restart sssd
sudo pam-auth-update --enable mkhomedir     # cria o home no primeiro login
```

{{% notice warning "TLS no laboratório" %}}
O SSSD **exige** TLS para autenticar com senha. Se ainda não configurou TLS no servidor,
para testar no laboratório use temporariamente `ldap_id_use_start_tls = false` e
`ldap_auth_disable_tls_never_use_in_production = true` — **nunca** em produção.
{{% /notice %}}

## Testando

```bash
getent passwd maria          # o usuário aparece sem existir em /etc/passwd
id maria
su - maria                   # cria /home/maria no primeiro login
ssh maria@cli01
```

## Diagnóstico

```bash
sudo sssctl domain-status curso.lan
sudo sss_cache -E            # limpa o cache (após alterar algo no LDAP)
sudo journalctl -u sssd
# mais detalhes: debug_level = 6 na seção [domain/...] e veja /var/log/sssd/
```

## Gerenciamento centralizado

Com usuários no LDAP:

- os **UIDs são os mesmos** em todas as máquinas — resolvendo o problema do NFS;
- o **Samba** pode usar o mesmo diretório (backend `ldapsam`) ou, melhor, o Samba AD;
- serviços como **Apache**, **Postfix/Dovecot**, **Squid** e **Nextcloud** podem autenticar
  no LDAP.

## Exercícios

1. Crie as OUs, o grupo `funcionarios` e dois usuários no LDAP.
2. Configure o SSSD no `cli01` e faça login com um usuário que só existe no LDAP.
3. Exporte `/home` por NFS do `srv01` e monte-o no `cli01`: o usuário do LDAP deve ver os
   mesmos arquivos nas duas máquinas.
