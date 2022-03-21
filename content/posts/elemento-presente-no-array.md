---
title: "Como checar se um array contém um determinado elemento"
date: 2020-01-03T05:00:59-03:00
tags:
  - codigo
  - bash-puro
cover:
  image: "img/elemento-presente-no-array.png"
  alt: "output de covid.sh"
---

Quando estamos trabalhando com arrays em shell scripts é comum termos situações onde queremos saber se um determinado elemento está presente no array.

O bash não tem um recurso específico para isso, portanto temos que arrumar um outro jeito. Neste artigo estudaremos três maneiras de alcançar esse objetivo.

## Método 1: infalível, porém "custoso"

A maneira que logo vem a mente é percorrer todo o array através de um loop e checar se o elemento está presente lá. No código a seguir veremos a função `elementInArray1`:

```
# uso: elementInArray1 elemento item1 item2 itemN
elementInArray1() {
  local element="$1"
  local array=("${@:2}")
  for temp in "${array[@]}"; do
    [[ "$temp" == "$element" ]] && return 0
  done
  return 1
}
```

Essa função considera que `$1` (o primeiro argumento) é o elemento que você quer checar se está no array, e que todos argumentos que vêm depois `${@:2}` (do segundo argumento em diante) são os elementos do array.

Através do loop `for` vamos testando os elementos um por um, comparando com o valor que queremos encontrar. Caso ele encontre, já vai retornar o sucesso (`return 0`) interrompendo a checagem.

Se chegar até o final do loop significa que não encontrou o valor que queremos, portanto retorna insucesso (`return 1`).

Vejamos essa função em ação:

```
$ # macete do alias: https://meleu.sh/dica-result
$ alias result='echo verdadeiro || echo falso'
$ 
$ # a função está salva no arquivo elementInArray.sh
$ . elementInArray.sh
$ term='melancia'
$ fruits=(pera uva maçã laranja kiwi)
$ elementInArray1 "$term" "${fruits[@]}" && result 
falso
$ 
$ term='uva'
$ elementInArray1 "$term" "${fruits[@]}" && result
verdadeiro
$ 
$ term='lar'
$ elementInArray1 "$term" "${fruits[@]}" && result
falso
```

## Método 2: mais eficiente, porém com uma limitação

Legal, mas acho que conseguimos fazer um método um pouco mais eficiente. Onde podemos nos aproveitar do _globbing_ do bash e fazer essa checagem com apenas um teste.

Vamos começar a função `elementInArray2` assim:

```
# OBS: código incompleto
elementInArray2() {
  local element="$1"
  local array=("${@:2}")
  [[ "${array[@]}" == *"$element"* ]]
}
```

O teste que está ocorrendo ali, pega todos os elementos do array e o transforma em uma só string e compara com `$element` sendo que aqueles `*` asteriscos dizem que ele pode ter qualquer coisa antes e depois.

Vamos dar uma olhada se ele vai funcionar legal:

```shell-session
$ # macete do alias: https://meleu.sh/dica-result
$ alias result='echo verdadeiro || echo falso'
$ 
$ # a função está salva no arquivo elementInArray.sh
$ . elementInArray.sh
$ term='melancia'
$ fruits=(pera uva maçã laranja kiwi)
$ elementInArray2 "$term" "${fruits[@]}" && result 
falso
$ 
$ term='uva'
$ elementInArray2 "$term" "${fruits[@]}" && result
verdadeiro
$ 
$ term='lar'
$ elementInArray2 "$term" "${fruits[@]}" && result
verdadeiro
$ # OPA! ERRO! 'lar' não está presente no array!
```

A princípio a função funcionou de acordo com o esperado, no entanto no último exemplo gerou um falso positivo.

A função encontrou `lar` mesmo que o array não tenha elemento algum com este valor. Como você já deve ter percebido, isso ocorreu por conta do valor `laranja`.

Vejamos uma maneira de contornar isso:

```
elementInArray2() {
  local element="$1"
  local array=("${@:2}")
  [[ "${array[@]}" == *" $element "* ]]
}
```

A única diferença aqui é que o foi adicionado um espaço antes e depois de `$element`. Vamos aos testes:

```shell-session
$ # macete do alias: https://meleu.sh/dica-result
$ alias result='echo verdadeiro || echo falso'
$ 
$ # a função está salva no arquivo elementInArray.sh
$ . elementInArray.sh
$ term='melancia'
$ fruits=(pera uva maçã laranja kiwi)
$ elementInArray2 "$term" "${fruits[@]}" && result
falso
$ 
$ term='uva'
$ elementInArray2 "$term" "${fruits[@]}" && result
verdadeiro
$ 
$ term='lar'
$ elementInArray2 "$term" "${fruits[@]}" && result
falso
$ 
$ fruits=(pera 'uva passa' maçã laranja kiwi)
$ # removi 'uva' substituindo por 'uva passa'
$ term='uva' # vamos ver se 'uva' ainda está no array
$ elementInArray2 "$term" "${fruits[@]}" && result
verdadeiro
$ # ERRO! Isso não deveria ter ocorrido! :(
```

A função atendeu legal, porém mais uma vez gerando um falso positivo. Ela acusa que `uva` está presente no array mesmo quando não há elemento algum contendo **apenas** `uva`. Obviamente isso ocorreu devido ao elemento `uva passa`.

Poderíamos ir adicionando mais gambiarras para contornar este problema, mais sinceramente, eu não gosto muito de gambiarras. Principalmente se for tornar o código de difícil leitura.

Vou me dar por satisfeito com esta função assim mesmo e tomar o cuidado de usá-la somente com arrays cujos elementos não contenham espaços.

