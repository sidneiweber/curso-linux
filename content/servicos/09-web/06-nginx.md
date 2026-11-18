+++
title = "Nginx e proxy reverso"
weight = 6
description = "Servidor estático, PHP-FPM e proxy reverso para aplicações."
+++

{{% notice note "Porta 80 ocupada" %}}
Apache e Nginx não podem escutar na mesma porta. Para praticar, use uma VM separada ou
pare o Apache (`sudo systemctl disable --now apache2`).
{{% /notice %}}

```bash
sudo apt install nginx
```

A estrutura no Debian é parecida com a do Apache: `sites-available/` e `sites-enabled/`
(sem comandos `a2en*` — crie o link com `ln -s`).

## Site estático com PHP

`/etc/nginx/sites-available/www.curso.lan`:

```nginx
server {
    listen 80;
    server_name www.curso.lan;
    root /var/www/www.curso.lan;
    index index.php index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.4-fpm.sock;
    }

    access_log /var/log/nginx/www.curso.lan.access.log;
    error_log  /var/log/nginx/www.curso.lan.error.log;
}
```

```bash
sudo ln -s /etc/nginx/sites-available/www.curso.lan /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default
sudo nginx -t && sudo systemctl reload nginx
```

## Proxy reverso

Aplicações modernas (Node.js, Python, Go, Java, containers) têm servidor HTTP próprio,
escutando em uma porta local. O Nginx fica na frente, cuidando de TLS, compressão,
cache e logs:

```nginx
server {
    listen 80;
    server_name app.curso.lan;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Teste com uma aplicação qualquer:

```bash
python3 -m http.server 3000 --bind 127.0.0.1 &
curl -H 'Host: app.curso.lan' http://localhost/
```

No Apache, o equivalente é `ProxyPass / http://127.0.0.1:3000/` com os módulos `proxy` e
`proxy_http`.

## Exercícios

1. Publique `www.curso.lan` e `intranet.curso.lan` no Apache; a intranet deve exigir senha e
   aceitar só a rede interna.
2. Ative HTTPS com um certificado do `mkcert` e redirecione HTTP → HTTPS.
3. Instale PHP-FPM e MariaDB e faça a página `db.php` conectar ao banco.
4. Em outra VM, coloque o Nginx como proxy reverso para uma aplicação na porta 3000.
