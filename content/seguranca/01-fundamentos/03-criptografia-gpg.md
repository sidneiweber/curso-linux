+++
title = "Criptografia com GPG"
weight = 3
description = "Chave pública e privada, gerar chaves, cifrar, assinar e verificar arquivos."
+++

## Chave pública e chave privada

Na criptografia **assimétrica**, cada pessoa tem um **par de chaves**:

| Chave | Quem tem | Serve para |
|---|---|---|
| **Pública** | qualquer pessoa (pode ser divulgada) | **cifrar** mensagens para você e **verificar** suas assinaturas |
| **Privada** | **somente você** (protegida por senha) | **decifrar** o que foi cifrado para você e **assinar** |

```mermaid
graph LR
  A[Arquivo] -- "cifra com a chave PÚBLICA de Maria" --> C[Arquivo cifrado]
  C -- "só a chave PRIVADA de Maria decifra" --> D[Arquivo]
```

O mesmo princípio é usado pelo SSH, pelo HTTPS e pela verificação dos pacotes do `apt`.
O **GnuPG** (`gpg`) é a implementação livre do padrão OpenPGP.

## Gerando o par de chaves

```bash
gpg --full-generate-key       # assistente completo (tipo, tamanho, validade)
gpg --quick-generate-key "Maria Silva <maria@curso.lan>"   # padrão moderno (ed25519)
```

{{% notice info "Comando antigo" %}}
Em materiais antigos aparece `gpg --gen-key`. Ele ainda funciona, mas pede menos opções;
`--full-generate-key` permite escolher o algoritmo e a validade.
{{% /notice %}}

## Listando e trocando chaves

```bash
gpg --list-keys                  # chaves públicas no seu chaveiro
gpg --list-secret-keys           # suas chaves privadas
gpg --export --armor maria@curso.lan > maria.pub   # exporta a pública para enviar
gpg --import joao.pub            # importa a chave pública de outra pessoa
gpg --fingerprint joao@curso.lan # confira a impressão digital com o dono, por outro meio
```

## Cifrando e decifrando

```bash
gpg --encrypt --recipient joao@curso.lan relatorio.pdf   # gera relatorio.pdf.gpg
gpg --decrypt relatorio.pdf.gpg > relatorio.pdf          # quem tem a chave privada decifra
gpg --symmetric segredo.txt      # cifra só com senha, sem par de chaves
```

## Assinando e verificando

A assinatura prova **quem** gerou o arquivo e que ele **não foi alterado**:

```bash
gpg --detach-sign --armor script.sh       # gera script.sh.asc
gpg --verify script.sh.asc script.sh      # "Good signature from ..."
```

É assim que se verifica a imagem ISO de uma distribuição antes de instalá-la.

{{% notice warning %}}
Faça backup da chave privada (`gpg --export-secret-keys --armor`) em local seguro e gere um
certificado de revogação (`gpg --gen-revoke`). Perder a chave privada significa perder o acesso
a tudo que foi cifrado para ela.
{{% /notice %}}

## Exercício

1. Gere um par de chaves para você.
2. Exporte sua chave pública para `minha.pub`.
3. Cifre um arquivo para você mesmo, apague o original e decifre-o.
4. Assine um arquivo, altere uma letra dele e tente verificar a assinatura.

{{% expand "Ver solução" %}}
```bash
gpg --quick-generate-key "Seu Nome <voce@curso.lan>"
gpg --export --armor voce@curso.lan > minha.pub
echo "teste" > nota.txt
gpg --encrypt --recipient voce@curso.lan nota.txt && rm nota.txt
gpg --decrypt nota.txt.gpg > nota.txt
gpg --detach-sign --armor nota.txt
echo "x" >> nota.txt
gpg --verify nota.txt.asc nota.txt    # BAD signature
```
{{% /expand %}}
