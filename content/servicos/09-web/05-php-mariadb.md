+++
title = "PHP e MariaDB"
weight = 5
description = "PHP-FPM, módulos, banco de dados MariaDB e backups."
+++

## PHP com PHP-FPM

O jeito moderno de rodar PHP é o **PHP-FPM**: um serviço separado, que o Apache acessa via
FastCGI. (O antigo `mod_php` obriga o Apache a usar o MPM *prefork*, mais pesado.)

```bash
sudo apt install php-fpm php-mysql php-cli php-curl php-gd php-mbstring php-xml php-zip php-intl
sudo a2enmod proxy_fcgi setenvif
sudo a2enconf php8.4-fpm
sudo systemctl reload apache2
```

Teste (e **apague** depois — ele expõe detalhes do servidor):

```bash
echo '<?php phpinfo();' | sudo tee /var/www/www.curso.lan/info.php
curl -sk https://www.curso.lan/info.php | grep -m1 'PHP Version'
sudo rm /var/www/www.curso.lan/info.php
```

## Configuração do PHP

O PHP-FPM lê `/etc/php/8.4/fpm/php.ini` (o CLI tem o seu próprio, em `cli/`). Prefira
sobrescrever em um arquivo separado:

`/etc/php/8.4/fpm/conf.d/90-curso.ini`:

```ini
memory_limit = 256M
upload_max_filesize = 64M
post_max_size = 64M
max_execution_time = 60
date.timezone = America/Sao_Paulo
expose_php = Off
```

```bash
php -m                              # módulos carregados
sudo systemctl restart php8.4-fpm
```

## MariaDB

```bash
sudo apt install mariadb-server
sudo mariadb-secure-installation    # remove usuários anônimos e a base de testes
sudo mariadb                        # o root local autentica pelo usuário do sistema
```

Criando a base e o usuário de uma aplicação:

```sql
CREATE DATABASE app CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'app'@'localhost' IDENTIFIED BY 'SenhaForte!2026';
GRANT ALL PRIVILEGES ON app.* TO 'app'@'localhost';
FLUSH PRIVILEGES;
SHOW DATABASES;
```

{{% notice warning "Escutar só localmente" %}}
Por padrão o MariaDB escuta só em `127.0.0.1` (`bind-address` em
`/etc/mysql/mariadb.conf.d/50-server.cnf`). Mantenha assim se o banco e o site estão na
mesma máquina. Se precisar de acesso remoto, libere apenas o IP da aplicação no firewall.
{{% /notice %}}

## Testando a pilha completa

```bash
sudo tee /var/www/www.curso.lan/db.php <<'PHP'
<?php
$db = new PDO('mysql:host=localhost;dbname=app;charset=utf8mb4', 'app', 'SenhaForte!2026');
echo 'Conectado ao MariaDB ' . $db->query('SELECT VERSION()')->fetchColumn();
PHP
curl -sk https://www.curso.lan/db.php
```

## Backup do banco

```bash
sudo mariadb-dump --single-transaction --routines app | zstd > /var/backups/app-$(date +%F).sql.zst
zstdcat /var/backups/app-2026-11-18.sql.zst | sudo mariadb app     # restauração
```

Agende o dump com um timer do systemd e inclua `/var/backups` no backup (capítulo de
backup do Linux Avançado).
