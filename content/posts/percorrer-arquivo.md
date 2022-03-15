---
title: "Como ler o conteúdo de um arquivo linha por linha em shell script"
description: >
  Veja uma maneira robusta de percorrer um arquivo linha por linha. E sem cair nas armadilhas sutis que nos trazem surpresas desagradáveis.
tags:
  - boas praticas
  - codigo
date: 2020-01-21T16:00:42-03:00
cover:
  image: "percorrer-arquivo.png"
  alt: "loop while para percorrer o arquivo linha a linha"
---

Uma das coisas que você certamente vai encarar um dia como um programador de shell scripts é a necessidade de percorrer um arquivo inteiro lendo cada linha e fazer algo com este conteúdo. Veremos neste artigo como fazer isso de maneira segura, robusta e evitando as possíveis armadilhas que podem aparecer no caminho.

Pra adiantar seu lado, vou logo de cara lhe dar a solução que eu considero mais robusta. Em seguida explica cada detalhe dessa estrutura.

```
while IFS= read -r linha || [[ -n "$linha" ]]; do
  echo "$linha"
  # faça algo mais interessante aqui...
done < "$arquivo"
```

Dei esse grande _spoiler_ pois percebi que esse artigo ficou relativamente grande, e uma das coisas que mais valorizo aqui é o seu tempo.

Se você só quer a solução, ali está ela. Mas se quiser entender cada detalhezinho dessa estrutura, continue lendo.

