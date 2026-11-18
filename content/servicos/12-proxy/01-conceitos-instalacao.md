+++
title = "Conceitos e instalação"
weight = 1
description = "Tipos de proxy, HTTPS, instalação e configuração dos clientes."
+++

## Tipos de proxy

| Tipo | Como funciona |
|---|---|
| **Explícito** | O navegador é configurado para usar o proxy (manual, WPAD/PAC ou política) |
| **Transparente** | O firewall redireciona o tráfego para o proxy sem configurar os clientes |
| **Reverso** | Fica na frente de **servidores** (Nginx, capítulo 9) — o oposto deste capítulo |

## E o HTTPS?

Hoje quase todo o tráfego é HTTPS. Com o proxy explícito, o navegador usa o método
`CONNECT` e o Squid apenas vê o **nome do site** (não as páginas nem o conteúdo). Isso já
permite bloquear por domínio e registrar acessos.

Inspecionar o conteúdo exige **SSL Bump**: o proxy abre o HTTPS com um certificado próprio
instalado em todos os clientes. É invasivo, quebra aplicativos com *certificate pinning* e
tem implicações legais (LGPD) — não abordaremos aqui.

{{% notice note "Cache" %}}
O ganho de cache, grande na época do HTTP, é pequeno com HTTPS. Hoje o Squid é usado
principalmente para **controle de acesso e registro**.
{{% /notice %}}

## Instalação

```bash
sudo apt install squid
sudo cp /etc/squid/squid.conf /etc/squid/squid.conf.orig
```

O arquivo original tem milhares de linhas de documentação. Veja só as linhas ativas:

```bash
grep -vE '^\s*(#|$)' /etc/squid/squid.conf
```

## Configuração mínima

As suas regras vão no `/etc/squid/squid.conf`, **logo abaixo** da linha:

```text
# INSERT YOUR OWN RULE(S) HERE TO ALLOW ACCESS FROM YOUR CLIENTS
```

Nesse ponto, as proteções padrão (bloqueio de portas não seguras e de `CONNECT` fora da
porta 443) já foram aplicadas, e depois vem o `http_access deny all` final.

```text
acl rede_interna src 192.168.56.0/24
http_access allow rede_interna
```

{{% notice warning "Não use o conf.d para as regras de acesso" %}}
No Debian, os arquivos de `/etc/squid/conf.d/` são incluídos no **início** do `squid.conf`,
antes das proteções padrão. Regras `http_access` colocadas ali passam na frente delas.
{{% /notice %}}

```bash
sudo squid -k parse            # valida
sudo systemctl restart squid
```

O Squid escuta na porta **3128**.

## Configurando os clientes

```bash
# teste rápido
curl -x http://proxy.curso.lan:3128 -I https://debian.org

# variáveis de ambiente (terminal, apt, curl, wget)
export http_proxy=http://proxy.curso.lan:3128
export https_proxy=$http_proxy
```

No navegador: Configurações → Rede → Proxy manual `proxy.curso.lan`, porta `3128`, para
HTTP e HTTPS. Em ambientes Windows com AD, distribui-se por GPO.

Para impedir que os usuários "pulem" o proxy, o firewall do `srv01` deve **bloquear** o
encaminhamento direto da LAN para as portas 80/443 da Internet (capítulo 2).
