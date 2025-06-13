---
title: Setup básico do BATS
description: >
  Como configurar o BATS para começar a ter testes automatizados para o seus programas Bash.
tags:
  - testes
date: 2025-06-13T15:00:00-03:00
cover:
  image: "img/bats-logo.png"
  alt: BATS logo
---


O objetivo principal desse artigo é mostrar um setup bem básico do BATS para que você possa facilmente testar seu código bash.

Não desenvolveremos funcionalidade alguma, apenas mostrarei uma maneira conveniente de fazer este setup.

> ## AVISO!
>
> Se você já leu o artigo [Aprenda TDD no Bash](../tdd-bash) não encontrará novidade alguma aqui!
>
> Escrevi esse artigo aqui pois percebi que o artigo original estava muito longo. Portanto resolvi quebrá-lo em dois:
>
> 1. Setup do BATS
> 2. [Fluxo de _Test-Driven Development_](../hello-tdd)

## O que é BATS?

[BATS](https://github.com/bats-core/bats-core) signfica Bash Automated Testing System. É um _framework_ de testes para bash, que permite que verifiquemos se o nosso programa está se comportando da maneira que queremos.

 Estou sendo breve pois já quero partir pra ação. Mas se você quiser, pode obter mais informações sobre o BATS na [documentação oficial](https://bats-core.readthedocs.io/).

### Instalando o BATS

A maioria das distribuições GNU/Linux possuem um pacote chamado `bats` que você pode instalar usando o gerenciador de pacotes oficial.

Eu particularmente prefiro instalar via [Homebrew](https://brew.sh/), pois é um método que funciona tanto pra GNU/Linux como pra MacOS.

```bash
brew install bats-core
```

Fique a vontade para instalar da maneira que você preferir.

## Não quero ler isso tudo! Me diz logo o que tenho que fazer!

Pula lá pro [Resumo](#resumo) que você vai ver um checklist. Mas se quiser entender o que está fazendo, talvez seja melhor continuar lendo daqui mesmo...


## Setup básico de um projeto com BATS

Antes de tudo, vamos preparar um diretório onde colocaremos o código do nosso projeto:

```bash
mkdir try-bats
cd try-bats
```

Pra não bagunçar o projeto misturando código do nosso programa com o código dos testes automatizados, vamos criar um diretório específico para os testes:

```bash
# assumindo que já estamos no 'try-bats/'
mkdir test
```

Criaremos nosso primeiro teste para checar se estamos aptos a executar nosso script. Para isso criamos o arquivo `test/my_code_test.bats`.

**OBSERVAÇÃO**: o arquivo tem a extensão `.bats` mas o conteúdo é basicamente bash! O BATS não é uma "linguagem" nova que você tem que aprender. A única coisa diferente que você verá num arquivo `.bats` é a declaração dos testes nesse formato: `@test "nome do teste"`. Todo o resto é apenas bash.

Aqui está o `test/my_code_test.bats`:

```bash
# eu prefiro colocar a descrição em inglês,
# fique a vontade pra colocar em português.
@test "can run the script" {
  ./my_code.sh
}
```

É isso mesmo! Nosso teste quer apenas verificar se conseguimos executar nosso programa, por isso está simplesmente chamando `./my_code.sh`! Vamos executar esse teste e ver o output:

```
$ bats test/my_code_test.bats 
my_code_test.bats
 ✗ can run the script
   (in test file test/my_code_test.bats, line 2)
     `./my_code.sh' failed with status 127
   /home/meleu/src/try-bats/test/my_code_test.bats: line 2: ./my_code.sh: No such file or directory

1 test, 1 failure
```

💥 Já começamos com uma falha!

Observe que no meio daquela mensagem temos o motivo da falha: `./my_code.sh: No such file or directory`.

O arquivo não existe. Pois então vamos criá-lo:

```bash
touch my_code.sh
```

E executar o teste novamente:

```
$ bats test/my_code_test.bats 
my_code_test.bats
 ✗ can run the script
   (in test file test/my_code_test.bats, line 2)
     `./my_code.sh' failed with status 126
   /home/meleu/src/try-bats/test/my_code_test.bats: line 2: ./my_code.sh: Permission denied

1 test, 1 failure
```

Outra falha. Novamente com uma dica do que devemos fazer: `./my_code.sh: Permission denied`.

Se temos um `Permission denied`, só nos resta dar permissão de execução pro arquivo e executar o teste novamente:

```
$ # dando permissão de execução
$ chmod a+x my_code.sh 

$ # executando o teste
$ bats test/my_code_test.bats 
my_code_test.bats
 ✓ can run the script

1 test, 0 failures
```

🥳🎉 Agora sim! Nosso primeiro teste passou!

Tá bom... Isso não é lá grande coisa. Estamos apenas validando que um arquivo tem permissão de execução. Essa parte foi só pra termos uma mini injeção de dopamina ao ver um teste passando.

### Organizando os arquivos do projeto

Já que criamos um diretório para os testes, vamos também criar um diretório para o código executável e colocar o arquivo lá:

```bash
mkdir src
mv my_code.sh src/my_code.sh
```

Dessa forma teremos essa estrutura de diretórios:

```
$ tree
.
├── src
│   └── my_code.sh
└── test
    └── my_code_test.bats

2 directories, 2 files
```

Agora vamos executar nosso teste novamente:

```
$ bats test/my_code_test.bats 
my_code_test.bats
 ✗ can run the script
   (in test file test/my_code_test.bats, line 2)
     `./my_code.sh' failed with status 127
   /home/meleu/src/try-bats/test/my_code_test.bats: line 2: ./my_code.sh: No such file or directory

1 test, 1 failure
```

Ooops! Nosso teste voltou a falhar por não encontrar o arquivo!

Isso está acontecendo pois não estamos especificando o caminho até o arquivo! Vamos resolver isso:

```bash
@test "can run the script" {
  ./src/my_code.sh
}
```

Executando o teste:

```
$ bats test/my_code_test.bats 
my_code_test.bats
 ✓ can run the script

1 test, 0 failures
```

OK. Nosso arquivo foi encontrado e o teste passou. Mas ainda assim ficamos com aquela sensação de que essa solução não parece muito robusta.

Vamos tentar por exemplo entrar no diretório do `test/` e executar o teste de lá:

```
$ cd test/

$ bats my_code_test.bats 
my_code_test.bats
 ✗ can run the script
   (in test file my_code_test.bats, line 2)
     `./src/my_code.sh' failed with status 127
   /home/meleu/src/try-bats/test/my_code_test.bats: line 2: ./src/my_code.sh: No such file or directory

1 test, 1 failure
```

O teste simplesmente não encontrou nosso codigo, só por que mudamos de diretório... Não queremos um teste tão "frágil" assim. Vamos resolver isso criando um `setup` pro nosso teste.

### Fazendo o `setup`

Em um arquivo BATS, a função `setup` tem um significado especial: ela é uma função que será executada antes de cada teste (se quiser mais detalhes veja a [documentação aqui](https://bats-core.readthedocs.io/en/stable/writing-tests.html#setup-and-teardown-pre-and-post-test-hooks)).

Vamos criar uma função de `setup` para adicionar o caminho pro nosso executável direto no `PATH`. Dessa forma podemos executar o `my_code.sh` sem nos preocupar em especificar o caminho.

Nosso `test/my_code_test.bats` então fica assim:

```bash
setup() {
  PATH="${BATS_TEST_DIRNAME}/../src:${PATH}"
}

@test "can run the script" {
  my_code.sh
}
```

Ali no `setup` estamos nos aproveitando da variável `$BATS_TEST_DIRNAME`, que o BATS já deixa preenchida com o caminho pro diretório do arquivo de teste. A partir desse diretório vamos para `../src`, que é onde está o nosso executável.

Observe que o teste também está sendo feito com uma chamada direta ao `my_code.sh`, sem especificar o caminho. Nós podemos fazer isso pois o `setup` já deixou o `PATH` devidamente preparado.

Vamos conferir se isso realmente funciona:

```
$ bats my_code_test.bats 
my_code_test.bats
 ✓ can run the script

1 test, 0 failures
```

Maravilha! Estamos no caminho certo!

### O que vimos até agora

- Criamos o diretório `try-bats` para começar um novo projeto do zero.
- Colocamos nosso arquivo de teste no diretório `test/`.
- Nosso arquivo executável ficará em `src/`.
- Usamos o `setup` para atualizar o `PATH` com o caminho para o executável.
- Usamos a variável `$BATS_TEST_DIRNAME` pra pegar o caminho do diretório onde está nosso arquivo de teste.
- Nosso teste chama o arquivo executável usando apenas o nome do arquivo (sem precisar especificar o caminho completo)

Só pra lembrar, no momento nosso `test/my_code_test.bats` está assim:

```bash
setup() {
  PATH="${BATS_TEST_DIRNAME}/../src:${PATH}"
}

@test "can run the script" {
  my_code.sh
}
```

E o nosso `src/my_code.sh` nada mais é que um arquivo vazio com permissão de execução.


## BATS helpers

O projeto BATS oferece helpers com algumas conveniências que podem nos ajudar bastante na escrita de testes.

No nosso projeto nós queremos verificar se a saída do programa é `Hello, World`. Fazemos isso criando **asserções** sobre o que o programa gera como saída.

Para criar asserções vamos precisar do helper `bats-assert`. Vamos usar também o `bats-support` para que ele nos forneça mensagens de erro/falha amigáveis, dando mais clareza sobre onde nosso código está quebrando.

Para utilizarmos estes helpers, vamos primeiro fazer com que nosso projeto fique em um repositório git:

```bash
# assumindo que estamos no diretório 'hello-tdd'
git init
```

Para "instalar" BATS helpers vamos adicioná-los como submódulos:

```bash
# bats-assert: responsável pelas asserções
git submodule add \
  https://github.com/bats-core/bats-assert.git \
  test/test_helper/bats-assert

# bats-support: responsável por mensagems de falha mais amigáveis
git submodule add \
  https://github.com/bats-core/bats-support.git \
  test/test_helper/bats-support

# bats-core: o bats propriamente dito (útil para usarmos em pipelines)
git submodule add \
  https://github.com/bats-core/bats-core.git \
  test/bats
```

Observe que estamos também "instalando" como submódulo o próprio `bats-core`. Isso é útil, por exemplo, para rodar testes diretamente numa pipeline de Integração Contínua (mas isso é papo para um outro artigo).

No momento nossa estrutura de diretórios está assim:

```
hello-tdd/
├── src/
└── test/
    ├── bats/        # 👈 arquivos do bats-core aqui
    └── test_helper/ # 👈 bats-support e bats-assert aqui
```


### Fazendo asserções

Só para ilustrar vamos testar um programa que imprime `Hello, World` na tela.

Para já ir preparando o psicológico para o fluxo de _Test-Driven Development_, vamos começar escrevendo nosso teste (antes mesmo do código que vai imprimir `Hello, World`).

Primeiro vamos carregar os helpers no nosso `setup`, dessa forma:

```bash
setup() {
  load 'test_helper/bats-support/load'
  load 'test_helper/bats-assert/load'

  PATH="${BATS_TEST_DIRNAME}/../src:${PATH}"
}
```

Para testar tal programa precisamos fazer **asserções** sobre sua saída. Nosso teste ficará assim:

```bash
# ... conteúdo original do test/my_code_test.bats

@test "say Hello, World" {
  run my_code.sh # 👈 note que estamos usando um `run` aqui!
  assert_output "Hello, World"
}
```

Nesse código estamos usando o `run` para chamar o nosso programa. Isso nos traz várias conveniências, como por exemplo automaticamente salvar a saída gerada pelo programa para que possamos verificar com o `assert_output`.

> Estou omitindo explicações detalhadas do `run` e do `assert_output`. Se tiver dúvidas deixe ali nos comentários.

Vamos executar o teste e ver o resultado:

```
$ bats test/my_code_test.bats 
my_code_test.bats
 ✓ can run the script
 ✗ say Hello, World
   (from function `assert_output' in file test/test_helper/bats-assert/src/assert_output.bash, line 194,
    in test file test/my_code_test.bats, line 16)
     `assert_output "Hello, World"' failed
   
   -- output differs --
   expected : Hello, World
   actual   :
   --
   

2 tests, 1 failure
```

O que é bacana de usar o `assert_output` é que ele diz claramente o que era esperado na saída e o que foi realmente impresso:

```
   expected : Hello, World
   actual   :
```

Esperamos `Hello, World` mas não imprimimos coisa alguma. Isso já era de se esperar, afinal o nosso `my_code.sh` é apenas um arquivo vazio com permissão de execução.

Vamos resolver isso escrevendo o clássico hello-world em bash no `src/my_code.sh`:

```bash
#!/usr/bin/env bash

echo "Hello, World"
```

Executemos o teste novamente:

```
$ bats test/hello_test.bats 
hello_test.bats
 ✓ can run the script
 ✓ say Hello, World

2 tests, 0 failures
```

Woohool!!! 🥳🎉 O teste passou! 

E aqui finalizamos nosso pontapé inicial de como fazer um setup para começar a usar o BATS para realizar testes automatizados em seus projetos bash.


## Resumo

Aqui está um resumo rápido e direto do que você deve fazer para ter um setup básico de testes para seu projeto bash usando BATS.

### Checklilst

- [ ] Crie um diretório para o projeto
- [ ] Crie os diretórios `test` e `src`
- [ ] Inicie um repo git
- [ ] Instale BATS
- [ ] Instale os BATS helpers

### Passo a passo

```bash
# diretório do projeto
mkdir my_project
cd my_project

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

Agora a estrutura de diretórios do seu projeto estará assim:

```
.
├── src/
└── test/
    ├── bats/        # 👈 arquivos do bats-core aqui
    └── test_helper/ # 👈 bats-support e bats-assert aqui
```

Para facilmente invocar o `my_code.sh` em qualquer um dos seus testes, coloque isso na função `setup` do seu `test/my_code_test.bats`:

```bash
setup() {
  load 'test_helper/bats-support/load'
  load 'test_helper/bats-assert/load'

  PATH="${BATS_TEST_DIRNAME}/../src:${PATH}"
}

# Agora você consegue facilmente chamar seu código
@test "can run the script" {
  run my_code.sh
  # veja como fazer asserções no README de
  # https://github.com/bats-core/bats-assert
}
```


## Conclusão

Aprendemos como iniciar um projeto bash configurando o BATS como o framework de testes. Também instalamos os helpers para podermos fazer asserções facilmente.

No próximo artigo começaremos a usar a metodologia de Test-Driven Development para implementar novas funcionalidades no nosso "Hello, World".


## Referências

- [BATS Tutorial](https://bats-core.readthedocs.io/en/stable/tutorial.html) - o tutorial oficial do BATS.