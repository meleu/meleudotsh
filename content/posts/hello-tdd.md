---
title: Aplicando TDD no Bash
description: >
  Aprenda Desenvolvimento Guiado por Testes no Bash.
tags:
  - testes
date: 2025-06-13T19:00:00-03:00
cover:
  image: "img/hello-bats-tdd.png"
  alt: bats output
---



O objetivo principal desse artigo é ensinar o básico de como praticar Test-Driven Development em um projeto bash utilizando o BATS como framework de testes.

Para isso escreveremos um "Hello, World poliglota".

> ## AVISO!
>
> Se você já leu o artigo [Aprenda TDD no Bash](../tdd-bash) não encontrará novidade alguma aqui!
>
> Escrevi esse artigo aqui pois percebi que o artigo original estava muito longo. Portanto resolvi quebrá-lo em dois:
>
> 1. [Setup do BATS](../bats)
> 2. Fluxo de _Test-Driven Development_


Durante a leitura pode ser que você ache que estou progredindo de forma demasiadamente lenta para resolver um problema tão simples, e isso é verdade! É intencional. Pois quero mostrar o passo a passo do TDD (e não como escrever hello-world).

Claro que estas ferramentas são bem mais úteis para resolver "problemas reais". Mas como o foco aqui é ensinar as técnicas de TDD e utilização do BATS, eu não quero adicionar mais complexidade usando um "problema real".

Ah! E tenho certeza que no caminho você vai acabar aprendendo alguns macetinhos de bash. 😉


## O que é BATS?

