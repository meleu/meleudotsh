---
title: "explainshell: obtendo uma explicação rápida de uma linha de comando e cada um dos seus argumentos"
description: >
  Escreva uma linha de comando inteira (pode incluir pipes e redirecionamentos) e veja uma explicação de cada comando e cada um de seus argumentos.
tags:
  - links
  - codigo
date: 2020-01-14T15:53:34-03:00
---

Recentemente conheci mais um site que certamente é muito útil para iniciantes e até mesmo para usuários de linha de comando já iniciados: https://explainshell.com/

## conheça o https://explainshell.com

Na página inicial deste site tem um campo de texto onde você pode digitar uma linha de comando inteira (incluindo até mesmo `|` pipes e `<<` redirecionamentos `>`) e obter uma explicação do(s) comando(s) e de cada um dos argumentos utilizados.

Por exemplo, [clique neste link](https://explainshell.com/explain?cmd=find+%2F+-type+d+-name+deleteme+2%3E%2Fdev%2Fnull) para ver a explicação de `find / -type d -name deleteme 2>/dev/null`.

As informações dos comandos e suas respectivas opções são provenientes das manpages. Poderíamos consultar tudo isso diretamente no sistema, é verdade. Mas ter uma explicação breve somente das opções que estamos usando é uma comodidade atraente. Sem contar que ele também explica, pipes, redirecionamentos, etc.


## explainshell no terminal

OK. Legal. Tudo muito bonitinho... Mas só que para nós, amantes da telinha preta, ter que abrir o browser, entrar na página e fazer uma consulta dessas não tem muito cabimento, né mesmo?

A primeira coisa que pensei quando vi explainshell foi **"Cara! Seria muito massa se tivesse uma ferramenta dessas diretamente na linha de comando!"**

Dando uma olhada no [repositório com o código do explainshell.com](https://github.com/idank/explainshell), eu observei que várias pessoas também possuem este mesmo desejo de ter o explainshell diretamente na linha de comando. Mas infelizmente este recurso ainda não existe.

O mais próximo a isso que eu vi, foi um pequeno script (basicamente um PoC - _Proof of Concept_) criado por um dos usuários e [compartilhado em um gist](https://gist.github.com/shivansh/a05e3b4152fa75f6f398d423c295edb5), onde ele pega a saída de um `w3m -dump` e usa o `grep` para dar uma limpeza.

Tomei a liberdade de expandir o script acrescentando um pouco mais de robustez e a opção de ver a resposta no terminal ou abrir no browser.


### o script `explainshell`

Eis o meu código:

```
#!/usr/bin/env bash
# explainshell - faz uma consulta a explainshell.com para
#                obter a explicação da linha de comando.
#
# DEPENDÊNCIA: w3m, grep, xdg-open (se quiser abrir no browser)
#
# inspiração:
# https://gist.github.com/shivansh/a05e3b4152fa75f6f398d423c295edb5

readonly MAIN_URL='https://explainshell.com/explain?cmd='

# se quiser abrir no browser, mude para USE_BROWSER='true'
readonly USE_BROWSER='false'

# explicação desta função em https://meleu.sh/urlencode/
urlencode() {
  local LC_ALL=C
  local string="$*"
  local length="${#string}"
  local char

  for (( i = 0; i < length; i++ )); do
    char="${string:i:1}"
    if [[ "$char" == [a-zA-Z0-9.~_-] ]]; then
      printf "$char" 
    else
      printf '%%%02X' "'$char" 
    fi
  done
  printf '\n' # opcional
}


main() {
  if [[ -z "$1" ]]; then
    echo "
ERRO: falta argumento
Uso: $0 'linha de comando'" >&2
    exit 1
  fi

  local url="${MAIN_URL}$(urlencode "$*")"

  if [[ "$USE_BROWSER" == 'true' ]]; then
    xdg-open "$url"
    exit "$?"
  fi

  response=$(w3m -dump "$url")
  cat -s <(grep -v -e explainshell -e • -e □ -e "source manpages" <<< "$response")
}

main "$@"
```

### explicando o código

O script começa com duas variáveis globais. A `MAIN_URL` é a URL que usaremos para fazer a pesquisa. E a `USE_BROWSER` é para definir se você quer obter o resultado da pesquisa no browser ou direto no terminal. Deixei `USE_BROWSER='false'` como default, portanto se quiser abrir no browser, mude para `true` (se quiser exercitar um pouco, adicione uma opção de invocar o script com `--browser`, por exemplo).

Em seguida temos a função `urlencode()`, que foi explicada em detalhes [num post anterior](http://meleu.sh/urlencode).

Na função `main()` nos certificamos se o usuário passou o comando que queremos pesquisar. Se tiver passado, vamos transformar a string em uma URL válida chamando a função `urlencode`.

Em seguida verificamos se a preferência é abrir no browser. Em caso positivo, a consulta será aberta direto no browser. Fazemos isso com o `xdg-open`, mas se você estiver no MacOS, mude para `open`. E se tiver no Windows/[Cygwin](https://cygwin.com), mude para `cygstart` (eu falo um pouco destes comandos [num post anterior](http://meleu.sh/abrir-qualquer-arquivo/)).

Se `USE_BROWSER` não for igual à `true`, significa que a consulta será feita direto no terminal utilizando o método exposto pelo usuário [@shivansh](https://gist.github.com/shivansh/a05e3b4152fa75f6f398d423c295edb5) com o `w3m` (certifique-se que esse programa está instalado no seu sistema).


A saída desse comando no terminal não é lá tão boa, como podemos ver no exemplo a seguir para a explicação de um simples `ls -lah`.

```shell-session
$ ./explainshell ls -lah

ls(1)

-lah

list directory contents

-l     use a long listing format

-a, --all
       do not ignore entries starting with .

-h, --human-readable
       with -l, print sizes in human readable format (e.g., 1K 234M 2G)

```

Nota-se que a explicação do `ls` fica distante do comando. E se o comando for grande, com pipes... Essa confusão vai ficar ainda maior.

Portanto utilizar para obter um resultado legível com esse script, só se for uma de comando pequeno.

Para comandos grandes, é mais interassante visualizar no site mesmo.


## Fontes

- https://explainshell.com/
- https://github.com/idank/explainshell
- https://gist.github.com/shivansh/a05e3b4152fa75f6f398d423c295edb5
