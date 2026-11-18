# Curso Linux

Material do curso de Linux — do básico à administração de serviços e segurança.

🌐 **https://cursolinux.sidneiweber.com.br**

## Estrutura

- `content/basico` — história, instalação, comandos, usuários, permissões
- `content/avancado` — boot, discos, processos, kernel, logs, rede, TCP/IP, IPv6
- `content/servicos` — DHCP, DNS, SSH, NFS, Samba, LDAP, web, e-mail, proxy
- `content/seguranca` — fundamentos, contas, firewall, ataques, auditoria, detecção

## Rodando localmente

Requer [Hugo](https://gohugo.io/) extended (≥ 0.149).

```bash
git clone --recurse-submodules git@github.com:sidneiweber/curso-linux.git
cd curso-linux
hugo server
```

Feito com o tema [Relearn](https://github.com/McShelby/hugo-theme-relearn).