[BATS](https://github.com/bats-core/bats-core) significa Bash Automated Testing System. É um _framework_ de testes para bash, que permite que verifiquemos se o nosso programa está se comportando da maneira que queremos.

No artigo anterior vimos o setup básico de um projeto bash usando o BATS. Neste artigo estou assumindo que você já leu o artigo anterior e já tem tudo que precisa pra começarmos a aprender TDD com bash.

## O que é TDD

TDD significa Test-Driven Development, ou seja, desenvolvimento guiado por testes.

Trata-se de uma metodologia onde primeiro escrevemos o teste e deixamos que esse teste guie o desenvolvimento.

O processo do TDD segue um ciclo iterativo geralmente conhecido como "Red-Green-Refactor". 

1. Escrevemos um teste definindo uma nova funcionalidade. Nesse primeiro momento a funcionalidade nem existe, portanto o teste falha (fase "Red")
2. Em seguida, escrevemos o mínimo de código necessário pra passar no teste (fase "Green").
3. Por fim, o código é refatorado para melhorar sua estrutura e legibilidade. Fazemos isso com a segurança dos testes garantindo que não estamos quebrando nada (fase "Refactor")

Eu acho o TDD extremamente valioso pelos seguintes motivos:

- encoraja um design de código mais modular e menos acoplado.
- ajuda a identificar e corrigir bugs mais cedo.
- **se bem aplicado**, reduz bastante o custo e tempo de manutenção.
- os testes servem como uma documentação viva do sistema, já que descrevem o comportamento esperado do programa.

Só coisa boa, né? Mas, obviamente, todas essas "maravilhas" possuem um preço: aprender TDD é trabalhoso e requer bastante prática.

Esse artigo é exatamente uma maneira de estimular o início dessa jornada de praticante de TDD 

Se você se importa com a qualidade do seu trabalho, saiba que esse esforço vale muito a pena!

Agora chega de blablabla! Se você ainda está aqui lendo é por que ao menos têm esperança de que isso é uma coisa útil de se aprender. Então vamos pra prática!

## Iniciando o projeto

Vamos seguir aquele checklist mencionado no artigo anterior:

### Checklilst

- [ ] Crie um diretório para o projeto
- [ ] Crie os diretórios `test` e `src`
- [ ] Inicie um repo git
- [ ] Instale BATS
- [ ] Instale os BATS helpers

### Passo a passo

```bash
# diretório do projeto
mkdir hello-tdd
cd hello-tdd

# diretório dos testes e do código de produção
mkdir test src

# iniciando um repositório
git init

# bats-core: o bats propriamente dito (útil para usarmos em pipelines)
git submodule add \
  https://github.com/bats-core/bats-core.git \
  test/bats

# bats-assert: responsável pelas asserções
git submodule add \
  https://github.com/bats-core/bats-assert.git \
  test/test_helper/bats-assert

# bats-support: responsável por mensagems de falha mais amigáveis
git submodule add \
  https://github.com/bats-core/bats-support.git \
  test/test_helper/bats-support
```

Crie um arquivo `test/hello_test.bats` com esse conteúdo:

```bash
setup() {
  load 'test_helper/bats-support/load'
  load 'test_helper/bats-assert/load'

  PATH="${BATS_TEST_DIRNAME}/../src:${PATH}"
}

# Agora você consegue facilmente chamar seu código
@test "TODO: CHANGE THIS TEST..." {
  run my_code.sh
  # veja como fazer asserções no README de
  # https://github.com/bats-core/bats-assert
}
```

### Estrutura inicial do projeto

Agora você deverá ter uma estrutura tipo assim:

```
$ tree -F -L 2
./
├── src/
│   └── hello.sh*
└── test/
    ├── bats/        # 👈 arquivos do bats-core aqui
    ├── hello_test.bats
    └── test_helper/ # 👈 bats-support e bats-assert aqui
```



## Começando com TDD

Você provavelmente já sabe como criar um "Hello, World" em bash. Peço que resista à tentação de escrever o código e siga comigo pra usarmos TDD, onde escreveremos os testes primeiro. Antes mesmo do nosso código principal existir.

Já temos uma estrutura inicial do nosso arquivo de testes em `test/hello_test.bats`, agora vamos adicionar o nosso primeiro teste e fazer uma **asserção** sobre sua saída. Nosso teste ficará assim:

```bash
# ... conteúdo original do test/hello_test.bats

@test "say Hello, World" {
  run hello.sh # 👈 note que estamos usando um `run` aqui!
  assert_output "Hello, World"
}
```

Nesse código estamos usando o `run` para chamar o nosso programa. Isso nos traz várias conveniências, como por exemplo automaticamente salvar a saída gerada pelo programa para que possamos verificar com o `assert_output`.

> Pra não quebrar o nosso _flow_ de TDD, eu estou omitindo explicações detalhadas do `run` e do `assert_output`. Se tiver dúvidas deixe ali nos comentários.

Vamos executar o teste e ver o resultado:

```
$ bats test/hello_test.bats 
hello_test.bats
 ✗ say Hello, World
   (from function `assert_output' in file test/test_helper/bats-assert/src/assert_output.bash, line 194,
    in test file test/hello_test.bats, line 16)
     `assert_output "Hello, World"' failed
   
   -- output differs --
   expected : Hello, World
   actual   :
   --
   

1 tests, 1 failure
```

O que é bacana de usar o `assert_output` é que ele diz claramente o que era esperado na saída e o que foi realmente impresso:

```
   expected : Hello, World
   actual   :
```

Esperamos `Hello, World` mas não imprimimos coisa alguma. Isso já era de se esperar, afinal o nosso `hello.sh` é apenas um arquivo vazio com permissão de execução.

Vamos resolver isso escrevendo o clássico hello-world em bash no `src/hello.sh`:

```bash
#!/usr/bin/env bash

echo "Hello, World"
```

Executemos o teste novamente:

```
$ bats test/hello_test.bats 
hello_test.bats
 ✓ say Hello, World

1 tests, 0 failures
```

Woohool!!! 🥳🎉 O teste passou! 

### Controle de versão

Terminamos de implementar uma nova funcionalidade e todos os testes estão passando. Isso é um bom momento pra fazer um commit.

Se fizermos alguma besteira, podemos facilmente voltar para esse commit onde tudo estava funcionando.

```
git commit --all --message "Hello, World"
```

## Hello, meleu

Agora queremos que o nosso hello-world seja capaz de cumprimentar o nome que passamos como argumento para o programa. E se não passarmos nome algum, queremos continuar cumprimentando o mundo inteiro com `Hello, World`.

Lembre-se: escreva o teste primeiro!

```bash
# ... conteúdo original do test/hello_test.bats

@test "say hello to people" {
  run hello.sh meleu
  assert_output "Hello, meleu"
}
```