A minha explicação vai partir da solução [_naïve_](https://en.wikipedia.org/wiki/Naivety) (ou seja, a solução mais básica e ingênua) de como percorrer cada linha de um arquivo. E a partir de cada problema que encontrarmos vamos adicionar uma maneira de contornar. No final de tudo teremos aquela estrutura que mostrei acima.


## Solução 1: `read linha`

Para fins didáticos, imaginemos que queremos um script que simplesmente imprima na tela cada linha de um arquivo de maneira similar ao uso mais básico do comando `cat`.

Vamos chamar esse script de `mycat.sh`, e usaremos redirecionamento para um loop `while` para alcançar o nosso objetivo.

A primeira vez que eu ouvi falar de redirecionamento de conteúdo de um arquivo para um loop (há duas décadas atrás), a primeira coisa que tentei foi o seguinte:

```
#!/usr/bin/env bash
# mycat.sh
#
# versão 1: solução naïve

arquivo="$1"

while read linha; do
  echo "$linha"
done < "$arquivo"
```

Nós vamos testando se o nosso `mycat.sh` está legal através da comparação com a saída do comando `cat` real.

```shell-session
$ # primeiro com cat
$ cat echo-e.txt 
Uma coisa interessante sobre o comando 'echo' é que quando
usamos a opção -e podemos usar algumas sequências de caracteres
com um significado especial.

Por exemplo \n significa uma nova linha, e \t significa <TAB>.
$ 
$ # depois com mycat.sh
$ ./mycat.sh echo-e.txt 
Uma coisa interessante sobre o comando 'echo' é que quando
usamos a opção -e podemos usar algumas sequências de caracteres
com um significado especial.

Por exemplo n significa uma nova linha, e t significa <TAB>.
$ 
```

Aparentemente tudo bem. Mas ali na última linha ao invés de exibir `\n` e `\t`, o script mostrou apenas `n` e `t`. Ou seja, ele "engoliu" a contra-barra.

Mas nós sabemos que o `read` é cheio de opções. Deve ter uma maneira de resolver isso.

Dando uma olhada no `help read` encontramos a opção `-r`, que diz "_do not allow backslashes to escape any characters_". Ou seja, não permitir o contra-barra de "escapar" caracter algum.

OK, hora de melhorar o nosso script...


## Solução 2: `read -r linha`

Direto ao código:

```
#!/usr/bin/env bash
# mycat.sh
#
# versão 1: solução naïve
# versão 2: lidando com \contrabarras\

arquivo="$1"

while read -r linha; do
  echo "$linha"
done < "$arquivo"
```

Vamos como o `mycat.sh` vai lidar com o `echo-e.txt` agora:

```shell-session
$ ./mycat.sh echo-e.txt 
Uma coisa interessante sobre o comando 'echo' é que quando
usamos a opção -e podemos usar algumas sequências de caracteres
com um significado especial.

Por exemplo \n significa uma nova linha, e \t significa <TAB>.
$ 
```

Opa! Legal! Vamos testar agora com um arquivo `simples.txt` que possui um pouquinho mais de conteúdo:

```shell-session
$ # primeiro com o cat
$ cat simples.txt 
Este é um arquivo texto muito simples
=====================================

  Este é o primeiro parágrafo.

  Aqui no segundo parágrafo vamos colocar
alguns caracteres com \contrabarra\, como por
exemplo o \n e o \t, só para testes...

  Acho que está tudo bem. Vamos terminando
aqui no terceiro parágrafo. Até próxima!
$ 
$ # agora com o mycat.sh
$ ./mycat.sh simples.txt 
Este é um arquivo texto muito simples
=====================================

Este é o primeiro parágrafo.

Aqui no segundo parágrafo vamos colocar
alguns caracteres com \contrabarra\, como por
exemplo o \n e o \t, só para testes...

Acho que está tudo bem. Vamos terminando
aqui no terceiro parágrafo. Até próxima!
$ 
```

As contra-barras, OK. Mas agora tem uma outra inconsistência ali: os espaços no início de cada parágrafo foram omitidos. :(

Isso está acontecendo porque usamos o `read` para atribuir um valor a uma variável, ele automaticamente ignora os espaços que ficam tanto no começo como no final do conteúdo que você informar como entrada.

Na verdade isso acontece devido a variável de ambiente `$IFS`. As letras IFS são um acrônimo para _Internal Field Separator_, esta variável é responsável por "quebrar" o conteúdo de uma linha de comando em argumentos separados.

Geralmente o conteúdo do `IFS` é espaço, tabulação e nova linha. Portanto quando o shell encontra esses caracteres ele os ignora e encara o que vem a seguir como um novo parâmetro.

(Se você não tem a menor ideia do que estou falando, não precisa se desesperar. Fica aqui meu compromisso de posteriormente escrever um artigo detalhando o `$IFS` de uma maneira que você nunca mais vai esquecer.)

Para contornar esse problema do script ficar "engolindo" os espaços no começo das linhas, vamos usar um `$IFS` vazio. Assim o `read` vai considerar que tudo que está vindo como entrada é apenas um único argumento.


## Solução 3: `IFS= read -r linha`

```
#!/usr/bin/env bash
# mycat.sh
#
# versão 1: solução naïve
# versão 2: lidando com \contrabarras\
# versão 3: lidando com espaços no início/fim

arquivo="$1"

while IFS= read -r linha; do
  echo "$linha"
done < "$arquivo"
```

Essa sintaxe que estamos usando `IFS= read -r linha` é uma maneira de dizer ao shell que queremos alterar a variável de ambiente `$IFS` apenas para a execução do comando `read` que vem a seguir.

Essa "técnica" é mencionada lá na manpage do bash, na seção que fala sobre ENVIRONMENT, num parágrafo que diz o seguinte: (tradução livre):

> O ambiente para qualquer comando simples ou função pode ser temporariamente alterado prefixando o comando/função com atribuições de parâmetro (...). Estas atribuições afetam somente o ambiente visto por aquele comando.

Portanto, `IFS= read -r linha` vai alterar o `IFS` somente durante a execução do `read`, depois disso ele volta ao normal.

Vamos testar:

```shell-session
$ # primeiro com cat
$ cat simples.txt 
Este é um arquivo texto muito simples
=====================================

  Este é o primeiro parágrafo.

  Aqui no segundo parágrafo vamos colocar
alguns caracteres com \contrabarra\, como por
exemplo o \n e o \t, só para testes...

  Acho que está tudo bem. Vamos terminando
aqui no terceiro parágrafo. Até próxima!
$ 
$ # agora com mycat.sh
$ ./mycat.sh simples.txt 
Este é um arquivo texto muito simples
=====================================

  Este é o primeiro parágrafo.

  Aqui no segundo parágrafo vamos colocar
alguns caracteres com \contrabarra\, como por
exemplo o \n e o \t, só para testes...

  Acho que está tudo bem. Vamos terminando
aqui no terceiro parágrafo. Até próxima!
$ 
```

Agora sim! Tudo igualzinho ao cat!

Aparentemente já temos uma solução bastante robusta, não é mesmo?

Porém na nossa vida acontecem coisas inesperadas... Ao longo da sua vida de programador shell-script você vai se deparar com arquivos que não foram gerados num ambiente UNIX-like. Podendo inclusive se deparar com um arquivo texto que não terminha com um caractere nova linha.

Para simular tal situação vamos utilizar o `head` com a opção `-c` (note que nos exemplos a seguir algumas vezes o prompt aparece em lugares diferentes):

```shell-session
$ # quero apenas 105 bytes de simples.txt
$ head -c 105 simples.txt 
Este é um arquivo texto muito simples
=====================================

  Este é o primeiro parág$
$ # obs: prompt aqui ----^
$ 
$ # vamos jogar isso num arquivo
$ head -c 105 simples.txt > atipico.txt
$ 
$ # testando com cat
$ cat atipico.txt 
Este é um arquivo texto muito simples
=====================================

  Este é o primeiro parág$
$ # obs: prompt aqui ----^
$ 
$ 
$ # agora vamos testar com mycat.sh
$ ./mycat.sh atipico.txt 
Este é um arquivo texto muito simples
=====================================

$ 
```

Eita! No teste com `mycat.sh` a última linha foi omitida!

É que no mundo UNIX o caractere nova linha é realmente levado muito a sério...

O problema do `mycat.sh` acontece porque quando o `read` não encontra o caracter de nova linha ele "retorna falso" (ou seja, ao final da execução a variável `$?` é diferente de zero).

No entanto uma coisa curiosa acontece: mesmo retornando falso, **o valor da variável é atualizado**.

Observe essa demonstração:

```shell-session
$ # testando com var1
$ read var1
digitei isso e teclei <ENTER>
$ echo "$?"
0
$ # esse zero significa que o read terminou com sucesso
$ echo "$var1"
digitei isso e teclei <ENTER>
$ 
$ # vamos a outro teste, com var2
$ read var2
mandando um end-of-file com <CTRL+D>$ echo "$?"
1
$ # esse um significa que o read terminou com alguma falha
$ echo "$var2"
mandando um end-of-file com <CTRL+D>
$ # no entanto o conteúdo de var2 foi atualizado!
```

Desta forma ficou claro que quando o `read` recebe alguns caracteres e encontra um _end-of-file_ antes de encontrar um nova linha, acontecem duas coisas:

1. O conteúdo da variável é atualizado.
2. O `read` retorna com falha.

Vamos nos aproveitar destes dois fatos e tornar o nosso `mycat.sh` ainda mais robusto!


## Solução 4: `IFS= read -r linha || [[ -n "$line" ]]`

```
#!/usr/bin/env bash
# mycat.sh
#
# versão 1: solução naïve
# versão 2: lidando com \contrabarras\
# versão 3: lidando com espaços no início/fim
# versão 4: lidando com término inesperado do input

arquivo="$1"

while IFS= read -r linha || [[ -n "$linha" ]]; do
  echo "$linha"
done < "$arquivo"
```

A sintaxe `||` pode ser "traduzida" como "se, e somente se, o comando a esquerda do `||` falhar, execute o comando a direita". Portanto, caso o `read` falhe o script testa se a variável `$linha` não está vazia. Isso é aproveitar aqueles fatos que mencionei acima (1. `read` falha; 2. variável é atualizada).

Agora, se o `read` falhar e também o teste da variável `$linha` vazia falhar, aí chegamos ao fim do `while`.

Vamos direto ao teste:

```shell-session
$ # primeiro com cat
$ cat atipico.txt 
Este é um arquivo texto muito simples
=====================================

  Este é o primeiro parág$ 
$ # prompt voltou aqui --^
$ 
$ # agora testando com mycat.sh
$ ./mycat.sh atipico.txt 
Este é um arquivo texto muito simples
=====================================

  Este é o primeiro parág
$ # <-- o prompt voltou aqui
$ 
```

OK... Não ficou 100% igual, pois com o `cat` o prompt retorna logo após o final efetivo do arquivo.

O `mycat.sh` imprime um nova linha após o final do arquivo. No entanto isso não é um caracter nova linha que milagrosamente apareceu ao final de `$linha`, mas sim um nova linha impresso pelo `echo`. Portanto se você está usando essa estrutura para percorrer o arquivo linha por linha para fazer um processamento mais sério do que simplesmente imprimir a linha, pode ficar tranquilo que os dados estarão íntegros.

Ufa! Acabamos!



## Fontes

- https://mywiki.wooledge.org/BashFAQ/001
- `help read`
- https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#Environment
