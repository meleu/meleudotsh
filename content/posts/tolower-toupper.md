---
title: "Como converter de maiúsculas para minúsculas (e vice-versa) com bash"
description: >
  Em várias linguagens nós contamos com toLower()/toUpper() para converter strings para minúsculas/maiúsculas. No bash podemos ter essa mesma funcionalidade.
tags:
  - codigo
  - bash-puro
  - fundamentos
date: 2020-01-25T16:09:01-03:00
cover:
  image: "img/tolower-toupper.png"
  alt: "função camelCase()"
---

Desde a versão 4.0 do bash (lançada em 2009) nós temos disponível alguns operadores para lidar com _case conversion_.

Nos exemplos a seguir imagine que `var` contém uma string:

- `${var,,}`: conteúdo de `var` todo em minúsculas
- `${var^^}`: conteúdo de `var` todo em maiúsculas
- `${var,}`: primeira letra de `var` minúscula
- `${var^}`: primeira letra de `var` maiúscula
- `${var~~}`: inverte de maiúsculo para minúsculo, ou vice-versa, todo o conteúdo de `var`
- `${var~}`: inverte de maiúsculo para minúsculo, ou vice-versa, a primeira letra de `var`


## Funções Para _Case Conversion_

De posse desse conhecimento, vamos fazer nossas versões bash de `toLower()` e `toUpper()` e, por que não?, de `camelCase()`.

```
#!/usr/bin/env bash
# lowUpCamel.sh

toLower() {
  echo "${*,,}"
}

toUpper() {
  echo "${*^^}"
}

camelCase() {
  local array=( ${*,,} )      # 1. tudo minúsculo. OBS: sem "aspas"
  local string="${array[*]^}" # 2. primeiras letras maiúsculas
  echo "${string// /}"        # 3. remove espaços
}
```

Tanto em `toLower()` quanto em `toUpper()` estamos pegando todos os argumentos usando o `$*`, só que já aproveitamos para fazer a conversão usando os operadores que aprendemos aqui neste artigo.

Agora analisemos a função `camelCase()`.

Primeiramente pegamos todos os argumentos (vindo de `$*`) e armazenamos em `array` já convertendo tudo para minúsculo. É importante que a expressão não esteja entre aspas, para que cada string sem espaços seja um elemento distinto no array.

Em seguida armazenamos em `string` cada elemento de `array` já convertendo a primeira letra de cada elemento para maiúscula.

E por último usamos um `echo` para mostrar na tela o conteúdo de `string` só que removendo todos os espaços.

**Nota**: se você é do tipo preciosista e quer argumentar que num camel case "de verdade" a primeira letra tem que ser minúscula, fica então como um exercício reescrever a função `camelCase()` para lidar com isso. [Aqui está uma dica de leitura](https://wiki.bash-hackers.org/syntax/arrays#getting_values) para realizar essa tarefa. ;)


## Aplicação Prática: Renomeando arquivos

Na minha opinião a opção de converter tudo para minúsculo com `${var,,}` é especialmente útil. Isso pelo simples fato de ser comum querermos renomear arquivos para que seus nomes fiquem todos em minúsculo.

Imaginemos por exemplo aqueles arquivos de fotos tiradas pela máquina fotográfica ou celular: `IMG_20200125_102030.JPG`.

Para renomear todos os arquivos com nomes desse tipo para sua versão minúscula, basta fazermos:

```
for arquivo in *.JPG; do mv "$arquivo" "${arquivo,,}" ; done
```

Também podemos criar um script `mvlower.sh` que simplesmente recebe uma lista de arquivos e renomeia cada um deles para sua versão com letras minúsculas:

```
#!/usr/bin/env bash
# mvlower.sh

mvLower() {
  local filepath
  local dirpath
  local filename

  for filepath in "$@"; do
    # OBS: temos que preservar o path do diretório!
    dirpath=$(dirname "$filepath")
    filename=$(basename "$filepath")
    mv "$filepath" "${dirpath}/${filename,,}"
  done
}

mvLower "$@"
```

## Fontes

- release notes do bash 4.0: https://lists.gnu.org/archive/html/bug-bash/2009-02/msg00164.html
- compilação das principais features adicionadas ao bash em cada versão: https://mywiki.wooledge.org/BashFAQ/061
- explicação para a técnica de remover espaços usada na função `camelCase`: https://www.tldp.org/LDP/abs/html/parameter-substitution.html#PSGLOB
- tive a ideia para esse post lendo o livro "bash Cookbook" de Carl Albing e JP Vossen.
