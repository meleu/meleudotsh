---
title: "Deixe o bash mais rigoroso com seu script e evite dores de cabeça"
description: >
  Habilite essas opções para deixar o bash mais exigente e interromper o seu script assim que algum problema for encontrado.
tags:
  - boas-praticas
date: 2022-04-07T11:08:14-03:00
cover:
  image: "img/strict-mode.png"
  alt: habilitando bash strict-mode
---

> Este artigo é parte da série "**Práticas de programação shell que mudarão sua vida**"
> 
> Os artigos da série são:
> 
> 1. [deixe seu bash mais rigoroso](/bash-rigoroso)
> 2. [use um `trap` para saber exatamente onde seu script quebrou](https://meleu.sh/trap-err)
> 3. [use o shellcheck](https://meleu.sh/shellcheck)

No livro [The Art of Unix Programming](http://www.catb.org/esr/writings/taoup/html/), o Eric S. Raymond diz o seguinte:

> Quando precisar falhar, falhe ruidosamente e o mais cedo possível.

Neste artigo eu vou tratar da situação de "falhar o mais cedo possível". Em um outro artigo eu vou falar sobre o "falhar ruidosamente".

🤔 - "Não entendi... E Qual é a vantagem de falhar o mais cedo possível?"

Excelente pergunta! A vantagem é que quanto mais cedo você visualizar um erro, mais rápido você irá corrigí-lo.

Imagine um erro que só aparece depois de 2 meses que você mexeu no código... Você terá que gastar um tempo até se contextualizar e ter clareza do que aquele código faz.

## Motivação

Um dos problemas que todo programador shell passa ou já passou é perceber que seu script "bugou" em alguma parte mas a execução do script continuou.

Exemplo: você esperava que a saída de um `grep` fosse preencher o conteúdo de uma certa variável mas a saída do `grep` veio vazia. O seu script continua executando (com sua variável vazia) e em algum outro ponto que você vai usar essa variável você percebe que ela está vazia.

Se você programa em shell há algum tempo certamente já passou por situações como essa, certo?

O que vamos ver nesse artigo vai fazer você nunca mais sofrer com isso.

## Não quero ler isso tudo! Me diz logo o que tenho que fazer!

"Sempre" use isso no topo dos seus scripts:
```bash
set -euo pipefail
```

**Observação**: a opção `u` tem algumas nuances que você precisa estar ciente. Se realmente não quiser ler e nem arriscar, certifique-se de ao menos usar `set -eo pipefail`.


## Opções do comando `set`

O comando `set` permite que configuremos algumas opções do bash. Neste artigo vamos focar em 3 opções que você deve considerar sempre ativar em seus scripts.

**Observação**: O `set` faz mais do que ligar/desligar opções do bash. Para uma descrição mais detalhada digite `help set` no seu prompt (ou veja a manpage do bash).

### Saia imediatamente ao falhar

```bash
# versão longa
set -o errexit

# versão curta
set -e
```


Veja o seguinte script:

```bash
#!/usr/bin/env bash
# set+e.sh

echo "O comando a seguir vai falhar..."
cat /diretorio/arquivo-inexistente

echo "O comando falhou mas você está lendo essa mensagem... :("
```

Vamos executá-lo:
```txt
$ bash set+e.sh 
O comando a seguir vai falhar...
cat: /diretorio/arquivo-inexistente: No such file or directory
O comando falhou mas você está lendo essa mensagem... :(
```

Como percebemos, o `cat` falhou e o script continuou sua execução. Como não queremos que isso aconteça, vamos usar o `set -e`:

```bash
#!/usr/bin/env bash
# set-e.sh

set -e # exit on fail - saia ao falhar

echo "O comando a seguir vai falhar..."

cat /diretorio/arquivo-inexistente

echo "O comando falhou e essa mensagem nem será impressa. :)"
```

Executando:
```txt
$ bash set-e.sh 
O comando a seguir vai falhar...
cat: /diretorio/arquivo-inexistente: No such file or directory
```


Uma coisa que vale a pena ressaltar, é que em cláusulas condicionais (como `if`, `while` e `||`) o erro no comando **não** causa a interrupção do programa.

Exemplo:
```bash
#!/usr/bin/env bash
# set-e.v2.sh

set -e # 

arquivo='/dir/arquivo-inexistente'

echo "--- inicio do arquivo ---"

if cat "${arquivo}"; then
  echo "--- fim do arquivo ---"
else
  echo "--> Aqui deveria ter os comandos"
  echo "--> a serem executados caso o comando"
  echo "--> 'cat' falhe..."
fi

echo "A falha do 'cat' foi \"capturada\" pelo 'if'"
echo "O programa continua..."
```

Executando:
```txt
$ bash set-e.v2.sh 
--- inicio do arquivo ---
cat: /dir/arquivo-inexistente: No such file or directory
--> Aqui deveria ter os comandos
--> a serem executados caso o comando
--> 'cat' falhe...
A falha do 'cat' foi "capturada" pelo 'if'
O programa continua...
```


### Status code do primeiro comando que falhar numa pipeline

```bash
# somente versão longa
set -o pipefail
```

Quando você faz um encadeamento de comando usando o `|`, **todos** os comandos desse encadeamento são executados e o valor de retorno será o valor do último comando.

No exemplo a seguir, imagine que queremos procurar se um determinado usuário está no arquivo `/etc/passwd` e queremos converter o nome todo para maiúsculo:

```bash
#!/usr/bin/env bash
# pipe-test.sh

grep UsuarioInvalido /etc/passwd \
  | cut -d: -f1 \
  | tr [:lower:] [:upper:]
```

Executando e verificando se o script terminou com sucesso ou falha:
```txt
$ bash pipe-test.sh && echo sucesso || echo falha
sucesso
```

O QUE?! Como assim "sucesso"?! 😱

Pois é... Isso acontece porque o bash está retornando o status code do último comando da pipeline. Aquele `tr` não fez nada, mas terminou com sucesso.

Agora, ao habilitarmos o `pipefail`, o status code da pipeline será o status do primeiro comando que falhar. Vamos fazer um teste:
```bash
#!/usr/bin/env bash
# pipe-test2.sh

set -o pipefail

grep UsuarioInvalido /etc/passwd \
  | cut -d: -f1 \
  | tr [:lower:] [:upper:]
```

Executando e verificando se o status:
```
$ bash pipe-test.sh && echo sucesso || echo falha
falha
```

Agora sim! Isso faz mais sentido! 👍


### Não permita variáveis não declaradas

```bash
# versão longa
set -o nounset

# versão curta
set -u
```

Essa opção faz o bash falhar quando ele encontra uma expansão de variável que não foi declarada.

Vamos testar com o famigerado "Hello World" com um nome:
```bash
#!/usr/bin/env bash
# hello.sh

set -u

echo "Hello, ${name}"
echo "Seja bem vindo..."
```

Executando:
```txt
$ bash -u hello.sh 
hello.sh: line 6: name: unbound variable
```

Muito frequentemente a opção `set -u` nos ajuda a evitar problemas.

Lembra todo aquele tempo que você gastou tentando debugar seu script pra só depois de muita irritação você perceber que o problema foi apenas um erro de digitação no nome da variável? Quem nunca?

#### Ponto de atenção no `set -u`

O `set -u` tem o seguinte efeito colateral: ele fará o seu script falhar ao tentar expandir TODA E QUALQUER variável que não teve o seu valor explicitamente declarado.

Vou mostrar um exemplo de quando isso pode ser incoveniente:
```bash
#!/usr/bin/env bash
# hello2.sh

set -u

if [[ -z "${name}" ]]; then
  echo "Hello World"
else
  echo "Hello, ${name}"
fi

echo "Seja bem vindo..."
```

Executando:
```txt
$ bash hello2.sh 
hello2.sh: line 6: name: unbound variable
```

Mesmo que você pense "ah, eu já estou tomando cuidado de verificar se a variável está vazia", ainda assim o bash vai parar o script ao encontrar aquela variável sem valor.

Pra resolver esse problema você precisa explicitamente dar algum valor para a variável `name`, podendo inclusive ser um valor vazio:
```bash
#!/usr/bin/env bash
# hello3.sh

set -u

# string vazia
name=

if [[ -z "${name}" ]]; then
  echo "Hello World"
else
  echo "Hello, ${name}"
fi

echo "Seja bem vindo..."
```

Executando:
```txt
$ bash hello3.sh 
Hello World
Seja bem vindo...
```

OK, esse exemplo "Hello World" foi bem simplório... Vamos falar de algo mais concreto.

Na "vida real" é bastante comum usarmos o `[[ -z "${VARIAVEL}" ]]` para verificar se uma variável de ambiente está vazia ou não. Exemplo hipotético:

```bash
if [[ -z "${ENV_VARIABLE}" ]]; then
  echo "'ENV_VARIABLE' está vazia"
  echo "vamos fazer algo quanto a isso..."
fi
```

Esse exemplo é só pra mostrar que queremos que nosso script faça algo quando aquela variável estiver vazia. No entanto, com o `set -u` aquele `if` pode acabar quebrando antes mesmo da verificação de variável vazia.

Dando um exemplo mais "vida real" ainda (só vai fazer sentido se você entende de `git`):

No meu trabalho usamos GitLab CI para esteira de Integração Contínua. O GitLab automaticamente define algumas variáveis para você usar na sua esteira. Uma dessas variáveis é o `CI_COMMIT_BRANCH`.

Eu já passei por um cenário onde essa variável não veio preenchida (quando a pipeline é disparada por um Merge Request) e isso acabou quebrando meu script.

Para contornar essa situação, tive que usar uma das técnicas de expansão de parâmetros, da seguinte forma:
```bash
if [[ -z "${CI_COMMIT_BRANCH:-}" ]]; then
  # ...
fi
```

Expansão de parâmetros está fora do escopo desse artigo, mas pra resumir:

```
# se $parametro for vazio, a expansão abaixo
# vai gerar a string "valor default"
${parametro:-valor default}
```

Como você pode ver, o que vem depois `:-` será o valor *default*. E no exemplo que dei, `${CI_COMMIT_BRANCH:-}`, o valor *default* é uma string vazia.


## Conclusão

Neste artigo vimos como uma simples linha de código no início dos seus scripts vai te salvar de muitas dores de cabeça e evitar que você desperdice tempo caçando bugs perfeitamente evitáveis.

```bash
set -euo pipefail
```


## Fontes

- `help set`
- `man bash`
- The Art of Unix Programming - [Rule of Repair](http://www.catb.org/esr/writings/taoup/html/ch01s06.html#id2878538) 