O máximo que eu faria é remover a atribuição das variáveis locais para deixar a função com apenas uma linha (prejudica um pouco a legibilidade, mas... pô! é só uma única linha!).

```
elementInArray2() {
  [[ "${@:2}" == *" $1 "* ]]
}
```

## Método 3: a solução "perfeita", mas depende do `extglob` ativado

Este método eu vi [num grupo de telegram sobre shell script](https://t.me/shellscript_x) (postagem do [SHAMAN](https://github.com/shellscriptx) e depois com um toque de requinte do [Robson Alexandre](https://github.com/robsonalexandre)) e achei muito eficiente. Até agora não consegui ver um cenário onde esse método falha, mas como não fiz testes ostensivos, digo que é a solução "perfeita" entre aspas

Neste método usaremos um recurso não muito convencional do bash: `extglob`. Segundo o [Greg's Wiki](http://mywiki.wooledge.org/BashFAQ/061) (uma das minhas fontes de conhecimento sobre bash favoritas) esse recurso foi adicionado no bash 2.02, em 1998 (mais de 20 anos). Portanto, usar essa solução parece ser portável o suficiente.

Para utilizar `extglob` ele precisa estar habilitado. Se ele não estiver habilitado por padrão, basta fazer um `shopt -s extglob`.

Uma vez habilitado, podemos (dentre outras coisas) buscar uma string dentro de uma lista onde cada elemento é separado por um `|` pipe. Exemplo:

```
[[ $element == @(element1|element2|elementN) ]]
```

Um rápido exemplo prático:

```shell-session
$ # macete do alias: https://meleu.sh/dica-result
$ alias result='echo verdadeiro || echo falso'
$ 
$ [[ um == @(um|dois|tres) ]] && result
verdadeiro
$ [[ quatro == @(um|dois|tres) ]] && result
falso
```

Portanto, para aplicar essa técnica no nosso propósito de checar se um elemento está presente no array, primeiro teremos que pegar os elementos do array e separá-los com um `|` pipe.

Para isso vamos usar o `joinBy()` que mostrei e expliquei como funciona [num outro post](http://meleu.sh/join-bash) (e que repito a seguir).

O código fica assim:

```
joinBy() {
  local IFS="$1"
  echo "${*:2}"
}

# se extglob não estiver habilitado, basta executar:
# shopt -s extglob
elementInArray3() {
  local element="$1"
  local array=("${@:2}")
  [[ "$element" == @($(joinBy '|' "${array[@]//|/\\|}")) ]]
}
```

Um trechinho que pode parecer confuso é o `${array[@]//|/\\|}`, mas eu explico.

Estamos usando este esquema: `${variavel//padrao/substituto}`.

Só que no nosso caso o padrão é `|` e o substituto é `\\|`. Portanto uma string como `essa|aqui` vai virar `essa\|aqui`.

Fazemos isso para "escapar" o `|` pipe e fazer com o que ele não seja considerado como um separador quando estivermos usando o esquema de `@(...|...|...)`

Agora vejamos esse método em ação:

```shell-session
$ # macete do alias: https://meleu.sh/dica-result
$ alias result='echo verdadeiro || echo falso'
$ 
$ # a função está salva no arquivo elementInArray.sh
$ . elementInArray.sh
$ term='melancia'
$ fruits=(pera uva maçã laranja kiwi)
$ elementInArray3 "$term" "${fruits[@]}" && result
falso
$ 
$ term='uva'
$ elementInArray3 "$term" "${fruits[@]}" && result
verdadeiro
$ 
$ term='lar'
$ elementInArray3 "$term" "${fruits[@]}" && result
falso
$ 
$ fruits=(pera 'uva passa' maçã laranja kiwi)
$ # removi 'uva' substituindo por 'uva passa'
$ term='uva' # vamos ver se 'uva' ainda está no array
$ elementInArray3 "$term" "${fruits[@]}" && result
falso
$ 
$ 
$ # agora testando com '|' pipes dentro do array
$ array=(um dois 'tres|quatro' 'cinco seis')
$ elementInArray3 dois "${array[@]}" && result
verdadeiro
$ 
$ elementInArray3 'tres' "${array[@]}" && result
falso
$ 
$ elementInArray3 'tres|quatro' "${array[@]}" && result
verdadeiro
$ 
$ elementInArray3 'cinco' "${array[@]}" && result
falso
$ 
$ elementInArray3 'cinco seis' "${array[@]}" && result
verdadeiro
```

Como podemos ver, é um método bastante robusto.

Mais uma vez agradeço aos companheiros [SHAMAN](https://github.com/shellscriptx) e [Robson Alexandre](https://github.com/robsonalexandre) pelo conhecimento compartilhado.


## Resumo

1. Se quer um método infalível, use a versão com loop.
2. Se tem certeza que seu array não possui elementos contendo espaço, use a versão mais eficiente.
3. Se não vê problemas em habilitar `extglob`, use esta versão!
4. Agora a melhor recomendação: **use o que foi mostrado aqui como inspiração e crie sua própria solução!**


## Fontes

- https://github.com/dylanaraps/pure-bash-bible
- https://mywiki.wooledge.org/glob
- https://mywiki.wooledge.org/BashFAQ/061
- https://www.tldp.org/LDP/abs/html/parameter-substitution.html#PSGLOB
- https://www.gnu.org/software/bash/manual/html_node/Pattern-Matching.html
- https://stackoverflow.com/a/15394738/6354514
- https://stackoverflow.com/a/9429887/6354514
- [grupo de shell script do telegram](https://t.me/shellscript_x)
- `man bash` na seção de "Pattern Matching"
