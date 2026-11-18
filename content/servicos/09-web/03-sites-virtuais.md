+++
title = "Sites virtuais e logs"
weight = 3
description = "Hospedando vários sites, autenticação básica e formato de logs."
+++

Vamos publicar dois sites no mesmo servidor: `www.curso.lan` e `intranet.curso.lan`
(crie os registros `CNAME` no DNS do capítulo 4).

```bash
sudo mkdir -p /var/www/{www,intranet}.curso.lan
echo '<h1>Site público</h1>' | sudo tee /var/www/www.curso.lan/index.html
echo '<h1>Intranet</h1>'     | sudo tee /var/www/intranet.curso.lan/index.html
```

`/etc/apache2/sites-available/www.curso.lan.conf`:

```text
<VirtualHost *:80>
    ServerName www.curso.lan
    ServerAlias curso.lan
    DocumentRoot /var/www/www.curso.lan

    <Directory /var/www/www.curso.lan>
        Options -Indexes
        AllowOverride None
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/www.curso.lan-error.log
    CustomLog ${APACHE_LOG_DIR}/www.curso.lan-access.log combined
</VirtualHost>
```

`/etc/apache2/sites-available/intranet.curso.lan.conf` — só para a rede interna e com senha:

```text
<VirtualHost *:80>
    ServerName intranet.curso.lan
    DocumentRoot /var/www/intranet.curso.lan

    <Directory /var/www/intranet.curso.lan>
        AuthType Basic
        AuthName "Intranet"
        AuthUserFile /etc/apache2/.htpasswd
        <RequireAll>
            Require ip 192.168.56.0/24
            Require valid-user
        </RequireAll>
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/intranet-error.log
    CustomLog ${APACHE_LOG_DIR}/intranet-access.log combined
</VirtualHost>
```

```bash
sudo htpasswd -c /etc/apache2/.htpasswd maria      # -c só na primeira vez
sudo a2ensite www.curso.lan intranet.curso.lan
sudo a2dissite 000-default
sudo apache2ctl configtest && sudo systemctl reload apache2
sudo apache2ctl -S          # lista os vhosts e qual arquivo define cada um
```

```bash
curl http://www.curso.lan
curl -u maria http://intranet.curso.lan
```

{{% notice tip "Autenticação no LDAP" %}}
Com o módulo `authnz_ldap`, a intranet pode aceitar os usuários do LDAP (capítulo 8) em vez
de um `.htpasswd`: `AuthBasicProvider ldap` e `AuthLDAPURL "ldap://srv01/ou=pessoas,dc=curso,dc=lan?uid"`.
{{% /notice %}}

## Logs

O formato `combined` registra:

```text
192.168.56.20 - maria [18/Nov/2026:10:15:32 -0300] "GET / HTTP/1.1" 200 312 "-" "curl/8.14.1"
  cliente      usuário        data                    requisição   status bytes referer agente
```

```bash
sudo tail -f /var/log/apache2/www.curso.lan-access.log
# páginas mais acessadas
sudo awk '{print $7}' /var/log/apache2/www.curso.lan-access.log | sort | uniq -c | sort -rn | head
# erros 404
sudo awk '$9 == 404 {print $7}' /var/log/apache2/*access.log | sort | uniq -c | sort -rn | head
```

Os logs do Apache já vêm com rotação configurada em `/etc/logrotate.d/apache2`. Para
relatórios visuais, use o **GoAccess** (`goaccess access.log --log-format=COMBINED`).
