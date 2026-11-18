+++
title = "Resposta a incidentes"
weight = 3
description = "O que fazer quando suspeitar de uma invasão."
+++

## Sinais de comprometimento

- Processos desconhecidos ou CPU alta sem motivo (mineração de criptomoeda).
- Conexões de saída estranhas (`ss -tnp`), especialmente para portas altas ou IRC.
- Usuários, chaves SSH, tarefas de cron ou serviços que ninguém criou.
- Logs apagados ou com lacunas; alertas do AIDE, Suricata ou auditd.
- Reclamações externas: o IP entrou em listas de spam, o provedor avisou de ataques saindo.

## Primeira avaliação (sem destruir evidências)

```bash
w; last -n 20                                # quem está/esteve conectado
ps auxf --sort=-%cpu | head -20
sudo ss -tunap                               # conexões e processos
sudo lsof -i -P -n | grep -v LISTEN
sudo find / -xdev -mtime -2 -type f -not -path '/proc/*' 2>/dev/null | head -50   # alterados em 2 dias
for u in $(cut -d: -f1 /etc/passwd); do sudo crontab -l -u $u 2>/dev/null | sed "s/^/$u: /"; done
sudo cat /root/.ssh/authorized_keys; ls -la /home/*/.ssh/
systemctl list-unit-files --state=enabled
```

Lembre-se: se o invasor tem root, os comandos do sistema podem estar **adulterados**.
Prefira ferramentas de um pendrive confiável ou analise o disco a partir de outra máquina.

## Etapas da resposta

1. **Preparação** (antes!): contatos, backups testados, logs centralizados, este roteiro.
2. **Identificação**: confirmar o incidente e o escopo — quais máquinas, quais dados.
3. **Contenção**: isolar a máquina da rede (desconectar ou bloquear no firewall/switch),
   sem desligá-la se for coletar evidências da memória.
4. **Coleta de evidências**: cópia da memória (LiME, AVML), imagem do disco (`dd`), logs —
   com hash e registro de quem manipulou (cadeia de custódia).
5. **Erradicação**: descobrir **como** entraram e corrigir a causa.
6. **Recuperação**: **reinstalar** a partir de mídia confiável e restaurar dados de um
   backup **anterior** à invasão; trocar **todas** as credenciais que estavam na máquina.
7. **Lições aprendidas**: o que falhou, o que detectou, o que muda.

{{% notice warning "Não \"limpe\" um servidor invadido" %}}
Remover o malware e seguir em frente quase nunca é suficiente: rootkits e backdoors
sobrevivem. Um sistema comprometido com acesso root deve ser **reinstalado**.
{{% /notice %}}

## Obrigações legais

Incidentes com **dados pessoais** podem exigir comunicação à **ANPD** e aos titulares
(LGPD, art. 48). Envolva o jurídico/DPO da organização desde o início.

## Exercícios

1. Escreva o plano de resposta a incidentes do seu laboratório: contatos, onde estão os
   backups, como isolar cada máquina.
2. Peça a um colega para "invadir" o `srv01` (criar um usuário, uma tarefa no cron e uma
   chave SSH) e tente descobrir tudo com os comandos acima, o AIDE e o auditd.
