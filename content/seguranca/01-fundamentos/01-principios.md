+++
title = "Princípios de segurança"
weight = 1
description = "Confidencialidade, integridade, disponibilidade, menor privilégio e defesa em profundidade."
+++

## O tripé CIA

| Pilar | Pergunta | Ameaça típica | Proteção |
|---|---|---|---|
| **Confidencialidade** | Só quem deve vê os dados? | Vazamento, interceptação | Permissões, criptografia (TLS, LUKS) |
| **Integridade** | Os dados são os originais? | Alteração, malware | Hashes, assinaturas, controle de integridade |
| **Disponibilidade** | O serviço está acessível? | DDoS, falha, ransomware | Redundância, backup, limites |

## Princípios práticos

- **Menor privilégio**: cada usuário e serviço com o mínimo de permissões necessário.
  O servidor web não precisa ler `/etc/shadow`; o estagiário não precisa de `sudo`.
- **Defesa em profundidade**: várias camadas independentes. Se o firewall falhar, a
  autenticação ainda protege; se a senha vazar, o 2FA ainda protege.
- **Superfície de ataque mínima**: o que não está instalado não pode ser explorado.
- **Seguro por padrão**: negar tudo e liberar o necessário — não o contrário.
- **Registrar e monitorar**: sem logs não há como detectar nem investigar.
- **Atualizar**: a maioria das invasões explora falhas **já corrigidas** e não aplicadas.

## Ameaças comuns a servidores Linux

1. Senhas fracas em SSH, painéis e bancos de dados expostos (força bruta automatizada).
2. Aplicações web vulneráveis (CMS e plugins desatualizados, injeção de SQL).
3. Serviços expostos sem necessidade (banco de dados, Redis, painéis de administração).
4. Software desatualizado com falhas conhecidas (CVEs).
5. Credenciais vazadas (chaves em repositórios Git, `.env` público).
6. Configurações erradas (diretórios listáveis, permissões 777, buckets públicos).

## Segurança no Linux × outros sistemas

O Linux herdou do Unix um modelo de segurança sólido:

| Recurso | O que oferece |
|---|---|
| Usuários e permissões | Separação entre usuários e entre serviços |
| root separado + `sudo` | Uso administrativo pontual e registrado |
| Repositórios assinados | Software de origem verificada, atualizado de forma centralizada |
| Código aberto | Falhas são auditáveis e corrigidas rapidamente |
| **AppArmor** / **SELinux** | Controle de acesso obrigatório: limita o que cada programa pode fazer, mesmo como root |
| *Namespaces* e *cgroups* | Isolamento (base dos containers) |
| *Capabilities* | Dividem os poderes do root em partes |

Nada disso protege um sistema mal configurado ou desatualizado — segurança é processo,
não produto.
