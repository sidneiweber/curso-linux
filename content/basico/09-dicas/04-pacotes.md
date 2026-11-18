+++
title = "Descobrindo pacotes"
weight = 4
description = "apt-file: qual pacote contém um arquivo ou comando."
+++

Quer usar um comando (por exemplo, `import`) mas não sabe qual pacote o fornece?
O `apt-file` pesquisa nos arquivos de **todos os pacotes**, inclusive os não instalados.

```bash
sudo apt install apt-file
sudo apt-file update              # atualiza a base de dados

apt-file search bin/import        # → imagemagick: /usr/bin/import
apt-file list htop                # lista os arquivos de um pacote
```

| Comando | Pesquisa em |
|---|---|
| `dpkg -S arquivo` | pacotes **instalados** |
| `apt-file search arquivo` | **todos** os pacotes do repositório |

{{% notice tip %}}
Muito útil ao compilar programas: quando falta um cabeçalho como `zlib.h`, rode
`apt-file search zlib.h` para descobrir qual pacote `-dev` instalar.
{{% /notice %}}

No Ubuntu, ao digitar um comando inexistente, o próprio terminal sugere o pacote
(*command-not-found*). No Fedora, o equivalente é `dnf provides */import`.
