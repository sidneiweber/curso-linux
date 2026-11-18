+++
title = "A estratégia do atacante"
weight = 1
description = "As etapas de uma invasão e onde interrompê-las."
+++

Invasões raramente são um "golpe de mestre". Seguem etapas previsíveis — e cada uma é uma
oportunidade de detecção e bloqueio.

| Etapa | O que o atacante faz | Defesa |
|---|---|---|
| **1. Reconhecimento** | Descobre IPs, domínios, serviços, versões, e-mails de funcionários | Expor o mínimo; não revelar versões; monitorar varreduras |
| **2. Acesso inicial** | Explora uma falha, adivinha uma senha, envia phishing | Atualizações, MFA, chaves SSH, WAF, conscientização |
| **3. Execução e persistência** | Instala *backdoor*, cria usuário, agenda tarefa no cron, adiciona chave SSH | Monitorar integridade de arquivos, `authorized_keys`, cron, novos serviços |
| **4. Escalada de privilégio** | De `www-data` para root: `sudo` mal configurado, SUID, kernel vulnerável | Menor privilégio, AppArmor, atualizações do kernel |
| **5. Movimento lateral** | Usa credenciais/chaves encontradas para invadir outras máquinas | Segmentação de rede, credenciais únicas, firewall interno |
| **6. Objetivo** | Rouba dados, criptografa (ransomware), minera criptomoeda, envia spam | Backup offline, criptografia, monitoramento de tráfego e CPU |
| **7. Encobrimento** | Apaga logs, esconde processos (rootkit) | Logs enviados para **outro** servidor, verificação de integridade |

A base de conhecimento **MITRE ATT&CK** ([attack.mitre.org](https://attack.mitre.org))
cataloga essas táticas e técnicas com exemplos reais — é a referência usada por equipes
de defesa.

## O ataque mais comum não é sofisticado

A maior parte dos ataques a servidores na Internet é **automatizada**: robôs varrem todos
os IPs em busca de SSH com senha fraca, painéis expostos e falhas conhecidas. Um servidor
novo recebe tentativas em **minutos**. Veja no seu:

```bash
journalctl -u ssh --since today | grep -c 'Invalid user'
journalctl -u ssh --since today | grep 'Invalid user' | awk '{print $8}' | sort | uniq -c | sort -rn | head
```
