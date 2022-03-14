---
title: "Por que você deve usar aspas SEMPRE"
description: >
  Não perca tempo pensando 🤔 "Será que tem problema deixar essa variável sem aspas?". Simplesmente use-as SEMPRE!
date: 2020-01-26T16:11:52-03:00
tags:
  - fundamentos
  - boas praticas
---

Esse post é para tentar convencê-los de que suas variáveis devem ser protegidas com aspas duplas **SEMPRE**. E quando eu digo **sempre**, é sempre **mesmo**. Até mesmo em `$(subshells)`.

OK... Tá bom. As vezes precisamos referenciar variáveis sem aspas, mas faça isso só quando for estritamente necessário. E quando isso acontecer, deixe um comentário falando sobre esta necessidade.

Insisto: Não fique perdendo tempo pensando "Uhmm... Será que tem problema se eu deixar essa variável sem aspas?". Simplesmente use-as. E quando _precisar não usá-las_, deixe isso claro através de um comentário.

**NOTA**: lembrando que estou levando em consideração o bash. Se você está usando outro shell o comportamento pode ser diferente, mas ainda assim eu recomendo seguir essa "regra".

## Motivos

Referências a variáveis sem usar aspas **irão bugar seu script**. Tenha essa certeza. Vamos ver alguns exemplos.

### o shell vai pensar que tem **mais** parâmetros do que você realmente está passando

```
#!/usr/bin/env bash
# fileInfo.sh
# informações sobre o arquivo passado como parâmetro
arquivo=$1
file $arquivo
```
```shell-session
$ # criando um arquivo cujo nome contem espaços
$ echo "dummy file" > "nome com espaços"
$ 
$ # veja o problema:
$ ./fileInfo.sh "nome com espaços" 
nome:    cannot open `nome' (No such file or directory)
com:     cannot open `com' (No such file or directory)
espaços: cannot open `espaços' (No such file or directory)
$ 
```

Use aspas para expandir `$@` ou `${array[@]}` para um loop `for`:

```
#!/usr/bin/env bash
# forLoops.sh
forSemAspas() {
    for word in $@; do
        echo word: $word
    done
}

forComAspas() {
    for word in "$@"; do
        echo "word: $word"
    done
}
```
```shell-session
$ . forLoops.sh 
$ forSemAspas um dois "dois e meio" tres
word: um
word: dois
word: dois
word: e
word: meio
word: tres
$ forComAspas um dois "dois e meio" tres
word: um
word: dois
word: dois e meio
word: tres
$ 
```

### o shell vai pensar que tem **menos** parâmetros do que você realmente está passando

Problemas com o `printf`:
```shell-session
$ arg1="um"
$ arg2="dois"
$ arg3="tres"
$ printf "1st: %s\n2nd: %s\n3rd: %s\n" $arg1 $arg2 $arg3
1st: um
2nd: dois
3rd: tres
$ 
$ # agora vamos deixar arg2 vazio
$ arg2=
$ 
$ printf "1st: %s\n2nd: %s\n3rd: %s\n" $arg1 $arg2 $arg3
1st: um
2nd: tres
3rd: 
$ # arg3 virou o segundo argumento! :(
$ 
$ # usando aspas, tudo fica deboas, veja:
$ printf "1st: %s\n2nd: %s\n3rd: %s\n" "$arg1" "$arg2" "$arg3"
1st: um
2nd: 
3rd: tres
$ 
```

## exceções

Observe que as exceções são muito bem específicas...

### quando você realmente quer uma lista de palavras separadas por espaços

Algumas vezes queremos uma lista de palavras bem separadinhas por espaços, por exemplo quando queremos que cada palavra seja um elemento distinto de um array:
```
array=( $* ) # OBS: necessário deixar sem aspas
```

No meu [artigo sobre _case convertion_](http://meleu.sh/tolower-toupper/) você verá um exemplo de como precisei de uma lista de palavras individualmente separadas por espaços para que cada uma fosse um elemento distinto de um array.


### quando estiver usando expressão regular dentro de um `[[ ... ]]`

Não use aspas quando sua variável for uma expressão regular que você quer usar com o `[[` e `=~`:
```shell-session
$ string='se curte shell, visite meleu.sh'
$ regex='meleu\.sh$'
$ # expressão regular para: "meleu.sh" seguido de final da linha
$ 
$ [[ "$string" =~ $regex ]] && echo sim || echo nao
sim
$ [[ "$string" =~ "$regex" ]] && echo sim || echo nao
nao
$ # expressão regular entre aspas é considerada uma string literal
```

## Palavras finais

Observe que na seção de motivos, eu elenquei casos de uso muito comuns de se ver em qualquer script. São situações que acontecem com frequencia. Enquanto as excessoes acontecem com uma frequência muito menor.

Portanto vou insistir pela quarta ou quinta vez:

Regra nº 1: **SEMPRE USE ASPAS**.

Regra nº 2: se violar a regra nº 1 se for estritamente necessário e mencione isso num comentário.


## Fontes

- http://mywiki.wooledge.org/Quotes
- http://wiki.bash-hackers.org/syntax/quoting