Executando:
```
$ bats test/hello_test.bats
hello_test.bats
 ✓ say Hello, World
 ✗ say hello to people
   (from function `assert_output' in file test/test_helper/bats-assert/src/assert_output.bash, line 194,
    in test file test/hello_test.bats, line 19)
     `assert_output "Hello, meleu"' failed

   -- output differs --
   expected : Hello, meleu
   actual   : Hello, World
   --


2 tests, 1 failure
```

**Observação**: acostume-se a não ficar irritado vendo testes falharem!

Confie no processo! O Desenvolvimento Guiado por Testes é assim, a gente vai checando as mensagens de falha dos nossos testes e isso vai guiando a nossa próxima ação.

Aqui a mensagem de erro está nos mostrando que esperávamos `Hello, meleu` mas obtivemos `Hello, World`.

Vamos resolver isso no nosso `src/hello.sh` da maneira mais _naïve_ possível:

```bash
#!/usr/bin/env bash

echo "Hello, $1"
```

E vamos conferir se os testes passam:

```
$ bats test/hello_test.bats
hello_test.bats
 ✗ say Hello, World
   (from function `assert_output' in file test/test_helper/bats-assert/src/assert_output.bash, line 194,
    in test file test/hello_test.bats, line 14)
     `assert_output "Hello, World"' failed

   -- output differs --
   expected : Hello, World
   actual   : Hello,
   --

 ✓ say hello to people

2 tests, 1 failure
```

😳

O nosso novo teste passou mas acabamos quebrando um outro.

Eu gostaria que você refletisse um pouco sobre esse caso.  Com um nosso reles hello-world podemos extrapolar para um cenário de "problema real". Pense em quantas vezes você pegou aquele seu código que está funcionando bem, e adicionou uma nova funcionalidade. Fez um rápido teste manual, ficou satisfeito com o resultado e seguiu adiante. Pouco depois você percebeu que a sua nova funcionalidade quebrou alguma outra parte do programa.

Esse é o tipo de cenário que o TDD não permite que ocorra! Como você tem testes automatizados, se você quebrar algo que antes estava funcionando, sua bateria de testes já vai te avisar.

Quando nos deparamos com essa situação de fazer quebrar um teste que estava passando, a primeira atitude que devemos tomar é desfazer nossa última alteração e pensar numa alteração melhor.

O que eu acredito que devemos fazer aqui é definir um valor default para o caso do usuário não passar nome algum. Portanto o `hello.sh` fica assim:

```bash
#!/usr/bin/env bash

echo "Hello, ${1:-World}"
```

Executando os tests:

```
$ bats test/hello_test.bats
hello_test.bats
 ✓ say Hello, World
 ✓ say hello to people

2 tests, 0 failures
```

Que maravilha! Todos os testes passando!

Quando a gente se depara com todos os testes passando, isso imediatamente deve disparar em nossa mente o desejo de **refatorar**.

### Refatoração

Vamos nos aproveitar da segurança dos testes e focar na qualidade do nosso código. O objetivo é deixá-lo mais legível e de mais fácil manutenção.

Óbvio que para um hello-world não tem como ser mais simples do que `echo "Hello, World"`, mas vou aproveitar o nosso exemplo para escrever esse código com algumas práticas que eu **sempre** uso nos meus códigos bash.

Primeiro: **todo código deve estar dentro de uma função**.

Portanto eu faria o nosso `hello.sh` assim:

```bash
#!/usr/bin/env bash

hello() {
  echo "Hello, ${1:-World}"
}

hello "$@"
```

Executando os testes:

```
$ bats test/hello_test.bats
hello_test.bats
 ✓ say Hello, World
 ✓ say hello to people

2 tests, 0 failures
```

Maravilha! Fizemos uma mudança e confirmamos que nada quebrou!

A outra prática que eu **sempre** adoto no meu código bash: **todos argumentos devem ser armazenados em uma variável com um nome significativo**.

Nosso `hello.sh` então fica assim:

```bash
#!/usr/bin/env bash

hello() {
  local name="${1:-World}"
  echo "Hello, ${name}"
}

hello "$@"
```

Execute os testes novamente e observe que nada quebrou!

Acho que cabe um novo commit aqui:

