---
title: "Crie aplicações CLI robustas com o Bashly"
description: >
  Chega daquela trabalheira toda pra fazer um CLI decente usando apenas Bash.
tags:
  - bashly
  - codigo
  - bash
  - ferramentas
date: 2025-08-29T11:15:14-03:00
cover:
  image: "img/bashly-logo.png"
  alt: Logo do Bashly
---

Neste artigo conheceremos o Bashly, um framework que permite que criemos aplicações CLI bem robustas usando apenas bash.

Faremos isso com uma abordagem bem mão-na-massa, desenvolvendo uma aplicação muito simples porém com um "acabamento" profissional.

## Por que usar Bashly?

Imagine o seguinte cenário...

Queremos criar um gerador de números aleatórios. Em bash isso é tão simples que podemos fazer com um simples comando: `echo $RANDOM`.

A gente nem precisaria escrever um script só pra isso. No entanto começamos a pensar em outras funcionalidades, por exemplo: queremos especificar o número máximo a ser gerado.

Imaginando que nosso programa se chama `rndm`, poderíamos ter os seguintes usos:

```bash
# simular lançamento de um dado:
rndm --max 6

# simular um cara-ou-coroa:
rndm --max 2
```

Talvez você até saiba como gerar números aleatórios dentro de uma faixa específica, a lógica pra fazer isso nem é complicada. Mas se algum dia já escreveu um programa bash fazendo _parsing_ de `--opções` da linha de comando você sabe bem o que vai acontecer: o nosso simples programinha vai explodir em complexidade só por conta do código necessário para lidar com essas opções.

Ah! E já que você adicionou opções ao seu programa, você também tem que providenciar um `--help` para que o usuário saiba quais são as opções disponíveis e como usá-las corretamente.

Outra coisa: se você vai aceitar input do usuário, é importante validar o que ele está enviando para o seu programa.

No final das contas você vai gastar mas energia mental lidando com todas essas minúcias de parsing de opções e help do que com o problema que você realmente quer resolver: gerar números aleatórios.

É pra resolver essa dor que o Bashly foi criado! Com ele teremos facilidade para:

- fazer parsing de `--opções`
- criar mensagens de help facilmente
- validar input
- verificar dependências
- e muitas outras coisas típicas de uma aplicação CLI robusta.

Delegue essas tarefas tediosas para o Bashly e foque na lógica do problema que você realmente precisa resolver.

Para ilustrar como criar um CLI robusto usando o Bashly, criaremos um programa gerador de número randômicos. Ele começará bem simples, mas irá gradualmente receber muitas funcionalidades interessantes ao longo do tutorial.

> **Observação**: para usar o Bashly é necessário saber lidar com arquivos YAML (o que é uma coisa bem simples).

## Instalando o Bashly

O Bashly é uma gem do Ruby. Na ecossistema Ruby nós chamamos de _gem_ os pacotes de software (como um npm package para o NodeJS, ou um crate para o Rust).

**Observação**: apesar de ser desenvolvido em Ruby, você não precisa saber nada de Ruby para usar o Bashly!

O Bashly depende que você tenha o Ruby instalado numa versão 3.2 ou maior. Execute `ruby --version` no seu terminal, se sua versão for igual ou maior que 3.2 você já está pronto pra instalar o Bashly. Caso contrário você precisará instalar uma versão adequada do Ruby.

