+++
title = "Restauração"
weight = 4
description = "Recuperando arquivos e sistemas inteiros — e testando antes de precisar."
+++

## Recuperando arquivos

```bash
# restic: um arquivo específico de uma versão
restic snapshots
restic restore latest --target /tmp/restaura --include /home/maria/relatorio.odt
restic mount /mnt/restic        # navega pelas versões como pastas comuns

# tar
tar -xvf etc-2026-11-18.tar.zst -C /tmp/restaura etc/nginx

# rsync (no sentido inverso)
rsync -a /mnt/backup/home/maria/Documentos/ /home/maria/Documentos/
```

Restaure **em um diretório temporário** e só depois copie o que precisa — assim você não
sobrescreve versões mais novas por engano.

## Recuperando um sistema inteiro

1. Instale um Debian mínimo (ou inicie um *live* e particione/formate os discos).
2. Restaure `/etc` e os dados (`/home`, `/srv`, `/var/www`…).
3. Reinstale os pacotes a partir da lista:
   ```bash
   sudo apt install dselect && sudo dselect update
   sudo dpkg --set-selections < /var/backups/pacotes.txt
   sudo apt-get dselect-upgrade
   ```
4. Restaure os bancos de dados a partir dos dumps (`pg_restore`, `mysql < dump.sql`).
5. Ajuste `/etc/fstab` (UUIDs novos!), reinstale o GRUB e reinicie.

Em servidores, guardar a configuração como código (Ansible) torna os passos 1–3
automáticos e repetíveis.

## Teste de restauração

Agende — de verdade, no calendário — um teste periódico:

- restaurar arquivos aleatórios e comparar com o original (`restic check --read-data-subset=5%`);
- subir uma VM a partir do backup e verificar se os serviços funcionam;
- medir quanto tempo levou (é o seu RTO real).

## Exercícios

1. Faça um backup completo e dois incrementais de um diretório com `tar
   --listed-incremental`; apague o diretório e restaure-o.
2. Crie um repositório restic, faça três backups alterando arquivos entre eles e restaure
   um arquivo da primeira versão.
3. Escreva o script de backup da página anterior, agende-o com um timer e verifique o
   resultado no `journalctl`.