```
git commit --all --message "Hello, meleu"
```

Agora estamos prontos para implementar ainda mais _features_ no nosso hello-world...

## Um hello-world poliglota

Uma características dos sistemas Unix-like (o que inclui o GNU/Linux e o MacOS) é que a variável de ambiente `$LANG` é utilizada para determinar o idioma utilizado nas mensagens do sistema para o usuário.

Pra começar vamos avaliar o que temos nessa variável. Isso vai variar de como você configurou seu sistema. Se você usa português brasileiro, provavelmente vai ver algo assim:

```
$ echo $LANG
pt_BR.UTF-8
```

Se seu sistema está em inglês, talvez veja `en_US.UTF-8`. Esse valor depois do ponto `.` pode estar diferente.

Na real o que nos interessa aqui é apenas os dois primeiros caracteres. Dali podemos saber qual é o idioma configurado no sistema. Vamos nos aproveitar disso para criar um hello-world poliglota.

Detectaremos o idioma checando a variável `$LANG`, e se não reconhecermos o conteúdo da variável, vamos cumprimentar em inglês mesmo, com `Hello`.

Começando com o português.

### Olá

Como vimos, se nosso sistema está em português, a variável `$LANG` será algo tipo `pt_BR.UTF-8`.

Uma técnica bem útil de shell em geral (não é nem específico de BATS) é que quando queremos passar um valor para uma variável de ambiente apenas para execução de um único comando, podemos usar a seguinte estratégia:

```bash
ENV_VAR=valor meu_commando
```

Vamos nos aproveitar dessa técnica na hora de escrever nosso teste, que ficará assim:

```bash
# ... conteúdo original do test/hello_test.bats

@test "say olá to people, in Portuguese" {
  LANG=pt_BR.UTF-8 run hello.sh meleu
  assert_output "Olá, meleu"
}
```

Executando o teste:

```
$ bats test/hello_test.bats
hello_test.bats
 ✓ say Hello, World
 ✓ say hello to people
 ✗ say olá to people
   (from function `assert_output' in file test/test_helper/bats-assert/src/assert_output.bash, line 194,
    in test file test/hello_test.bats, line 24)
     `assert_output "Olá, meleu"' failed

   -- output differs --
   expected : Olá, meleu
   actual   : Hello, meleu
   --


3 tests, 1 failure
```

Conforme esperado, todos os testes que já existiam continuam passando. Apenas o novo teste falhou, e ele já nos informa o que está errado: ele espera `Olá, meleu` e nosso programa forneceu `Hello, meleu`.

Vamos resolver isso com um `if` no nosso `src/hello.sh`:

```bash
#!/usr/bin/env bash

hello() {
  local name="${1:-World}"

  # comparando com 'pt*' para considerar qualquer valor
  # que começa com 'pt' como sendo língua portuguesa.
  if [[ "$LANG" == pt* ]]; then
    echo "Olá, ${name}"
  else
    echo "Hello, ${name}"
  fi
}

hello "$@"
```

Executamos o teste:

```
$ bats test/hello_test.bats
hello_test.bats
 ✓ say Hello, World
 ✓ say hello to people
 ✓ say olá to people, in Portuguese

3 tests, 0 failures
```

#### Refatorar?

Um teste acabou de passar, e isso me desperta o desejo de refatorar. Eu dou uma olhada no código e decido que ele está nos atendendo muito bem.

Nada de otimização prematura! O código está passando nos testes e está legível o suficiente. Se no futuro ele ficar mais complexo, podemos refatorar, mas desta vez vamos seguir sem mudanças.

#### Commit

Vamos commitar:

```
git commit --all --message "Olá, meleu"
```

### Hola

Vamos agora cumprimentar como _nuestros hermanos_: em espanhol. Aqui na nossa vizinhança (ao redor do Brasil) temos muitos exemplos de países onde provavelmente os usuários terão um `$LANG` assim (estou omitindo o `.UTF-8`):

 - Argentina: `es_AR`
 - Colômbia: `es_CO`
 - Paraguai: `es_PY`
 - Uruguai: `es_UY`

Como podemos ver, todos começam com `es`, o que significa espanhol.

