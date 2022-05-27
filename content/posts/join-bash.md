---
title: "Como juntar elementos de um array separando-os com um caracter qualquer"
description: >
  Algumas linguagens (como JavaScript e PHP) possuem uma função 'join()' (ou 'implode()') que serve para juntar os elementos de um array separando-os por um caracter. Vamos ver aqui uma maneira simples de fazer isso em bash.
date: 2019-12-31T14:46:31-03:00
cover:
  image: "img/join-bash.png"
  alt: "função joinBy()"
---

Eu estava precisando juntar os elementos do array em uma só string usando um `|` como separador.

Em linguagens como JavaScript e PHP isso é obtido através de funções/métodos nativos, tipo `join()` ou `implode()`. Em bash teremos que implementar nós mesmos.

## Código

Uma rápida googlada me levou a uma solução [bem simples e bacana](https://stackoverflow.com/a/9429887/6354514). Mostro aqui a minha versão:

```
joinBy() {
  local IFS="$1"
  echo "${*:2}"
}
```

Agora vamos ver esse código em ação:

```
$ . joinBy.sh
$ joinBy , zacarias mussum 'didi mocó' 'dedé santana'
zacarias,mussum,didi mocó,dedé santana
$ joinBy | zacarias mussum 'didi mocó' 'dedé santana'
zacarias: command not found
$ # Ooops! Precisamos proteger o pipe com 'aspas'
$ joinBy '|' zacarias mussum 'didi mocó' 'dedé santana'
zacarias|mussum|didi mocó|dedé santana
```

## Explicando

Checando a manpage do bash, na seção de "Special Parameters", conseguimos ver a resposta para isso. Lá vemos a explicação do `*` asterisco e do `@` arroba no contexto de parâmetros da linha de comando (`$1`, `$2`, etc.), mas a mesma lógica se aplica a expansão de arrays.

Este é um trecho da descrição da expansão feita com `*` asterisco (tradução livre feita por mim):

> Quando a expansão ocorre entre aspas duplas, expande para uma única palavra com o valor de cada parâmetro separado pelo primeiro caractere da variável especial *`IFS`*. Isto é, `"$*"` é equivalente a `"$1c$2c..."`, onde `c` é o primeiro caractere da variável *`IFS`*.

E a seguir um trecho da descrição da expansão feita com `@` arroba:

> Quando a expansão ocorre entre aspas duplas, cada parâmetro expande para uma palavra separada [por um espaço]. Isto é `"$@"` é equivalente a `"$1" "$2" ...`

**Resumo:**
- `@` arroba: expande separando com um espaço.
- `*` asterisco: expande separando com o primeiro caractere do `IFS`.

Devido a esta propriedade, se tentarmos a função `joinBy()` usando `@` arroba no lugar de `*` asterisco (`echo "${@:2}"`), não obteremos o resultado desejado.

> **Atualização**
>
> O Blau Araujo fez um vídeo explicando no detalhe essas diferenças entre o `*` e o `@`. Confira aqui: [# Bash: expansões dos parâmetros `@` e `*` - quais são as diferenças?](https://youtu.be/x6Vv9Lb2WWQ)

## Fontes

- https://stackoverflow.com/a/9429887/6354514
- `man bash` na seção de "Special Parameters"