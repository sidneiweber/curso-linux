+++
title = "Conceitos"
weight = 1
description = "HTTP, servidores web, Apache x Nginx."
+++

## HTTP em 30 segundos

O navegador abre uma conexão (porta **80** para HTTP, **443** para HTTPS) e envia uma
**requisição**; o servidor devolve uma **resposta** com um código de status:

```bash
curl -v http://srv01.curso.lan/
```

```text
> GET / HTTP/1.1
> Host: srv01.curso.lan
>
< HTTP/1.1 200 OK
< Server: Apache
< Content-Type: text/html
```

| Código | Significado |
|---|---|
| `200` | OK |
| `301` / `302` | Redirecionamento permanente / temporário |
| `401` / `403` | Não autenticado / proibido |
| `404` | Não encontrado |
| `500` / `502` / `503` | Erro no servidor / no serviço por trás do proxy / indisponível |

O cabeçalho **`Host`** permite que um único IP hospede **vários sites**: o servidor escolhe
qual entregar pelo nome pedido. É o princípio dos **sites virtuais** (*virtual hosts*).

## Apache ou Nginx?

| | Apache | Nginx |
|---|---|---|
| Arquitetura | Processos/threads (MPM) | Orientado a eventos |
| Configuração | `.htaccess` por diretório, muitos módulos | Centralizada, sintaxe enxuta |
| Ponto forte | Flexibilidade, hospedagem compartilhada, PHP legado | Arquivos estáticos, proxy reverso, alto volume de conexões |

Os dois são excelentes e muito usados. É comum ver o Nginx na frente (TLS, cache,
estáticos) repassando para o Apache ou para aplicações.
