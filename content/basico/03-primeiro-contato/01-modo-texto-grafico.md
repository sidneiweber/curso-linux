+++
title = "Modo texto vs. modo gráfico"
weight = 1
description = "Terminal, consoles virtuais e ambiente gráfico."
+++

O Linux pode ser usado de duas formas:

- **Modo gráfico (GUI)**: janelas, ícones e mouse — fornecido por um *ambiente de desktop*
  (GNOME, KDE Plasma, XFCE…);
- **Modo texto (CLI)**: comandos digitados em um **shell** (normalmente o **Bash**).

Servidores geralmente rodam **apenas em modo texto**, por isso dominar o terminal é essencial.

## Acessando o modo texto

- **Emulador de terminal** dentro do ambiente gráfico: atalho `Ctrl` + `Alt` + `T` (na maioria das distros);
- **Consoles virtuais (TTYs)**: `Ctrl` + `Alt` + `F3` … `F6`. Para voltar ao gráfico,
  normalmente `Ctrl` + `Alt` + `F1` ou `F2`.

## O prompt

```text
maria@notebook:~$
```

| Parte | Significado |
|---|---|
| `maria` | usuário logado |
| `notebook` | nome da máquina (*hostname*) |
| `~` | diretório atual (`~` = diretório pessoal) |
| `$` | usuário comum — o **root** usa `#` |
