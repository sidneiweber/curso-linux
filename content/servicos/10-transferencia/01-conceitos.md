+++
title = "FTP, FTPS e SFTP"
weight = 1
description = "Diferenças entre os protocolos e qual escolher."
+++

| Protocolo | Porta | Criptografia | Observações |
|---|---|---|---|
| **FTP** | 21 + portas de dados | **Nenhuma** — senha em texto puro | Dois canais (controle e dados), modos ativo/passivo: complica firewall e NAT |
| **FTPS** | 21 (explícito) / 990 | TLS | É o FTP com TLS; mesmas complicações de portas |
| **SFTP** | 22 | SSH | Nada a ver com FTP: é um subsistema do SSH. Uma porta só, chaves, mesmo servidor do SSH |

{{% notice tip "Recomendação" %}}
Use **SFTP** sempre que possível. Clientes como FileZilla, WinSCP e Cyberduck suportam
SFTP. Mantenha FTP apenas para equipamentos/sistemas que não suportam outra coisa — e
nesse caso, com TLS (FTPS) e usuários restritos.
{{% /notice %}}

## FTP: modo ativo × passivo

- **Ativo**: o **servidor** abre a conexão de dados de volta para o cliente — barrado pelo
  NAT/firewall do cliente.
- **Passivo**: o **cliente** abre a conexão de dados para uma porta alta informada pelo
  servidor — é o modo usado hoje; exige liberar uma **faixa de portas** no firewall do servidor.
