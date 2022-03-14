---
title: "here string: evitando o uso desnecessário do echo"
description: >
  É comum vermos códigos usando 'echo $var | comando'. Nesse artigo veremos como a técnica here strings evita este uso desnecessário do echo, melhorando a legibilidade e performance do seu código (também veremos raras situações onde essa técnica não nos atende).
tags:
  - fundamentos
  - boas praticas
date: 2020-01-17T15:58:54-03:00
---

## O que é um _here string_?

Um _here string_ nada mais é do que uma maneira de transformar uma string na entrada padrão (stdin) de um programa. Normalmente utilizamos essa técnica pegando a string contida em uma variável.

A sintaxe, pegando de uma variável, é a seguinte:

```bash
COMMAND <<< "$VAR"
```

Onde o conteúdo variável `$VAR` será colocado na entrada padrão do `COMMAND`.

Usar a técnica do _here string_ evita o uso de estruturas como:

```bash
# uso desnecessário do echo
echo "$var" | grep padrao
```

Podemos obter o mesmo resultado usando a sintaxe do _here string_:

```bash
grep padrao <<< "$var"
```

Na minha opinião, o código fica mais limpo e legível desta forma. Além disso, também temos um ganho de performance.

A explicação para isso está num trecho do [manual do bash que fala de _Pipelines_](https://www.gnu.org/software/bash/manual/html_node/Pipelines.html#Pipelines) (tradução livre):

> Cada comando em um _pipeline_ é executado em seu próprio subshell, o que é um processo separado (...)

Ou seja, a cada pipe um subshell é inicializado, e isso gera um custo de performance. Principalmente se você estiver fazendo isso dentro de um loop com dezenas/centenas de iterações.

Espero que estes dois fatores (1. código limpo, 2. performance) sejam motivos suficientes para você começar a usar o _here string_ a partir de agora.


## Quando não usar _here strings_

Conforme podemos ver [no próprio manual do bash](https://www.gnu.org/software/bash/manual/html_node/Redirections.html#Here-Strings) (tradução livre):

> O resultado é enviado à entrada padrão do comando, como uma única string e **com uma nova linha adicionada ao final**

Portanto, em casos onde esse caractere de nova linha não é desejado, não podemos usar o _here string_. Estes casos são bem raros, mas acontecem.

Eu me deparei com esse problema quando estava trabalhando numa aplicação onde eu precisava calcular o _checksum_ MD5 dos 8 primeiros bytes de um arquivo. Vou ilustrar o problema através do exemplo a seguir:

```
$ cat texto.txt
conteudo do arquivo

$ head -c8 texto.txt | md5sum
b59853db2f3ef8f156a72e38c30ba7d2  -
$ 
$ # armazenando o conteúdo em uma variavel
$ variavel=$(head -c8 texto.txt)
$ md5sum <<< "$variavel"
746378123fbe2cbca33ed5d88f35c5bb  -
$ # ué! era pra ser igual ao md5 lá de cima!
$ 
$ # vejamos com o 'echo -n' (sem nova linha no final):
$ echo -n "$variavel" | md5sum
b59853db2f3ef8f156a72e38c30ba7d2  -
$ # agora sim! o checksum é igual! :)
```

Conforme podemos ver, nas raríssimas situações onde o caracter de nova linha no final da string pode alterar o resultado desejado, o uso de _here strings_ não resolve.


## Cuidado com o exagero

Legal, você aprendeu que usando _here strings_ você tem um ganho de performance, mas também não precisa ficar fissurado com isso e usar _here strings_ em toda e qualquer oportunidade.

Entenda que a função dessa técnica é **colocar uma string no arquivo de entrada padrão de um comando**.

Eu já vi um caso onde a pessoa usou `comando <<< $(< arquivo)`. Ou seja, através do redirecionamento no `$()` ele colocou o conteúdo de um arquivo numa string para logo em seguida mandar essa string para a entrada padrão do `comando`. Neste caso bastava usar simplesmente `comando < arquivo`.


## Fontes

- https://www.tldp.org/LDP/abs/html/x17837.html
- https://www.gnu.org/software/bash/manual/html_node/Redirections.html#Here-Strings
- https://www.gnu.org/software/bash/manual/html_node/Pipelines.html#Pipelines