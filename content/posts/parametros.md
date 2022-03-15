---
title: "Como lidar com parâmetros passados na linha de comando em shell scripts"
description: >
  Veremos como o bash recebe dados diretamente da linha de comando, e iremos além: veja a peculiaridade do '$0' (e receba um easter egg); entenda a real diferença entre o '$*' e o '$@'; e veja  como usar os comando 'shift' e 'set' para manipular os parâmetros.
tags:
  - fundamentos
date: 2020-02-02T16:14:09-03:00
cover:
  image: "parametros.png"
  alt: "parametros no shell script"
---

O shell script é uma "linguagem" muito permissiva. Não é necessário muito conhecimento pra você começar a fazer algumas coisas legais. Isso é bom, fazer coisas legais é divertido, mas também tem um perigo embutido: você pode acabar ir levando adiante uma prática ruim que funciona em determinado cenário mas que em outro pode trazer consequências indesejadas (a [não utilização de aspas duplas pra proteger suas variáveis](http://meleu.sh/aspas-sempre/) é um exemplo dessas práticas ruins).

Numa tentativa de difundir essa cultura das boas práticas de programação em shell script, vou começar a escrever também artigos abordando os conceitos mais básicos e fundamentais do shell script.

Pra começar, hoje falarei sobre parâmetros da linha de comando. Mesmo que você já conheça esse tema tão fundamental para o shell script, garanto que vai tirar algo de útil desse artigo. Pois vamos entender no detalhe a real diferença entre `$*` e `$@`, o que o `$0` tem de tão peculiar, e também veremos o comando `shift` e o `set`.


## Parâmetros

Um parâmetro da linha de comando é qualquer coisa vem após o nome do comando. Exemplo:

```shell-session
$ ls -l arquivo.txt
```

Na linha de comando acima, o `ls` é o comando, o `-l` é o primeiro parâmetro (as vezes chamado de argumento), e o `arquivo.txt` é o segundo parâmetro.

Situação similar acontece quando alguém executa nosso script assim:
```shell-session
$ ./meuscript.sh um dois tres
```

Aqui `./meuscript.sh` é o comando, `um` é o primeiro parâmetro, `dois` é o segundo, e `tres` é o terceiro.

Nós temos acesso a estes parâmetros através de algumas variáveis "especiais". Digo especiais pois elas não são exatamente como as variáveis que normalmente usamos.

Pra começar elas não obedecem as regras de nomeclatura de variáveis, pois elas usam números; e também nós não podemos mudar o valor destas variáveis pelas vias "tradicionais" de atribuição (só conseguimos alterar seus valores com a ajuda de comandos dedicados a isso, como o `shift` e o `set`, que veremos adiante).

Veja esta relação:

parâmetro | significado
--- | ---
`$0`              | SEMPRE será o nome (ou o caminho) do script<br> chamado na linha de comando (mais detalhes abaixo)
`$1`, `$2`, a `$9` | `$1`: 1º parâmetro, `$2`: 2º parâmetro, até o `$9`.<br> Passados para o script ou função.
`${10}`, `${11}`, ... | quando o número do parâmetro possui mais<br> de um dígito é necessário o uso das chaves.
`$*`              | todos os parâmetros passados para o script ou função<br> em uma única string (mais detalhes abaixo)
`$@`              | todos os parâmetros passados para o script ou função<br> cada um em strings separadas.
`$#`              | número de parâmetros (sem contar com o `$0`).

Pra ficar mais claro, nada melhor do que um exemplo:

```
#!/usr/bin/env bash
# parametros.sh

echo "Nome do script: $0"
echo "Número total de parâmetros: $#"
echo "Primeiro parâmetro: $1"
echo "Segundo parâmetro: $2"
echo "Décimo quinto parâmetro: ${15}"
echo "Todos os parâmetros: $*"
```

Vamos passar o alfabeto inteiro para esse script e dar uma olhada:

```shell-session
$ ./parametros.sh a b c d e f g h i j k l m n o p q r s t u v w x y z
Nome do script: ./parametros.sh
Número total de parâmetros: 26
Primeiro parâmetro: a
Segundo parâmetro: b
Décimo quinto parâmetro: o
Todos os parâmetros: a b c d e f g h i j k l m n o p q r s t u v w x y z
```

## Parâmetros para funções

Os parâmetros funcionam do mesmo jeito para funções. Porém com apenas uma nuance: `$0` **não** será o nome da função, e sim o nome do comando shell script (ou do shell, se estiver direto no prompt).

Vejamos um exemplo que deixa isso bastante claro:

```
#!/usr/bin/env bash
# parametroParaFuncao.sh

minhaFuncao() {
  echo "---> argumento zero: $0"
  echo "---> 3º argumento (dentro de $FUNCNAME): $3"
}

echo "3º argumento (antes da função): $3"
minhaFuncao a b c d e f g h i j k l m n o ...
echo "3º argumento (depois da função): $3"
```

Em execução:

```shell-session
$ ./parametroFuncao.sh um dois tres
3º argumento (antes da função): tres
---> argumento zero: ./parametroFuncao.sh
---> 3º argumento (dentro de minhaFuncao): c
3º argumento (depois da função): tres
```

Observe as duas linhas que começam com `--->`. Note como que dentro de `minhaFuncao` o `$0` expandiu para o nome do script. E, na linha seguinte, como `$3` expandiu para `c`, que foi o terceiro argumento passado para `minhaFuncao` (enquanto que o terceiro argumento passado para o script foi `tres`).

Observe também um _easter egg_ que eu coloquei ali pra vocês: `$FUNCNAME` expande para o nome da função que está sendo executada (na verdade `FUNCNAME` faz um pouco mais que isso, mas isso é papo pra outro artigo).


## Diferença entre `$*` e `$@`

Conforme dito na tabela acima, o `$*` significa todos os parâmetros em uma única string, e o `$@` significa todos os parâmetros, cada um em strings separadas.

No script a seguir veremos isso bem claramente:

```
#!/usr/bin/env bash
# testargs.sh
#
# Ao executar este script use alguns parametros. Ex.:
# $ ./testargs.sh um dois tres quatro

if [[ -z "$1" ]]; then
  echo "Uso: $0 argumento1 argumento2 etc"
  exit 1
fi

echo "---> Listando argumentos com \"\$*\":"
num=1
for arg in "$*"; do
  echo "argumento #$num = $arg"
  ((num++))
done
# Conclusão: $* mostra todos os argumentos como uma única string

echo "---> Listando argumentos com \"\$@\":"
num=1
for arg in "$@"; do
  echo "argumento #$num = $arg"
  ((num++))
done
# Conclusão: $@ mostra cada argumento em strings separadas
```

Executando o script acima:

```shell-session
$ ./testargs.sh um dois tres quatro

---> Listando argumentos com "$*":
argumento #1 = um dois tres quatro

---> Listando argumentos com "$@":
argumento #1 = um
argumento #2 = dois
argumento #3 = tres
argumento #4 = quatro
```

Acho que com o exemplo acima ficou claro, certo?

Agora para entendermos no detalhe a real diferença entre o `$*` e o `$@` é necessário conhecermos a variável `$IFS`.

As letras IFS são uma sigla para _Internal Field Separator_, esta variável é responsável por "quebrar" o conteúdo de uma linha de comando em argumentos separados. Geralmente o conteúdo da `$IFS` é espaço, tabulação e nova linha. Portanto quando o shell encontra esses caracteres ele os ignora e encara o que vem a seguir como um novo parâmetro.

(Se você não tem a menor ideia do que estou falando, não precisa se desesperar. Fica aqui meu compromisso de posteriormente escrever um artigo detalhando o `$IFS` de uma maneira que você nunca mais vai esquecer.)

Voltando ao nosso tema: diferença entre o `$*` e o `$@`.

Quando você usa a notação com asterisco `"$*"`, o bash vai expandir isso para uma única string com o valor de cada parâmetro separado pelo primeiro caractere da variável `IFS`. Ou seja, se o primeiro caractere da variável `IFS` é `c`, então um `"$*"` vai expandir para `"$1c$2c$3c..."` (uma única string com o valor de cada parâmetro separado por `c`).

Como normalmente o primeiro caractere da variável `$IFS` é espaço, na maioria das vezes o `"$*"` é expandido para `"$1 $2 $3 ..."`.

Agora no caso da notação com arroba `"$@"`, o bash vai expandir isso para cada parâmetro entre aspas separados por um espaço. Ou seja, um `"$@"` vai expandir para `"$1" "$2" "$3" ...` (cada parâmetro entre aspas, separados por um espaço).

**Resumo**

- `@` arroba: expande separando com um espaço.
- `*` asterisco: expande separando com o primeiro caractere do `$IFS`.

A propósito: se tem curiosidade de ver como essa "sintonia" do `$*` com o `$IFS` pode ser bem aproveitada, dá uma olhada no artigo [Como juntar elementos de um array separando-os com um caracter qualquer](http://meleu.sh/join-bash/).


### O comando `shift`

O bash possui um comando embutido para lidar com parâmetros: o `shift`.

Para entender o `shift` é legal usarmos uma analogia de fila de supermercado.
Imagine uma fila com 10 pessoas, quando o primeiro é atendido, o segundo passa a ser o primeiro da fila, o terceiro será o segundo e assim por diante. E a fila agora tem 9 pessoas.

O `shift` é um comando que faz "a fila andar".

Quando você usa o `shift` o primeiro parâmetro da lista some e o segundo vai para `$1`, o terceiro vai para `$2`, e assim por diante.

Você pode também especificar quantas "pessoas da fila você vai atender" através do comando `shift N` onde `N` é o número de "pessoas".

Exemplo: se você usar `shift 2`, tanto o primeiro como o segundo parâmetro somem e o `$3` vira `$1`, o `$4` vira `$2` e assim por diante.

**OBSERVAÇÃO**: se `n` for maior que o número de parâmetros o `shift` **não** é executado.

Veja este exemplo:

```
#!/bin/bash
# shift-example.sh

echo "$#: $*"
echo -e "executando \"shift\""
shift
echo "$#: $*"
echo -e "executando \"shift 5\""
shift 5
echo "$#: $*"
echo -e "executando \"shift 7\""
shift 7
echo "$#: $*"
```
Em execução:
```shell-session
$ ./shift-exemplo.sh 1 2 3 4 5 6 7 8 9 0
10: 1 2 3 4 5 6 7 8 9 0
executando "shift"
9: 2 3 4 5 6 7 8 9 0
executando "shift 5"
4: 7 8 9 0
executando "shift 7"
4: 7 8 9 0
```

**Lembre-se**: Os valores que saem são perdidos. Se for precisar deles, salve-os em alguma variável.

Uma maneira muito útil de usar o `shift` é para ir pegando opções da linha de comando (`--help`, `--verbose`, etc.). Pretendo escrever um post sobre isso em breve, mas se você quiser pode ver diretamente na fonte de onde eu aprendi: Capítulo 4 do livro [Shell Script Profissional](https://novatec.com.br/livros/shellscript/) do Aurélio.


### O comando `set` para editar parâmetros

O que vou passar neste tópico não é sobre como usar "todo o poder do
comando set", e sim como usar set especificamente para editar parâmetros.
Não tem nenhum segredo! Veja este exemplo:

```
set um dois tres
```

Isso fará com que `$1` seja `um`, `$2` seja `dois`, `$3` seja `tres` e só!
Os valores anteriores, se existiam, serão sobrescritos, e também não existirá `$4`, `$5`, etc. mesmo que eles tenham existido antes do `set`.

Veja um exemplo de script:

```
#!/usr/bin/env bash
# setparam.sh

echo "--> Os $# parâmetros passados inicialmente foram:"
echo "$@"
echo "--> Mas agora eu vou alterá-los para 'um', 'dois' e 'tres'."
set um dois tres
echo "--> Os $# novos parâmetros agora são:
echo "$@"
```

Não interessa quantos parâmetros você passar para este script, no
final você só terá `$1`, `$2` e `$3` valendo `um`, `dois` e `tres`,
respectivamente.

```shell-session
$ ./setparam.sh eu adoro programação shell
--> Os 4 parâmetros passados inicialmente foram:
eu adoro programação shell
--> Mas agora eu vou alterá-los para 'um', 'dois' e 'tres'.
--> Os 3 novos parâmetros agora são:
um dois tres
```

Um exemplo de uso engenhoso do `set` eu aprendi no "Phrack Extraction Utility" e reproduzi num script similar que escrevi e que pode ser visto [nesse link](https://github.com/meleu/bashscripting/blob/master/src/Mextract.sh).


## Fontes

- `man bash` na seção de "Special Parameters"
- `help shift`
- `help set`
- https://mywiki.wooledge.org/BashGuide/Parameters
- livro: Shell Script Profissional, do Aurélio Jargas.
