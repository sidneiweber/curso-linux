+++
title = "Integridade de arquivos com AIDE"
weight = 1
description = "Detectando alterações em binários e configurações do sistema."
+++

Um invasor costuma alterar o sistema: troca binários (`ps`, `ls`, `sshd`) por versões que o
escondem, adiciona chaves SSH, cria serviços. Um **HIDS** (*Host Intrusion Detection System*)
de integridade tira uma "fotografia" dos arquivos (hashes, permissões, donos) e avisa quando
algo muda.

{{% notice note "Tripwire" %}}
O **Tripwire** é o precursor dessa ideia e ainda existe, mas o **AIDE** é a ferramenta livre
mais usada hoje e vem empacotada no Debian com configuração pronta.
{{% /notice %}}

## Instalação e base inicial

```bash
sudo apt install aide aide-common
sudo aideinit                      # cria a base (demora alguns minutos)
sudo cp /var/lib/aide/aide.db.new /var/lib/aide/aide.db
```

A base precisa ser criada com o sistema **limpo** (logo após a instalação).

## Verificando

```bash
sudo aide --check --config /etc/aide/aide.conf
```

Simule uma alteração e verifique de novo:

```bash
sudo touch /usr/local/bin/suspeito
echo '# teste' | sudo tee -a /etc/ssh/sshd_config
sudo aide --check --config /etc/aide/aide.conf | less
```

## Rotina

1. O pacote do Debian já agenda uma verificação **diária** (`dailyaidecheck`), com relatório
   por e-mail para o root.
2. Após **atualizações legítimas**, aceite as mudanças atualizando a base:

   ```bash
   sudo aide --update --config /etc/aide/aide.conf
   sudo cp /var/lib/aide/aide.db.new /var/lib/aide/aide.db
   ```

3. Guarde uma **cópia da base fora do servidor** (ou em mídia somente leitura) — um invasor
   com root poderia atualizar a base para esconder as alterações.

## Complementos

```bash
sudo apt install debsums rkhunter
sudo debsums -s                 # arquivos de pacotes alterados em relação ao original
sudo rkhunter --update && sudo rkhunter --check --sk   # procura rootkits conhecidos
```

O `rkhunter` gera falsos positivos; avalie cada aviso antes de se alarmar.
