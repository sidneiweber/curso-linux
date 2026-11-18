+++
title = "Estratégia de backup"
weight = 1
description = "Riscos, regra 3-2-1, tipos de backup, RPO e RTO."
+++

## Do que precisamos nos proteger?

| Ameaça | Exemplo | O que protege |
|---|---|---|
| Falha de hardware | Disco morre | RAID **e** backup |
| Erro humano | `rm -rf` no diretório errado | Backup com histórico, snapshots |
| Software / atualização | Banco de dados corrompido | Backup com histórico |
| Ransomware / invasão | Arquivos criptografados | Cópia **offline** ou imutável |
| Desastre físico | Incêndio, roubo | Cópia **fora do local** |

{{% notice warning "RAID não é backup" %}}
RAID protege contra a falha de um disco. Se você apagar um arquivo, o RAID apaga em todos
os discos ao mesmo tempo.
{{% /notice %}}

## Regra 3-2-1

- **3** cópias dos dados (o original + 2 backups);
- em **2** mídias/tipos diferentes;
- **1** delas fora do local (outra cidade, nuvem).

Versões atuais acrescentam: **1** cópia offline/imutável e **0** erros na restauração testada (3-2-1-1-0).

## Tipos de backup

| Tipo | Copia | Restaurar exige |
|---|---|---|
| **Completo** | Tudo | Só o último completo |
| **Incremental** | O que mudou desde o **último backup** (de qualquer tipo) | Completo + **todos** os incrementais |
| **Diferencial** | O que mudou desde o **último completo** | Completo + o último diferencial |

Ferramentas modernas (restic, Borg) fazem **deduplicação**: cada execução parece um backup
completo, mas só os blocos novos ocupam espaço — o melhor dos dois mundos.

## RPO e RTO

- **RPO** (*Recovery Point Objective*): quanto dado posso perder? Backup diário = até 24 h.
- **RTO** (*Recovery Time Objective*): quanto tempo posso ficar parado restaurando?

## O que copiar

| Incluir | Geralmente excluir |
|---|---|
| `/etc` (configurações) | `/proc`, `/sys`, `/dev`, `/run`, `/tmp` |
| `/home`, `/root` | Caches (`~/.cache`, `/var/cache`) |
| `/var/lib` seletivamente (dados de serviços) | Pacotes — reinstale com a lista |
| `/srv`, `/var/www`, `/opt` | |
| **Dumps** de bancos de dados | Arquivos do banco copiados com o banco rodando |
| Lista de pacotes: `dpkg --get-selections` | |
