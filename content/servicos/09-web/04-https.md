+++
title = "HTTPS"
weight = 4
description = "TLS com Let's Encrypt ou certificado interno, redirecionamento e cabeçalhos."
+++

Sem HTTPS, senhas e cookies trafegam em texto puro e navegadores marcam o site como
"Não seguro". O certificado prova que o servidor é quem diz ser.

## Sites públicos: Let's Encrypt

Para um domínio **real**, apontado para um servidor acessível pela Internet, o certificado é
gratuito e renovado automaticamente:

```bash
sudo apt install certbot python3-certbot-apache
sudo certbot --apache -d exemplo.com.br -d www.exemplo.com.br
systemctl list-timers | grep certbot      # renovação automática
sudo certbot renew --dry-run
```

O certbot cria o vhost `:443` e o redirecionamento HTTP → HTTPS.

## Laboratório/rede interna: certificado próprio

Para nomes internos como `curso.lan`, o Let's Encrypt não funciona. Crie uma **autoridade
certificadora (CA) interna** — com o `mkcert` é rápido:

```bash
sudo apt install mkcert libnss3-tools
mkcert -install                    # cria a CA local
mkcert www.curso.lan intranet.curso.lan
sudo mkdir -p /etc/ssl/curso
sudo mv www.curso.lan+1.pem /etc/ssl/curso/cert.pem
sudo mv www.curso.lan+1-key.pem /etc/ssl/curso/key.pem
```

Distribua o certificado da CA (`mkcert -CAROOT`, arquivo `rootCA.pem`) para os clientes e
instale-o como confiável (`/usr/local/share/ca-certificates/` + `update-ca-certificates`).

## Configurando o vhost

```bash
sudo a2enmod ssl headers http2
```

`/etc/apache2/sites-available/www.curso.lan.conf`:

```text
<VirtualHost *:80>
    ServerName www.curso.lan
    Redirect permanent / https://www.curso.lan/
</VirtualHost>

<VirtualHost *:443>
    ServerName www.curso.lan
    DocumentRoot /var/www/www.curso.lan
    Protocols h2 http/1.1

    SSLEngine on
    SSLCertificateFile    /etc/ssl/curso/cert.pem
    SSLCertificateKeyFile /etc/ssl/curso/key.pem

    Header always set Strict-Transport-Security "max-age=31536000"
    Header always set X-Content-Type-Options "nosniff"

    ErrorLog ${APACHE_LOG_DIR}/www.curso.lan-error.log
    CustomLog ${APACHE_LOG_DIR}/www.curso.lan-access.log combined
</VirtualHost>
```

```bash
sudo apache2ctl configtest && sudo systemctl reload apache2
curl -vI https://www.curso.lan 2>&1 | grep -E 'subject|issuer|HTTP/'
openssl s_client -connect www.curso.lan:443 -servername www.curso.lan </dev/null | head
```

{{% notice tip "Configuração TLS recomendada" %}}
O [Mozilla SSL Configuration Generator](https://ssl-config.mozilla.org/) gera as diretivas
de protocolos e cifras para Apache, Nginx e outros. Para sites públicos, confira a nota no
[SSL Labs](https://www.ssllabs.com/ssltest/).
{{% /notice %}}
