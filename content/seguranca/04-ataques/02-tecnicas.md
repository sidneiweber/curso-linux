+++
title = "Técnicas e ferramentas"
weight = 2
description = "Reconhecimento, força bruta, falhas web, escalada de privilégio — e como se defender."
+++

As ferramentas abaixo são usadas tanto por atacantes quanto por profissionais de
segurança em **testes de invasão autorizados**. A distribuição **Kali Linux** reúne todas.
Use-as apenas no laboratório.

## Reconhecimento

| Técnica | Ferramentas | Defesa |
|---|---|---|
| Varredura de portas e versões | `nmap -sV`, masscan | Firewall com `drop`, serviços escutando só onde precisam |
| Enumeração de DNS e subdomínios | `dig`, amass, certificados públicos (crt.sh) | Não publicar nomes internos; transferência de zona bloqueada |
| Informações públicas (OSINT) | Buscadores, LinkedIn, GitHub | Cuidado com o que se publica; varrer repositórios por segredos |

## Acesso por credenciais

| Técnica | Ferramentas | Defesa |
|---|---|---|
| Força bruta / lista de senhas | hydra, medusa | Chaves SSH, MFA, fail2ban, limite de conexões |
| *Password spraying* (uma senha comum em muitos usuários) | — | Política de senhas, bloqueio por conta |
| Credenciais vazadas | Bases de vazamentos | MFA, senhas únicas, monitorar vazamentos (haveibeenpwned) |
| Quebra de hashes roubados | john, hashcat | Hashes fortes (yescrypt), proteger `/etc/shadow` e bancos |

## Aplicações web

| Técnica | Ferramentas | Defesa |
|---|---|---|
| Injeção de SQL | sqlmap | Consultas parametrizadas, usuário do banco com privilégio mínimo |
| XSS, CSRF | Burp Suite, OWASP ZAP | Escape de saída, cabeçalhos (CSP), tokens CSRF |
| Upload de *webshell* | — | Validar uploads, `noexec` no diretório de uploads, PHP sem permissão de escrita no código |
| Componentes desatualizados | wpscan, nuclei | Atualizar CMS/plugins; remover o que não usa |

O **OWASP Top 10** ([owasp.org](https://owasp.org/Top10/)) lista os riscos mais críticos
em aplicações web.

## Exploração de falhas conhecidas

O **Metasploit** reúne exploits prontos para milhares de vulnerabilidades publicadas
(identificadas por **CVE**). A defesa é direta: **atualizar** e saber o que está exposto.

```bash
apt list --upgradable
sudo apt install debsecan && debsecan --suite trixie --only-fixed   # CVEs com correção disponível
```

## Escalada de privilégio local

Depois de entrar como um usuário comum, o atacante procura caminhos para root:

```bash
sudo -l                                         # o que posso rodar com sudo?
find / -perm -4000 -type f 2>/dev/null          # binários SUID
getcap -r / 2>/dev/null                         # capabilities
ls -l /etc/cron* /etc/systemd/system            # scripts graváveis executados como root?
uname -r                                        # kernel vulnerável?
```

Ferramentas como **linpeas** automatizam essa busca. Rode-as **no seu servidor** para
encontrar os problemas antes do atacante.

## Exercícios

1. Do `cli01`, faça um `nmap -sV srv01` antes e depois do firewall e compare.
2. Crie um usuário com senha fraca, ative temporariamente `PasswordAuthentication` e veja o
   `hydra` encontrá-la a partir de outra VM; depois veja o fail2ban bloqueando.
3. Rode as buscas de escalada de privilégio acima no `srv01` e justifique cada binário SUID.
