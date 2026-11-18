+++
title = "Lynis e OpenVAS"
weight = 2
description = "Auditoria de configuração local e varredura de vulnerabilidades."
+++

## Lynis: auditoria do próprio servidor

O **Lynis** roda **dentro** da máquina e verifica centenas de itens de configuração: SSH,
kernel, permissões, serviços, firewall, logs, contas…

```bash
sudo apt install lynis
sudo lynis audit system
```

Ao final, mostra um **índice de endurecimento** (*hardening index*), **avisos** e
**sugestões**, cada uma com um código e link para explicação:

```bash
sudo grep -E '^(warning|suggestion)' /var/log/lynis-report.dat | head -30
sudo lynis show details SSH-7408
```

Use-o como lista de tarefas: corrija, rode de novo, compare o índice. Não persiga 100 —
algumas sugestões não se aplicam ao seu caso.

## OpenVAS / Greenbone: varredura de vulnerabilidades

O **OpenVAS** (parte do **Greenbone Community Edition**) varre máquinas **pela rede** e
compara os serviços encontrados com uma base de dezenas de milhares de testes de
vulnerabilidades (CVEs), gerando relatórios com gravidade e recomendações.

{{% notice note "Nessus" %}}
Materiais antigos citam o **Nessus**, que deixou de ser livre em 2005 — o OpenVAS nasceu
como continuação aberta dele. O Nessus continua existindo como produto comercial.
{{% /notice %}}

A forma mais simples de instalar é com containers. Siga o guia oficial
[Greenbone Community Containers](https://greenbone.github.io/docs/latest/22.4/container/):
ele indica o arquivo `compose` da versão atual e os comandos para baixar as imagens e
subir os serviços (`docker compose ... up -d`).

A interface web fica em `http://127.0.0.1:9392`. A primeira sincronização da base de testes
leva bastante tempo (às vezes horas).

Fluxo de uso:

1. **Configuration → Targets**: cadastre `192.168.56.0/24` (ou o `srv01`).
2. **Scans → Tasks**: crie uma tarefa com o perfil *Full and fast* e execute.
3. Analise o relatório: comece pelos itens **High** e **Critical**.

Para resultados mais precisos, configure **credenciais** (SSH) no alvo: o scanner entra na
máquina e verifica as versões dos pacotes instalados (*authenticated scan*).

## Exercícios

1. Rode o Lynis no `srv01`, anote o índice, corrija cinco sugestões e rode novamente.
2. Varra o `srv01` com o OpenVAS e classifique os achados: corrigir, aceitar o risco ou falso positivo.
