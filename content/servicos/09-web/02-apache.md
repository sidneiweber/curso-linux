+++
title = "Apache"
weight = 2
description = "Instalação, estrutura de arquivos, parâmetros principais e módulos."
+++

```bash
sudo apt install apache2
systemctl status apache2
curl -I http://localhost
```

No `cli01`, acesse `http://srv01.curso.lan` — aparece a página padrão do Debian.

## Estrutura no Debian

| Caminho | Conteúdo |
|---|---|
| `/etc/apache2/apache2.conf` | Configuração principal |
| `/etc/apache2/ports.conf` | Portas (`Listen 80`, `Listen 443`) |
| `/etc/apache2/sites-available/` | Sites virtuais disponíveis |
| `/etc/apache2/sites-enabled/` | Links para os sites **ativos** |
| `/etc/apache2/mods-available/` / `mods-enabled/` | Módulos |
| `/etc/apache2/conf-available/` / `conf-enabled/` | Trechos de configuração |
| `/var/www/` | Arquivos dos sites |
| `/var/log/apache2/` | Logs |

O Debian traz comandos para ativar e desativar itens (eles só criam/removem os links):

```bash
sudo a2ensite meusite        # a2dissite para desativar
sudo a2enmod rewrite         # a2dismod
sudo a2enconf seguranca      # a2disconf
sudo apache2ctl configtest   # valida: "Syntax OK"
sudo systemctl reload apache2
```

## Parâmetros principais

| Diretiva | Função |
|---|---|
| `ServerName` | Nome do site |
| `ServerAlias` | Nomes adicionais |
| `DocumentRoot` | Diretório com os arquivos do site |
| `DirectoryIndex` | Arquivo padrão (`index.html index.php`) |
| `<Directory>` | Regras para um diretório (`Require`, `Options`, `AllowOverride`) |
| `ErrorLog` / `CustomLog` | Arquivos de log |
| `Redirect` | Redirecionamento simples |

## Endurecendo o básico

`/etc/apache2/conf-available/security.conf` (já ativo no Debian):

```text
ServerTokens Prod          # não revela a versão no cabeçalho Server
ServerSignature Off        # nem nas páginas de erro
TraceEnable Off
```

E não liste o conteúdo de diretórios sem `index`:

```text
<Directory /var/www/>
    Options -Indexes +FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
```

## Módulos úteis

| Módulo | Para quê |
|---|---|
| `ssl` | HTTPS |
| `rewrite` | Reescrita de URLs (frameworks PHP) |
| `headers` | Cabeçalhos de segurança (HSTS, CSP) |
| `proxy`, `proxy_http` | Proxy reverso para aplicações |
| `proxy_fcgi` | PHP-FPM |
| `http2` | HTTP/2 |
