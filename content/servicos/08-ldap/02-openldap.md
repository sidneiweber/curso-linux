+++
title = "Servidor OpenLDAP"
weight = 2
description = "Instalação do slapd, estrutura inicial, usuários, grupos e consultas."
+++

## Instalação

```bash
sudo apt install slapd ldap-utils
sudo dpkg-reconfigure slapd
```

Respostas no assistente:

| Pergunta | Resposta |
|---|---|
| Omitir configuração? | **Não** |
| Nome de domínio DNS | `curso.lan` (gera a base `dc=curso,dc=lan`) |
| Organização | `Curso Linux` |
| Senha do administrador | (anote) |
| Remover a base ao remover o slapd? | Não |
| Mover a base antiga? | Sim |

```bash
sudo slapcat                     # exporta tudo (só funciona localmente, como root)
```

{{% notice note "cn=config" %}}
O OpenLDAP atual guarda a **própria configuração** dentro do LDAP, na árvore `cn=config`,
alterada com LDIF e `ldapmodify -Y EXTERNAL -H ldapi:///`. O antigo `slapd.conf` não é mais usado.
{{% /notice %}}

## Cliente de linha de comando

Para não repetir opções, crie `/etc/ldap/ldap.conf` (vale para os comandos `ldap*`):

```text
BASE    dc=curso,dc=lan
URI     ldap://srv01.curso.lan
```

## Estrutura inicial

`base.ldif`:

```text
dn: ou=pessoas,dc=curso,dc=lan
objectClass: organizationalUnit
ou: pessoas

dn: ou=grupos,dc=curso,dc=lan
objectClass: organizationalUnit
ou: grupos

dn: cn=funcionarios,ou=grupos,dc=curso,dc=lan
objectClass: posixGroup
cn: funcionarios
gidNumber: 10000
```

```bash
ldapadd -x -D cn=admin,dc=curso,dc=lan -W -f base.ldif
```

## Usuários

`maria.ldif`:

```text
dn: uid=maria,ou=pessoas,dc=curso,dc=lan
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: maria
cn: Maria Souza
sn: Souza
mail: maria@curso.lan
uidNumber: 10001
gidNumber: 10000
homeDirectory: /home/maria
loginShell: /bin/bash
```

```bash
ldapadd -x -D cn=admin,dc=curso,dc=lan -W -f maria.ldif
ldappasswd -x -D cn=admin,dc=curso,dc=lan -W -S uid=maria,ou=pessoas,dc=curso,dc=lan
```

Adicionando a Maria a um grupo extra (`memberUid`):

```text
dn: cn=funcionarios,ou=grupos,dc=curso,dc=lan
changetype: modify
add: memberUid
memberUid: maria
```

```bash
ldapmodify -x -D cn=admin,dc=curso,dc=lan -W -f grupo.ldif
```

## Consultas

```bash
ldapsearch -x                                    # tudo (leitura anônima)
ldapsearch -x uid=maria                          # filtro
ldapsearch -x '(objectClass=posixGroup)' cn gidNumber
ldapsearch -x -D cn=admin,dc=curso,dc=lan -W '(uid=maria)' '*' '+'   # inclui atributos operacionais
ldapdelete -x -D cn=admin,dc=curso,dc=lan -W uid=joao,ou=pessoas,dc=curso,dc=lan
```

## TLS

Senhas não devem trafegar em texto puro. Gere um certificado (de uma CA interna ou do
Let's Encrypt, se o nome for público), configure `olcTLSCertificateFile` e
`olcTLSCertificateKeyFile` em `cn=config` e use `ldap://` com **StartTLS** (`ldapsearch -ZZ`)
ou `ldaps://`.

## Ferramentas gráficas

- **Apache Directory Studio**: cliente desktop completo.
- **phpLDAPadmin** / **LDAP Account Manager (LAM)**: interfaces web — o LAM facilita muito
  a criação de usuários POSIX.
