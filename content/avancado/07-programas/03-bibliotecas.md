+++
title = "Bibliotecas compartilhadas"
weight = 3
description = "ldd, ldconfig, LD_LIBRARY_PATH e erros de biblioteca."
+++

Programas reaproveitam código de **bibliotecas compartilhadas** (`.so`, *shared objects*),
carregadas na hora da execução pelo *dynamic linker* (`ld-linux`). Uma correção de segurança
na `libssl` vale para todos os programas que a usam, sem recompilar nenhum deles.

## Do que um programa depende?

```bash
ldd /usr/bin/curl
#   libcurl.so.4 => /lib/x86_64-linux-gnu/libcurl.so.4
#   libssl.so.3  => /lib/x86_64-linux-gnu/libssl.so.3
#   ...
```

## Onde o sistema procura

1. Caminhos gravados no próprio programa (*RPATH/RUNPATH*).
2. A variável `LD_LIBRARY_PATH`.
3. O cache `/etc/ld.so.cache`, gerado pelo `ldconfig` a partir de `/etc/ld.so.conf.d/*.conf`.
4. `/lib` e `/usr/lib`.

```bash
ldconfig -p | grep libssl       # bibliotecas no cache
```

## Adicionando um diretório de bibliotecas

Ao compilar algo em `/usr/local` ou `/opt`, as bibliotecas podem não ser encontradas:

```text
programa: error while loading shared libraries: libfoo.so.1: cannot open shared object file
```

Solução permanente:

```bash
echo '/opt/foo/lib' | sudo tee /etc/ld.so.conf.d/foo.conf
sudo ldconfig
```

Solução temporária (para testes):

```bash
LD_LIBRARY_PATH=/opt/foo/lib ./programa
```

## Versões e sonames

```bash
ls -l /lib/x86_64-linux-gnu/libssl.so*
# libssl.so.3 -> arquivo real
```

O número após `.so` é a versão de **ABI**: programas compilados para `libssl.so.3` não
funcionam com `libssl.so.1.1`. Por isso as distribuições mantêm versões diferentes em
pacotes diferentes (`libssl3`, `libssl1.1`).