Portanto vamos escrever nosso teste assim:

```bash
# ... conteúdo original do test/hello_test.bats

@test "say Hola to people, in Spanish" {
  LANG=es_AR.UTF-8 run hello.sh meleu
  assert_output "Hola, meleu"
}
```

Executamos o teste:

```
$ bats test/hello_test.bats
hello_test.bats
 ✓ say Hello, World
 ✓ say hello to people
 ✓ say olá to people, in Portuguese
 ✗ say Hola to people, in Spanish
   (from function `assert_output' in file test/test_helper/bats-assert/src/assert_output.bash, line 194,
    in test file test/hello_test.bats, line 31)
     `assert_output "Hola, meleu"' failed

   -- output differs --
   expected : Hola, meleu
   actual   : Hello, meleu
   --


4 tests, 1 failure
```

OK, o _output_ não foi o que está sendo esperado pelo teste.

Vamos lá no `src/hello.sh` e resolver assim:

```bash
#!/usr/bin/env bash

hello() {
  local name="${1:-World}"

  if [[ "$LANG" == pt* ]]; then
    echo "Olá, ${name}"
  elif [[ "$LANG" == es* ]]; then
    echo "Hola, ${name}"
  else
    echo "Hello, ${name}"
  fi
}

hello "$@"
```

Executamos os testes:

```
$ bats test/hello_test.bats
hello_test.bats
 ✓ say Hello, World
 ✓ say hello to people
 ✓ say olá to people, in Portuguese
 ✓ say Hola to people, in Spanish

4 tests, 0 failures
```

Beleza, tudo passando.


#### Refatoração

Será que devemos refatorar? Estou olhando para aquela estrutura `if-elif` sempre comparando com a `$LANG` e percebo que ela está implorando pra virar um `case`. Portanto vamos refatorar sim!

O `src/hello.sh` vai ficar assim:

```bash
#!/usr/bin/env bash

hello() {
  local name="${1:-World}"

  case "$LANG" in
    pt*) echo "Olá, ${name}" ;;
    es*) echo "Hola, ${name}" ;;
    *) echo "Hello, ${name}" ;;
  esac
}

hello "$@"

```

Execute os testes e observe que tá tudo passando! 🤓

#### Commit

Vamos commitar e passar para o próximo idioma...

```
git commit --all --message "Hola, meleu"
```

### Bonjour

A essa altura do campeonato adicionar um novo idioma ficou super simples: basta descobrirmos o "código" do idioma e como dizer "Hello" em tal idioma.

Agora queremos cumprimentar em francês. O código é `fr` e o cumprimento é "Bounjour".

Eu sei que você tá louco pra ir direto lá no `hello.sh` e adicionar o caso do francês. Resista a essa tentação! Lembre-se: **primeiro o teste!**

```bash
# ... conteúdo original do test/hello_test.bats

@test "say Bonjour to people, in French" {
  LANG=fr_FR.UTF-8 run hello.sh meleu
  assert_output "Bonjour, meleu"
}
```

Execute o teste:

```
$ bats test/hello_test.bats
hello_test.bats
 ✓ say Hello, World
 ✓ say hello to people
 ✓ say olá to people, in Portuguese
 ✓ say Hola to people, in Spanish
 ✗ say Bonjour to people, in French
   (from function `assert_output' in file test/test_helper/bats-assert/src/assert_output.bash, line 194,
    in test file test/hello_test.bats, line 36)
     `assert_output "Bonjour, meleu"' failed

   -- output differs --
   expected : Bonjour, meleu
   actual   : Hello, meleu
   --


5 tests, 1 failure
```

Agora que vimos que o nosso teste está falhando podemos alterar nosso código para passar no teste.

Nosso `src/hello.sh` fica assim:

```bash
#!/usr/bin/env bash

hello() {
  local name="${1:-World}"

  case "$LANG" in
    pt*) echo "Olá, ${name}" ;;
    es*) echo "Hola, ${name}" ;;
    fr*) echo "Bonjour, ${name}" ;;
    *) echo "Hello, ${name}" ;;
  esac
}

hello "$@"
```

Executando os testes:

