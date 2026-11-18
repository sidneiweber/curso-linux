+++
title = "BIND 9: zonas autoritativas"
weight = 3
description = "Zona direta, zona reversa, serial e verificação."
+++

## Declarando as zonas

`/etc/bind/named.conf.local`:

```text
zone "curso.lan" {
    type primary;
    file "/etc/bind/db.curso.lan";
};

zone "56.168.192.in-addr.arpa" {
    type primary;
    file "/etc/bind/db.192.168.56";
};
```

## Zona direta

`/etc/bind/db.curso.lan`:

```text
$TTL 3600
@   IN  SOA srv01.curso.lan. admin.curso.lan. (
            2026111801  ; serial (AAAAMMDDnn)
            3600        ; refresh
            900         ; retry
            604800      ; expire
            300 )       ; TTL negativo

    IN  NS  srv01.curso.lan.
    IN  MX  10 mail.curso.lan.

srv01   IN  A       192.168.56.10
cli01   IN  A       192.168.56.20
mail    IN  CNAME   srv01
www     IN  CNAME   srv01
ldap    IN  CNAME   srv01
proxy   IN  CNAME   srv01
arquivos IN CNAME   srv01
```

- `@` representa o próprio domínio (`curso.lan.`).
- Nomes **sem** ponto final são relativos à zona (`srv01` → `srv01.curso.lan.`); nomes
  **com** ponto final são absolutos. Esquecer o ponto é o erro mais comum:
  `srv01.curso.lan` (sem ponto) vira `srv01.curso.lan.curso.lan.`.
- No SOA, o e-mail é escrito com ponto no lugar do `@`: `admin.curso.lan.`.

## Zona reversa

`/etc/bind/db.192.168.56`:

```text
$TTL 3600
@   IN  SOA srv01.curso.lan. admin.curso.lan. (
            2026111801 3600 900 604800 300 )

    IN  NS  srv01.curso.lan.

10  IN  PTR srv01.curso.lan.
20  IN  PTR cli01.curso.lan.
```

## Verificando e carregando

```bash
sudo named-checkzone curso.lan /etc/bind/db.curso.lan
sudo named-checkzone 56.168.192.in-addr.arpa /etc/bind/db.192.168.56
sudo rndc reload
```

{{% notice warning "Incremente o serial" %}}
A cada alteração, **aumente o serial** do SOA. Secundários só copiam a zona quando o serial
aumenta, e é fácil esquecer. O formato `AAAAMMDDnn` (data + contador) ajuda.
{{% /notice %}}

## Testando

```bash
dig @192.168.56.10 srv01.curso.lan
dig @192.168.56.10 www.curso.lan
dig @192.168.56.10 curso.lan MX
dig @192.168.56.10 -x 192.168.56.20
dig @192.168.56.10 curso.lan AXFR     # deve ser RECUSADO (allow-transfer none)
```

No `cli01`, com `curso.lan` como domínio de busca, nomes curtos funcionam:

```bash
ping srv01
```

## Servidor secundário (opcional)

Em produção, toda zona deve ter pelo menos dois servidores. No primário, libere a
transferência para o secundário e avise-o das mudanças:

```text
zone "curso.lan" {
    type primary;
    file "/etc/bind/db.curso.lan";
    allow-transfer { 192.168.56.11; };
    also-notify { 192.168.56.11; };
};
```

No secundário:

```text
zone "curso.lan" {
    type secondary;
    primaries { 192.168.56.10; };
    file "/var/lib/bind/db.curso.lan";
};
```

## Exercícios

1. Crie as zonas direta e reversa e resolva todos os nomes a partir do `cli01`.
2. Adicione um registro `A` para `impressora` (`192.168.56.50`), **sem** incrementar o
   serial, recarregue e observe; depois corrija.
3. Use `dig +trace` para acompanhar a resolução de `www.debian.org` a partir da raiz.
