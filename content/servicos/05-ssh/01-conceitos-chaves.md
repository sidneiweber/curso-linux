+++
title = "Conceitos e autenticação por chaves"
weight = 1
description = "Como o SSH funciona, chaves do servidor e do usuário, ssh-agent."
+++

## Como funciona

1. O cliente conecta na porta **22/TCP**.
2. O servidor apresenta sua **chave de host**; o cliente confere se é a mesma da última vez
   (`~/.ssh/known_hosts`) — isso impede que alguém se passe pelo servidor.
3. Os dois combinam uma chave de sessão; a partir daí **tudo é criptografado**.
4. O usuário se autentica: por **senha** ou, melhor, por **chave pública**.

Na primeira conexão aparece a impressão digital do servidor:

```text
The authenticity of host '192.168.56.10' can't be established.
ED25519 key fingerprint is SHA256:Xb3k...
Are you sure you want to continue connecting (yes/no)?
```

Confira com o administrador (ou no servidor, com `ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub`).
Se essa mensagem aparecer **de novo** para um servidor conhecido, com aviso de chave
alterada, desconfie.

## Chaves de usuário

Um par de chaves: a **privada** fica com você (nunca sai da sua máquina); a **pública** vai
para o servidor.

```bash
ssh-keygen -t ed25519 -C "maria@notebook"     # gera ~/.ssh/id_ed25519 e .pub
ssh-copy-id usuario@192.168.56.10            # copia a pública para ~/.ssh/authorized_keys
ssh usuario@192.168.56.10                    # entra sem senha do servidor
```

Proteja a chave privada com uma **passphrase** e use o **ssh-agent** para digitá-la uma vez
por sessão:

```bash
eval "$(ssh-agent)"
ssh-add ~/.ssh/id_ed25519
ssh-add -l
```

{{% notice tip "Permissões" %}}
O SSH recusa chaves com permissões frouxas. `~/.ssh` deve ser `700`, `authorized_keys` e a
chave privada `600`.
{{% /notice %}}

## Configuração do cliente: ~/.ssh/config

```text
Host srv01
    HostName 192.168.56.10
    User maria
    IdentityFile ~/.ssh/id_ed25519

Host interno-*
    ProxyJump srv01          # passa pelo srv01 para chegar às máquinas internas
```

```bash
ssh srv01
```