```
$ bats test/hello_test.bats
hello_test.bats
 ✓ say Hello, World
 ✓ say hello to people
 ✓ say olá to people, in Portuguese
 ✓ say Hola to people, in Spanish
 ✓ say Bonjour to people, in French

5 tests, 0 failures
```

Topzera da balada! Todos os testes passando!

#### Refatoração

Está tudo certinho, os testes estão passando e talz... Mas aquele `case` está começando a me incomodar...

Eu acho que uma função hello-world deveria ser mais simples de ler do que ter essa pequena  maçaroca de `case` ali.

💡 Tive uma ideia: vamos criar uma função chamada `greeting()` que será responsável por imprimir o cumprimento no idioma correto!

Assim nosso hello-world vai voltar a ser tão simples quanto um `echo "$(greeting), ${name}"`

Portanto o `src/hello.sh` fica assim:

```bash
#!/usr/bin/env bash

hello() {
  local name="${1:-World}"
  echo "$(greeting), ${name}"
}

greeting() {
  case "$LANG" in
    pt*) echo "Olá" ;;
    es*) echo "Hola" ;;
    fr*) echo "Bonjour" ;;
    *) echo "Hello" ;;
  esac
}

hello "$@"
```

Vamos executar os testes:

```
$ bats test/hello_test.bats
hello_test.bats
 ✓ say Hello, World
 ✓ say hello to people
 ✓ say olá to people, in Portuguese
 ✓ say Hola to people, in Spanish
 ✓ say Bonjour to people, in French

5 tests, 0 failures
```

Que delicinha! Tudo passando!

Perceba que eu não precisei ir na linha de comando ficar testando cada um dos idiomas pra ver se algo quebrou. Bastou executar a bateria de testes pronto! Sinta o conforto e a gostosura disso! 🥰

#### Commit

Vamos pra mais um commitzinho:

```
git commit --all --message "Hola, meleu"
```

### Hallo, Ciao, Konnichiwa

Você já entendeu o espírito da coisa, né? Então fica aí como um pequeno exercício pra você treinar esse ciclo:

1. Escreva um teste para um novo idioma
2. Execute o teste e observe a mensagem de falha
3. Adicione o novo cumprimento lá no `case`
4. Certifique-se que o teste passou
5. Se encontrar uma maneira melhor de organizar o código, **refatore**.

E se quiser refatorar mais seguindo adiante com esse _over-engineering_ do nosso hello-world, manda brasa. Você pode, por exemplo, tentar usar um array associativo com os possíveis cumprimentos...

Nesse processo você vai ver que reconfortante que é ter testes automatizados e não precisar testar "na mão".

Lembre-se, durante a refatoração o flow é esse:

1. faz uma alteração no código
2. já roda os testes em seguida
3. se quebrar algum teste
    - desfazer a alteração
    - escolher uma melhor e voltar pro passo 1


## Recapitulando

### Test-Driven Development

- Escreva o teste antes de ter o código que será testado.
- Você **PRECISA** ver seu teste falhando
    - para que saibamos que temos testes relevantes;
    - notar que ele produz descrições de falha que são fáceis de entender e dão dicas do que devemos fazer.
- Escreva a menor quantidade possível de código para fazer seu teste passar.
- Por fim refatore, com a segurança dos seus testes automatizados.


## Palavras finais

Claro que um hello-world é extremamente trivial comparado com problemas da "vida real". O objetivo aqui foi apenas dar uma introduzida no TDD e como usar o BATS pra isso. Escolhi um problema simples exatamente para que pudéssemos focar nestes temas.

Espero que artigo tenha deixado você atiçado para se aprofundar no tema

Dar uma lida no [README do bats-assert](https://github.com/bats-core/bats-assert) pode ser bem legal para você ter uma noção de outras asserções que você pode usar.

Outra coisa importante de se ter em mente: programas em bash geralmente manipulam arquivos. Portanto uma lida no [README do bats-file](https://github.com/bats-core/bats-file) também será muito útil.

Deixe aí nos comentários se você gostaria de ver mais artigos nesse estilo por aqui.


## Referências

- [Learn Go with Tests](https://quii.gitbook.io/learn-go-with-tests) - livro que ensina Golang com TDD. Foi de onde peguei a inspiração para usar esse "hello-world poliglota".
