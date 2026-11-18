+++
title = "Conceitos de LDAP"
weight = 1
description = "Árvore de diretório, DN, objectClass, atributos e LDIF."
+++

**LDAP** (*Lightweight Directory Access Protocol*) é um protocolo para consultar e alterar
um **diretório**: uma base de dados hierárquica, otimizada para muitas leituras —
usuários, grupos, máquinas, catálogos de endereços.

O Active Directory da Microsoft, o FreeIPA e o OpenLDAP falam LDAP.

## A árvore (DIT)

```text
dc=curso,dc=lan                         ← base (o domínio)
├── ou=pessoas
│   ├── uid=maria
│   └── uid=joao
└── ou=grupos
    ├── cn=funcionarios
    └── cn=financeiro
```

## Nomes

| Sigla | Significado | Exemplo |
|---|---|---|
| `dc` | *domain component* | `dc=curso,dc=lan` |
| `ou` | *organizational unit* (uma "pasta") | `ou=pessoas` |
| `cn` | *common name* | `cn=funcionarios` |
| `uid` | identificador do usuário | `uid=maria` |

O **DN** (*distinguished name*) é o caminho completo e único de uma entrada, lido da
esquerda (mais específico) para a direita:

```text
uid=maria,ou=pessoas,dc=curso,dc=lan
```

## Entradas, objectClass e atributos

Cada entrada tem **atributos**; quais atributos ela pode ou deve ter é definido pelas suas
**objectClasses**, descritas no **esquema**:

| objectClass | Para quê | Atributos |
|---|---|---|
| `inetOrgPerson` | Pessoa | `cn`, `sn`, `mail`, `telephoneNumber` |
| `posixAccount` | Conta Unix | `uid`, `uidNumber`, `gidNumber`, `homeDirectory`, `loginShell` |
| `posixGroup` | Grupo Unix | `cn`, `gidNumber`, `memberUid` |
| `organizationalUnit` | OU | `ou` |

## LDIF

Entradas são lidas e escritas no formato texto **LDIF**:

```text
dn: uid=maria,ou=pessoas,dc=curso,dc=lan
objectClass: inetOrgPerson
objectClass: posixAccount
uid: maria
cn: Maria Souza
sn: Souza
mail: maria@curso.lan
uidNumber: 10001
gidNumber: 10000
homeDirectory: /home/maria
loginShell: /bin/bash
```

## Portas

| Porta | Uso |
|---|---|
| 389/TCP | LDAP (com **StartTLS** para criptografar) |
| 636/TCP | LDAPS (TLS desde o início) |

{{% notice note "Alternativas" %}}
Para uma rede só de Linux, o **FreeIPA** (LDAP + Kerberos + DNS + certificados, com
interface web) é bem mais simples de operar do que montar tudo à mão. Em redes mistas,
**Samba AD** ou o próprio Active Directory. Aqui usamos o OpenLDAP para entender as peças.
{{% /notice %}}