Eu gosto muito de usar "runtime version managers" como o [mise](https://mise.jdx.dev/) (uso e recomendo) ou [asdf](https://asdf-vm.com) para instalar interpretadores e compiladores em diversas versões. Recomendo que você faça o mesmo para instalar o Ruby numa versão 3.2 ou maior.

No caso do mise, eu simplesmente faço algo assim:

```bash
# instalando ruby 3.4 e atribuindo como default
mise use --global ruby@3.4
```

Uma vez que o Ruby tá instalado, vamos instalar o Bashly:

```bash
gem install bashly
```

Pra conferir que está tudo certinho:

```
$ bashly --version
1.3.2
```

No momento da escrita desse artigo a versão do Bashly é 1.3.2.

### IMPORTANTE: versão do Bash

O código final gerado pelo Bashly faz uso de arrays associativos e outras features que dependem do Bash na versão maior ou igual a 4.2 (que foi lançada em 2011).

Se você está usando uma distro Linux, muito provavelmente você já está usando uma versão compatível.

Se você está usando MacOS, seu Bash provavelmente está "travado" na versão 3.2.57. Mas não se preocupe: um simples `brew install bash` já resolve o problema (estou assumindo que você esteja usando o [Homebrew](https://brew.sh)).

## Iniciando um projeto

Vamos começar criando um diretório para o nosso projeto

```bash
mkdir rndm
cd rndm
```

Uma maneira de iniciar um projeto com o bashly, é usar `bashly init`, isso irá criar um arquivo chamado `src/bashly.yml`. Se você fizer isso observará que o arquivo já vem com muita informação e isso pode ser um pouco confuso para um primeiro contato.

Aqui nós vamos escrever o `bashly.yml` totalmente "na mão", e vamos aprender cada configuração com calma. Portanto abra o `src/bashly.yml` e remova todo o conteúdo. Em seguida adicione apenas isso:

```yaml
name: rndm
help: Prints a random number
version: 0.0.1
```

Agora basta executarmos `bashly generate`, e teremos um output assim:

```
$ bashly generate
creating user files in src
created src/root_command.sh
created ./rndm
run ./rndm --help to test your bash script
```

Pois vamos fazer exatamente o que ele está sugerindo ali no final da mensagem:

```
$ ./rndm --help
rndm - Prints a random number

Usage:
  rndm
  rndm --help | -h
  rndm --version | -v

Options:
  --help, -h
    Show this help

  --version, -v
    Show version number
```

🤩 - Olha isso!!

Não escrevemos uma única linha de bash e veja que help lindão que já temos!

Vamos entender rapidamente o que acabou de acontecer. De maneira simplificada, o comando `bashly generate` fez o seguinte:

1. leu o conteúdo do `src/bashly.yml`
2. entendeu que queremos criar um script chamado `rndm`
3. leu a descrição do script no `help:`
4. leu a versão do script no `version:`
5. criou um arquivo `src/root_command.sh`
6. gerou o script final `rndm`

Uma coisa que já percebemos é que o `rndm` já foi gerado com as funcionalidades de `--help` e  `--version`.

Outra coisa importante aqui é que o script final, o `rndm`, é totalmente "auto-contido". O que significa que você pode distribuí-lo e qualquer pessoa que tenha o bash instalado pode executá-lo (contando que você não introduza dependências externas, mas vamos falar disso daqui a pouco).

Agora vamos dar uma olhadinha no `src/root_command.sh`:

```bash
echo "# This file is located at 'src/root_command.sh'."
echo "# It contains the implementation for the 'rndm' command."
echo "# The code you write here will be wrapped by a function named 'root_command()'."
echo "# Feel free to edit this file; your changes will persist when regenerating."
inspect_args
```

Fazendo uma tradução livre do que está naqueles comentários:

```
Esse arquivo está localizado em 'src/root_command.sh'.
Ele contem a implementação do comando 'rndm'.
O código que você escrever aqui ficará dentro de uma função chamada 'root_command()'
Fique a vontade para editar este arquivo, suas mudanças serão persistidas ao regenerar.
```

Vamos falar sobre estes pontos com um pouco mais de detalhes...

Quando executamos o `bashly generate` o arquivo `src/root_command.sh` foi criado, e é aqui que devemos colocar a lógica do nosso programa.

Todo o conteúdo desse arquivo ficará dentro de uma função chamada `root_command()`. E se dermos uma olhadinha no arquivo `rndm` que foi gerado, ali pela linha 10, confirmaremos que isso realmente aconteceu:

```bash
#!/usr/bin/env bash
# ...

root_command() {
  # src/root_command.sh
  echo "# This file is located at 'src/root_command.sh'."
  echo "# It contains the implementation for the 'rndm' command."
  echo "# The code you write here will be wrapped by a function named 'root_command()'."
  echo "# Feel free to edit this file; your changes will persist when regenerating."
  inspect_args

}

# ...
```

Uma vez que o que o arquivo `src/root_command.sh` é criado, o Bashly não mexe mais nele. Podemos editá-lo a vontade que nosso código permanecerá intacto mesmo após executarmos `bashly generate` novamente.

Pois bem, esse conteúdo que vem no arquivo é só para nos avisar destas coisas. Já podemos começar a implementar a funcionalidade que queremos. Mas primeiro acho que é uma boa começarmos a versionar nosso projeto.

### Controle de versão

É bom usarmos um sistema de controle de versão, pois isso vai nos ajudar a acompanhar a evolução do projeto. Portanto vamos iniciar um repositório git e fazer um commit do que fizemos até agora:

```bash
git init
git add .
git commit -m 'Starting bashly project'
```

Observação: por uma questão de fluidez no texto, ao longo desse tutorial eu não irei me preocupar em ficar atualizando o `version: 0.0.1` dentro do nosso `src/bashly.yml`. Portanto nesse começo nosso versionamento será unicamente via git.

### Gerando números aleatórios

Vamos começar apagando todo o conteúdo do arquivo `src/root_command.sh` e finalmente colocar nosso super código gerador de números aleatórios:

```bash
echo "$RANDOM"
```

Vamos agora gerar o script novamente e conferir o resultado:

```
$ bashly generate
creating user files in src
skipped src/root_command.sh (exists)
created ./rndm
run ./rndm --help to test your bash script
```

Observe dessa vez o Bashly não criou o `src/root_command.sh` (_skipped_), pois o arquivo já existe e é onde nós vamos trabalhar o nosso código.

Apenas o script final `rndm` foi gerado novamente, dessa vez atualizado com a nosso código para imprimir um número aleatório. Portanto se executarmos o programa algumas vezes, veremos que ele realmente gera um número aleatório a cada execução:

```
$ ./rndm
8783

$ ./rndm
32008

$ ./rndm
12550
```

✅ Pronto! É só isso que queremos no momento!

Faça o commit dessa mudança e vamos para a próxima feature.

## Lidando com `--opções`

Algumas pessoas levam esse negócio de aleatoriedade muito a sério (especialmente aquelas que lidam com criptografia). Inclusive existe um serviço na web chamado [random.org](https://random.org) que se descreve como "um serviço de número aleatório **de verdade** que gera aleatoriedade a partir do ruído atmosférico". Bom, eu não entendo muito bem o que esse negócio de "ruído atmosférico" quer dizer, mas como o site existe desde 1998 e está funcionando até hoje, estou assumindo que eles são realmente bons no que fazem.

Uma coisa legal é que o site oferece um endpoint onde podemos obter um número aleatório. Aqui está um exemplo de como podemos fazer:

```bash
curl "https://www.random.org/integers/?num=1&min=0&max=32767&col=1&base=10&format=plain"
```

Se quiser entender no detalhe cada parâmetro que estamos passando para o endpoint, você pode ver a [documentação oficial no próprio site](https://www.random.org/clients/http/api/). Mas se quiser apenas focar no aprendizado do Bashly, é só continuar comigo aqui no tutorial...

Vamos imaginar que alguns dos usuários do nosso programa são bastante exigentes na questão da aleatoriedade e pra eles um simples `echo $RANDOM` não é suficiente. Para atender a estes usuários vamos fazer com que nosso programa solicite números aleatórios ao random.org.

O que eu tenho em mente para atender a este requisito é oferecer a opção `--web` para dizermos ao nosso programa que queremos um número da web (do site random.org).

O primeiro passo é especificar a flag `--web` no nosso `src/bashly.yml`, assim:

```yaml
name: rndm
help: Prints a random number
version: 0.0.1

# especificando flags
flags:
    # versão longa:
  - long: --web
    # também na versão curta:
    short: -w
    help: Get the random number from <https://random.org>.
```

Só com esse YAML já podemos rodar um `bashly generate` só pra ver como vai ficar nosso help:

```
$ bashly generate
creating user files in src
skipped src/root_command.sh (exists)
created ./rndm
run ./rndm --help to test your bash script

$ ./rndm --help
rndm - Prints a random number

Usage:
  rndm [OPTIONS]
  rndm --help | -h
  rndm --version | -v

Options:
  --web, -w
    Get the random number from <https://random.org>.

  --help, -h
    Show this help

  --version, -v
    Show version number
```

Olha que delicinha esse help sendo gerado com apenas algumas linhas no nosso YAML!

Agora vamos entender como "pegar" esse `--web` no nosso programa.

Quando passamos uma flag para o nosso programa, o Bashly coloca isso num array associativo chamado `$args`, onde cada chave é exatamente o nome da flag. Ou seja, ao passarmos `--web` ou `-w` na linha de comando podemos obter essa informação através do `${args[--web]}` lá no nosso código. Como se trata apenas de uma flag booleana, sem argumento algum, essa variável terá o valor `1` se for o `--web` for passada na linha de comando.

Vamos ver isso no código do `src/root_command.sh`:

```bash
# se usarmos 'rndm --web' ou 'rndm -w',
# o ${args[--web]} terá o valor '1'
if [[ "${args[--web]}" == 1 ]]; then
  curl \
    --silent \
    --location \
    "https://www.random.org/integers/?num=1&min=0&max=32767&col=1&base=10&format=plain"
else
  echo "$RANDOM"
fi
```

Agora podemos gerar o script novamente, com `bashly generate`, 

> ### Agilizando o `bashly generate`.
> 
> Você vai começar a notar que precisaremos executar o `bashly generate` a cada alteração nos arquivos. Uma maneira de agilizar esse processo é simplesmente abrir um outro terminal e executar:
>
> ```
> bashly generate --watch
> ```
>
> Dessa forma ele fica monitorando mudanças nos arquivos relevantes e já gera o script final automaticamente.

Vejamos se o `--web` realmente funciona:

```
$ # números gerados localmente
$ ./rndm
2934

$ ./rndm
16891

$ # números vindos de random.org
$ ./rndm --web
18253

$ ./rndm -w
137
```

Se você executar os comandos acima, vai observar que quando usa `rndm --web`, a resposta demora alguns milissegundos a mais, pois ele está indo buscar o número na web. Essa latência é esperada quando estamos lidando com sistemas distribuídos, então não temos muito o que fazer quanto a isso...

**Importante**: mesmo que você use a versão curta `-w`, a chave do array será **sempre** uma referência a versão longa, portanto no código usamos sempre `${args[--web]}`.

Podemos considerar essa feature como pronta. Portanto agora é um bom momento para mais um commit.

## Especificando dependências

Quando adicionamos a opção de pegar um número da web, acabamos introduzindo uma dependência: o comando `curl`.

Se executarmos nosso programa em um ambiente sem o `curl`, ele vai bugar com uma mensagem desse tipo:

```
$ # executando num ambiente sem o 'curl' instalado
$ ./rndm --web
./rndm: line 17: curl: command not found
```

Realmente sem o `curl` não tem como usar nosso script pra buscar o número na web. Mas não queremos que nosso usuário veja uma mensagem feiosa dessas.

Pra melhorar essa situação, vamos deixar explícito no nosso `src/bashly.yml` que nosso programa depende do `curl`. Assim o Bashly produz uma mensagem mais clara quando há uma dependência faltando.

```yaml
help: Prints a random number
version: 0.0.1

# especificando dependências
dependencies:
  - curl

flags:
  - long: --web
    short: -w
    help: Get the random number from <https://random.org>.
```

> **Observação**: a partir de agora estarei considerando que você está usando o `bashly generate --watch`, ou então sempre lembrando de gerar o script manualmente a cada alteração nos arquivos.

Vejamos o output quando executamos nosso script num ambiente sem o `curl` instalado:

```
$ # executando em um ambiente sem o 'curl'
$ ./rndm --web
missing dependency: curl
```

Isso 👆 é um pouco melhor do que um "command not found" esquisitão, não acha?

Mande mais um commit aí e vamos em frente...

## Modularizando código

Vamos dar uma olhada no nosso código novamente:

```bash
if [[ "${args[--web]}" == 1 ]]; then
  curl \
    --silent \
    --location \
    "https://www.random.org/integers/?num=1&min=0&max=32767&col=1&base=10&format=plain"
else
  echo "$RANDOM"
fi
```

Apesar do nosso código ser relativamente simples, eu já estou querendo dar nomes para estas operações. Exemplo: ao invés de chamar aquele `curl` grandão, eu quero simplesmente chamar `get_random_number_from_web`.

Para atingir esse objetivo vamos precisar criar funções, e para isso vamos criar um diretório assim:

```bash
mkdir -p src/lib/
```

Agora vamos criar um arquivo chamado `src/lib/random_number_functions.sh` e colocar nele o código referente aos dois tipos de obtenção de números aleatórios que nós temos:

```bash
# src/lib/random_number_functions.sh

generate_random_number() {
  echo "$RANDOM"
}

get_random_number_from_web() {
  curl \
    --silent \
    --location \
    "https://www.random.org/integers/?num=1&min=0&max=32767&col=1&base=10&format=plain"
}
```

Ao fazer isso 👆 já podemos alterar o nosso `src/root_command.sh` para ficar bem mais agradável de se ler:

```bash
# src/root_command.sh

if [[ "${args[--web]}" == 1 ]]; then
  get_random_number_from_web
else
  generate_random_number
fi
```

Gere e execute o `rndm` novamente para certificar-se de que tudo está funcionando conforme o esperado.

Uma coisa legal que vimos aqui é que o Bashly pegou o conteúdo de `src/lib/random_number_functions.sh` e colocou na versão final do script (o arquivo `rndm`). Por isso que conseguimos chamar as funções que criamos sem precisar ficar se preocupando em fazer `source` de arquivos.

De fato o Bashly pega o conteúdo de qualquer arquivo `src/lib/*.sh`, e coloca no script final. Portanto essa é uma excelente maneira de você modularizar seu código, permitindo que cada arquivo tenha um objetivo bem definido e específico, deixando seu código mais organizado e legível.


## Usando `--opções-com argumento`

Muitas vezes queremos gerar um número aleatório até um certo limite. Por exemplo para simular o lançamento de um dado de 6 lados, muito comum em jogos de tabuleiro. Nesse caso acho que faz sentido que nosso CLI tenha uma interface assim:

```bash
# gera número aleatório entre 1 e 6
rndm --max 6
```

Para isso temos que especificar no nosso YAML que queremos uma flag que aceita um argumento:

```yaml
name: rndm
help: Prints a random number
version: 0.0.1

dependencies:
  - curl

flags:
  - long: --web
    short: -w
    help: Get the random number from <https://random.org>.

  # especificando uma flag que aceita um argumento
  - long: --max
    arg: max_num
    help: Specifies the maximum number to be generated
```

Antes de escrever qualquer código novo vamos dar uma conferida em como ficou o help:

```
$ ./rndm --help
rndm - Prints a random number

Usage:
  rndm [OPTIONS]
  rndm --help | -h
  rndm --version | -v

Options:
  --web, -w
    Get the random number from <https://random.org>.

  --max MAX_NUM
    Specifies the maximum number to be generated

  --help, -h
    Show this help

  --version, -v
    Show version number
```

Um detalhe interessante é que como passamos aquele `arg: max_num` lá no YAML, o Bashly já faz duas coisas:

1. entende que a flag `--max` requer um argumento,
2. já monta um help especificando esse requerimento

Observação: o nome `max_num` não será utilizado no nosso código, ele é usado apenas no help. No nosso código vamos pegar o valor passado como argumento para o `--max` através do `${args[--max]}`. Primeiro vamos pegar o valor no `src/root_command.sh` e passá-lo para cada uma das funções, assim:

```bash
# src/root_command.sh

# observe que o argumento passado para o '--max'
# é obtido através do '${args[--max]}':
max_number="${args[--max]}"
# estamos salvando esse valor na variável 'max_number'
# pra referenciá-la facilmente abaixo...

if [[ "${args[--web]}" == 1 ]]; then
  get_random_number_from_web "$max_number"
else
  generate_random_number "$max_number"
fi
```

Agora precisamos adaptar as funções para lidar com esse argumento:

```bash
# src/lib/random_number_functions.sh

generate_random_number() {
  local max_number="$1"
  # nova lógica para gerar número respeitando o valor máximo:
  echo $((RANDOM % max_number + 1))
}

get_random_number_from_web() {
  local max_number="$1"
  curl \
    --silent \
    --location \
    "https://www.random.org/integers/?num=1&min=0&max=${max_number}&col=1&base=10&format=plain"
    # especifica valor máximo passado para random.org 👆
}
```

Vamos executar isso algumas vezes pra ver se funciona mesmo:

```
$ ./rndm --max 6
5

$ ./rndm --max 6
1

$ ./rndm --max 6
4

$ # pegando da web:
$ ./rndm --max 6 -w
6

$ ./rndm --max 6 -w
4

$ # agora sem especificar valor máximo:
$ ./rndm
./rndm: line 29: RANDOM % max_number + 1: division by 0 (error token is "max_number + 1")

$ ./rndm --web
Error: The maximum value must be an integer in the [-1000000000,1000000000] interval
```

😱 - O que?! Bugs detectados!

Se o usuário não especificar um valor para `--max`, o nosso script vai quebrar!

Vamos resolver esse problema da seguinte forma...

### Atribuindo um valor default para um argumento

Como vimos, nosso código está bugado! Acabamos tornando obrigatório que seja especificado um valor para `max_number`. Do contrário tanto a geração local quanto a requisição ao random.org irão quebrar.

Podemos resolver isso definindo um valor default. Aí a pergunta que surge é: qual valor utilizar como default?

Ali na mensagem de erro do `rndm --web` podemos ver que o máximo é 1.000.000.000 (um bilhão). No entanto a versão local do nosso gerador não é tão poderosa assim...

Na manpage do bash, se procurarmos por `RANDOM` na seção de "Shell Variables", veremos a informação de que `$RANDOM` gera um inteiro entre 0 e 32767. Então, por uma questão de consistência, vamos definir nosso valor default de `--max` como `32767`.

A notícia boa é que com o Bashly é muito simples definir um valor default para um argumento.

```yaml
name: rndm
help: Prints a random number
version: 0.0.1

dependencies:
  - curl

flags:
  - long: --web
    short: -w
    help: Get the random number from <https://random.org>.
  - long: --max
    arg: max
    help: Specifies the maximum number to be generated
	# Veja como é simples atribuir um valor default!
	# Obs.: as "aspas" são necessárias para que o valor
	#       seja considerado uma string, e não um número.
    default: "32767"
```

Uma conferida no help:

```
$ ./rndm --help
rndm - Prints a random number

Usage:
  rndm [OPTIONS]
  rndm --help | -h
  rndm --version | -v

Options:
  --web, -w
    Get the random number from <https://random.org>.

  --max MAX_NUMBER
    Specifies the maximum number to be generated
    Default: 32767

  --help, -h
    Show this help

  --version, -v
    Show version number
```

Legal! Ele deixa explícito para o usuário qual é o valor default! 👍

Agora vamos conferir se funciona mesmo:

```
$ # funciona mesmo sem especificar valor máximo:
$ ./rndm
8654

$ ./rndm
26564

$ ./rndm --web
9511

$ ./rndm --web --max 100
45

$ ./rndm --web --max 100
3

$ ./rndm --max 100
88
```

Aparentemente tudo OK. Mas vamos tentar dar uma avacalhada: 😈

```
$ ./rndm --max texto
./rndm: line 29: RANDOM % max_number + 1: division by 0 (error token is "max_number + 1")

$ ./rndm --max texto --web
Error: The maximum value must be an integer in the [-1000000000,1000000000] interval
```

😖 - Ouch!

Esse negócio de adicionar um valor máximo parecia ser simples, mas acabou trazendo um monte de bugs pro nosso programa! 😓

### Validando argumentos

Pra corrigir esse novo bug que aparece quando passamos valores inválidos para `--max` vamos precisar adicionar uma lógica de validação de input. Essa validação precisa simplesmente garantir que o valor é um número inteiro positivo.

Vamos resolver isso com essa expressão regular: `^[1-9][0-9]*$`. Que significa "um dígito entre 1 e 9 seguido de qualquer quantidade de dígitos entre 0 e 9".

Usando isso no nosso código, vai ficar assim:

```bash
# src/root_command.sh

max_number="${args[--max]}"

# aborta execução se o max_number não for um inteiro positivo
if ! [[ "$max_number" =~ ^[1-9][0-9]*$ ]]; then
  echo "The argument must be a positive integer. Given value: $max_number"
  exit 1
fi

if [[ "${args[--web]}" == 1 ]]; then
  get_random_number_from_web "$max_number"
else
  generate_random_number "$max_number"
fi
``` 

Conferindo:

```
$ ./rndm --max texto
The argument must be a positive integer. Given value: texto

$ ./rndm --max -1
The argument must be a positive integer. Given value: -1

$ ./rndm
26509
```

OK, parece que deu certo. Mas eu não estou gostando dessa lógica de validação poluindo meu código principal.

Vamos mover essa validação para um outro arquivo. Vamos criar o `src/lib/validations.sh` com o seguinte conteúdo:

```bash
# src/lib/validations.sh

# criando uma função específica de validação:
validate_positive_integer() {
  local number="$1"

  if ! is_positive_integer "$number"; then
    echo "The argument must be a positive integer. Given value: $number"
    exit 1
  fi
}

# regra pessoal:
# se vai fazer algo com expressões regulares, dê um jeito de
# nomear o que está fazendo! Nesse caso eu apenas criei uma
# função com um nome claro.
is_positive_integer() {
  [[ "$1" =~ ^[1-9][0-9]*$ ]]
}
```

Agora lá no nosso `src/root_command.sh` podemos chamar a nossa validação, assim:

```bash
# src/root_command.sh

max_number="${args[--max]}"

# 👇 simplesmente chamando a validação aqui!
validate_positive_integer "$max_number"

if [[ "${args[--web]}" == 1 ]]; then
  get_random_number_from_web "$max_number"
else
  generate_random_number "$max_number"
fi
```

Após fazer essas alterações, rode um `bashly generate` novamente e confira que as validações continuam funcionando.

```
$ ./rndm --max texto
The argument must be a positive integer. Given value: texto

$ ./rndm --max -1
The argument must be a positive integer. Given value: -1
```

Agora é uma boa hora para mais um commit.

### Validando argumentos da maneira Bashly

Apesar de já estarmos validando o argumento de `--max` chamando a função `validate_positive_integer` la dentro do `src/root_command.sh`, o Bashly oferece uma maneira ainda mais limpa de fazermos essa validação. De forma que podemos remover essas referências a validações do nosso código principal e deixá-lo bem limpinho e focado na geração de números aleatórios.

A maneira Bashly de fazer validação funciona da seguinte forma:

- Na configuração da flag, adicionamos uma linha assim:`validate: function_name`.
- Criamos uma função chamada `validate_function_name`, que será automaticamente executada antes de permitir que o input do usuário seja usado.
- Se essa função imprimir qualquer coisa em stdout, isso será considerado um erro. O conteúdo será exibido na tela, como mensagem de erro e o programa irá abortar com falha.

Pois vamos aplicar isso no nosso programa.

**Passo 1**: adicionar `validate: positive_integer` na configuração da flag.

```yaml
name: rndm
# ...

flags:
  # ...
  - long: --max
    # ...
    # 👇👇👇 apenas adicionamos essa linha
    validate: positive_integer
```

**Passo 2**: criar uma função chamada `validate_positive_integer`.

Já fizemos isso na seção anterior. Essa função está salva em `src/lib/validations.sh`.

**Passo 3**: função precisa imprimir algo em stdout para ser considerada um erro.

Nossa função já faz isso. A única coisa que iremos mudar aqui é que não precisamos mas de um `exit 1` explícito, pois isso será gerido pelo Bashly quando for detectado que algo foi enviado para stdout. Portanto a versão final da função fica assim:

```bash
# src/lib/validations.sh

validate_positive_integer() {
  local number="$1"
  if ! is_positive_integer "$number"; then
    echo "The argument must be a positive integer. Given value: $number"
  fi
}

# ...
```

Pronto! Agora vamos testar se isso dá certo mesmo:

```
$ ./rndm
26086

$ ./rndm --max 0
validation error in --max MAX_NUMBER:
The argument must be a positive integer. Given value: 0

$ ./rndm --max texto
validation error in --max MAX_NUMBER:
The argument must be a positive integer. Given value: texto

$ ./rndm --max -1
validation error in --max MAX_NUMBER:
The argument must be a positive integer. Given value: -1
```

Olha que bacana: o Bashly até melhorou a mensagem de erro, explicitando que é um problema na validação do `--max`!

OK, fizemos essa validação _a-la-Bashly_, mas o nosso `src/root_command.sh` continua com referências (desnecessárias) às funções de validação. Vamos limpar nosso código e você verá como que o Bashly permite que nosso código fique bem limpinho:

```bash
# src/root_command.sh

max_number="${args[--max]}"

if [[ "${args[--web]}" == 1 ]]; then
  get_random_number_from_web "$max_number"
else
  generate_random_number "$max_number"
fi
```

Faça novos testes e constate que tudo está funcionando corretamente. OK, pode ser que você encontre mais edge-cases para serem endereçados. Mas para uma demonstração de como trabalhar com o Bashly já vamos nos dar por satisfeitos. 😇

Vamos fazer mais um commit e ir encerrando essa primeira parte do tutorial.

## Finalizando (por enquanto)

Agora eu gostaria que você parasse por um momento e desse mais uma olhada no seu `src/root_command.sh`. Aprecie o quanto o código é simples.

Vamos também dar uma apreciada na estrutura de diretórios e arquivos do nosso projeto:

```
$ tree
.
├── rndm
└── src
    ├── bashly.yml
    ├── lib
    │   ├── random_number_functions.sh
    │   └── validations.sh
    └── root_command.sh

2 directories, 5 files
```

De código que nós realmente escrevemos, só temos estes 3 arquivinhos com propósitos muito bem definidos:

- `validations.sh`: responsável pela validação de input.
- `random_number_functions.sh`: responsável por conter as funções geradoras de números aleatórios.
- `root_command.sh`: a "porta de entrada" da aplicação, que chama a função correta de acordo com a opção do usuário.

Agora vamos lembrar das minúcias e complexidades que nós nem nos preocupamos pois o Bashly resolveu pra nós:

- mensagem de help lindona e completinha
- verificação de dependências
- parsing de `--opções`
- chamar a validação de input correta
- modularização de código

E isso é apenas uma breve introdução ao Bashly. Se você gostaria que eu escrevesse mais sobre este tema, deixe aí nos comentários. O Bashly tem muitos outros recursos interessantes que valem a pena serem explorados.

## Principais Aprendizados

#### O comando `bashly`

- Bashly é uma gem Ruby que depende do Ruby 3.2+.
    - não é necessário conhecimento de Ruby para usar Bashly
- o comando `bashly generate` lê o arquivo `src/bashly.yml` e gera o script final
    - use `bashly generate --watch` para ficar monitorando alterações e gerar automaticamente.

#### Arquivos de código

- o arquivo `src/bashly.yml` é um YAML que é como se fosse um contrato da interface entre nossa aplicação e o usuário.
    - podemos especificar dependências definindo `dependencies:`.
- o arquivo `src/root_command.sh` é a "porta de entrada" da aplicação.
- arquivos `src/lib/*.sh` são todos colocados no script final.

#### Argumentos da linha de comando

- flags
    - valores ficam em `${args[--nome-da-flag]}`
    - flags booleanas possuem valor `1` quando utilizadas
- validação de argumento:
    - na configuração da flag:`validate: function_name`
    - função `validate_function_name` será executada antes de usar o input do usuário.
    - se `validate_function_name` imprimir qualquer coisa em stdout, programa aborta.

## Referências

[Documentação do bashly.](https://bashly.dev)

