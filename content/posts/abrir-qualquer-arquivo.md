---
title: "Como abrir qualquer arquivo no programa correto a partir da linha de comando"
description: >
  No ambiente gráfico basta dar um duplo clique no arquivo que ele abre no programa correto. Veja como obter o mesmo resultado na linha de comando.
tags:
  - codigo
date: 2020-01-07T15:41:59-03:00
cover:
  image: "img/abrir-qualquer-arquivo.png"
  alt: "xdg-open"
---

Um recurso que facilita muito a vida do usuário quando usando um gerenciador de arquivos no ambiente gráfico é o fato de poder dar um duplo clique no ícone e o sistema já saber qual programa utilizar para abrí-lo.

## Conhecendo o `xdg-open`

Quando estamos utilizando o terminal **dentro de um ambiente gráfico** podemos ter uma funcionalidade similar usando o comando `xdg-open` em sistemas Linux.

Equivalentes ao comando `xdg-open` em outros ambientes:

- MacOS: `open`
- Windows/[Cygwin](https://cygwin.com/): `cygstart`
- Android/[termux](https://termux.com/): `xdg-open`.

Nos exemplos abaixo vou usar o `xdg-open`, mas se você estiver em outro ambiente, substitua-o pelo comando equivalente.

Se tiver com uma janelinha do terminal aberta, basta usar `xdg-open arquivo.ext` que o seu ambiente desktop vai detectar qual é o programa padrão para abrir o `arquivo.ext`.

E uma coisa interessante é que isso também serve para URLs, fazendo com que o seu navegador padrão seja aberto tentando visitar a URL que passada como parâmetro. Exemplo:

```sh
xdg-open 'https://meleu.sh/'
```

Eis uma tradução livre da descrição que vemos na própria manpage do `xdg-open`:

> xdg-open abre um arquivo ou URL na aplicação preferida do usuário. Se uma URL for fornecida, a URL será aberta no browser padrão. Se um arquivo for fornecido, o arquivo será aberto na aplicação padrão para arquivos daquele tipo.

A disponibilidade deste comando vai depender do ambiente Desktop que você está utilizando e se ele segue os padrões do [freedesktop.org](https://www.freedesktop.org/).

O freedesktop é um projeto focado em criar especificações para facilitar a interoperabiliade e compartilhamento de tecnologia para sistemas gráficos e de desktop de código aberto.

Uma pequena (e incompleta) lista dos ambientes desktop que aderem aos padrões do freedesktop pode ser encontrada aqui: [https://www.freedesktop.org/wiki/Desktops/](https://www.freedesktop.org/wiki/Desktops/)

Muitas das especificações produzidas pelo freedesktop recebem o nome XDG, uma sigla para _Cross-Desktop Group_. E muitos aplicativos feitos para implementar estas especificações começam com o prefixo `xdg-`. Experimente ir no seu terminal e digitar `xdg-` e pressionar TAB TAB, que você verá uma lista de aplicativos disponíveis.

Se tiver mais interesse nesse assunto, siga os links da seção [Fontes](#fontes).

## Bônus

Vou aproveitar o tema e mostrar uma breve função onde usamos a variável _builtin_ `$OSTYPE`, específica do bash, para detectar o Sistema Operacional. Outra opção é utilizar `uname`, mas eu gosto de usar soluções em [bash puro](http://meleu.sh/tags/bash-puro). :)

Quando você chama a função `openFile`, ela vai detectar em qual Sistema Operacional você está e executa o comando correspondente.

É um exemplo bem simples. Se precisar de algo mais robusto é preciso trabalhar um pouco mais nesse código.

```
# $OSTYPE é uma variável embutida do bash pode
# ser uma maneira interessante de detectar o SO
openFile() {
  local args="$@"

  case "$OSTYPE" in
    "cygwin"*)
      cygstart "$args"
      ;;
    "darwin"*) # MacOS
      open "$args"
      ;;
    *)
      xdg-open "$args"
      ;;
  esac
}
```

Algumas opções válidas para `$OSTYPE` que vi em uma [postagem no GitHub](https://github.com/dylanaraps/neofetch/issues/433#issue-188679046):

| Sistema Operacional | `$OSTYPE` |
|----|-----------|
| Linux | `linux-gnu`
| Cygwin | `cygwin`
| bash no Windows 10 | `linux-gnu`
| OpenBSD | `openbsd*`
| FreeBSD | `FreeBSD`
| NetBSD | `netbsd`
| Mac OS | `darwin*`
| iOS | `darwin9`
| Solaris  | `solaris*`
| Android (termux) | `linux-android*`
| Android | `linux-gnu`
| Haiku OS | `haiku`


## Fontes

- https://wiki.archlinux.org/index.php/Xdg-utils
- `man xdg-open`
- https://www.freedesktop.org/wiki/Specifications/
- `man bash`, descrição de `OSTYPE`
- https://stackoverflow.com/questions/394230/how-to-detect-the-os-from-a-bash-script
