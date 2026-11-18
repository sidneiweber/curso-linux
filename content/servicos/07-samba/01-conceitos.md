+++
title = "SMB e o Samba"
weight = 1
description = "O protocolo SMB, como o Samba se integra a redes Windows e seus modos de operação."
+++

## O protocolo SMB

O **SMB** (*Server Message Block*), também chamado de **CIFS** em sua versão antiga, é o
protocolo das "pastas compartilhadas" do Windows. Hoje usa apenas a porta **445/TCP**.

| Versão | Situação |
|---|---|
| SMB1 / CIFS | **Inseguro** (explorado pelo WannaCry). Desativado por padrão no Samba e no Windows |
| SMB2 | Windows Vista/2008 |
| SMB3 | Windows 8/2012 em diante: criptografia, assinatura, desempenho — **use este** |

## O que o Samba faz

- **Servidor de arquivos** para clientes Windows, macOS e Linux.
- **Servidor de impressão** (em conjunto com o CUPS).
- **Membro de domínio**: o servidor Linux entra em um domínio Active Directory existente e
  aceita os usuários dele.
- **Controlador de domínio Active Directory** (Samba AD DC): substitui um Windows Server no
  papel de AD — login centralizado, políticas de grupo (GPO), DNS integrado.

{{% notice note "E o \"Samba como PDC\"?" %}}
Materiais antigos ensinam o Samba como **PDC** (*Primary Domain Controller*), o modelo de
domínio do Windows NT 4. Esse modelo não é suportado por versões atuais do Windows.
Hoje o equivalente é o **Samba AD DC** — ou, mais comumente, um Active Directory/Entra ID
existente, com o Linux como **membro** do domínio.
{{% /notice %}}

## Conceitos de redes Microsoft

| Termo | Significado |
|---|---|
| **Grupo de trabalho** (*workgroup*) | Rede sem servidor central: cada máquina tem seus próprios usuários |
| **Domínio** | Usuários e políticas centralizados em controladores de domínio |
| **Compartilhamento** (*share*) | Pasta (ou impressora) disponibilizada na rede: `\\srv01\publico` |
| **NetBIOS** | Sistema de nomes antigo; hoje substituído por DNS |
