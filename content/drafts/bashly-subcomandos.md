
No [artigo anterior](/bashly) demos os nossos primeiros passos com o Bashly. Criamos um gerador de números aleatórios bem simples, porém com uma interface sólida. 

Com pouco esforço conseguimos:

- adicionar opções de linha de comando
- validação de input
- checagem de dependências
- uma mensagem de help bem "profissional"

## O que veremos?

Vamos assumir que o papo de aleatoriedade do último nos deu a ideia de criar um gerador de senhas aleatórias. Ao invés de criar uma nova aplicação "do zero" vamos aproveitar as funções que já temos nosso `rndm`, que gera números aleatórios, e apenas adicionar a funcionalidade de gerar senhas.

Vamos continuar nosso projeto e explorar mais algumas _features_ do Bashly. Veremos como ele pode continuar facilitando nossa vida ao criar aplicações CLI.

Primeiro aprenderemos a criar subcomandos. Isso vai permitir que nosso `rndm` possa ser utilizado de duas formas:
- `rndm number`: gera número aleatório
- `rndm password`: gera um password aleatório

Em seguida vamos implementar nosso gerador de password e ir adicionando _features_ a ele.


## Subcomandos

A ideia de subcomandos é muito comum em aplicações CLI modernas como Git (`git add`, `git commit`, `git push`) e Docker (`docker image pull`, `docker container run`). Pois é isso que faremos com nosso programa.

O Bashly permite a criação de subcomandos  de forma bem simples, basta usarmos uma estrutura como essa no `src/bashly.yml`:

```yaml
name: my_cli
help: description...

# basta colocar os subcomandos aqui
# dentro de "commands:"
commands:
  # defina o nome do seu subcomando:
  - name: my_subcommand
    # e aqui vem as mesmíssimas configs usadas
    # para comandos "simples" (root_command)
    # exemplo:
    flags:
      - long: --my-long-option
      # ...
```

Ou seja, basta definirmos um `commands:` e colocar dentro dele os nossos subcomandos.

Com um exemplo fica mais tranquilo de entender...

Vamos transformar nosso gerador de números aleatórios em um subcomando que será invocado assim: `rndm number`.

### `rndm number`

Só pra lembrar, atualmente nosso `src/bashly.yml` está assim:

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
    arg: max_num
    help: Specifies the maximum number to be generated
    default: "32767"
    validate: positive_integer
```

Para transformar nosso comando atual em um subcomando basta passarmos as atuais configurações do gerador de números para dentro de um item de `commands:`. Vamos fazer isso agora:

```yaml
name: rndm
# 👇 Vamos atualizar a descrição da aplicação
help: Do random stuff
# 👇 É uma boa darmos um bump na versão também
version: 0.0.2

#################################
# essas 3 linhas foram as únicas
# linhas adicionadas ao YAML
# 👇
commands:
  - name: number
    help: Prints a random number
# 👆
#################################

    # daqui pra baixo apenas ajustamos a indentação!
    # o conteúdo é o mesmo.
    dependencies:
      - curl

    flags:
      - long: --web
        short: -w
        help: Get the random number from <https://random.org>.

      - long: --max
        arg: max_num
        help: Specifies the maximum number to be generated
        default: "32767"
        validate: positive_integer
```

Vamos executar `bashly generate`:

```
$ bashly generate
creating user files in src
created src/number_command.sh
created ./rndm
run ./rndm --help to test your bash script
```

Observe que o arquivo `src/number_command.sh` foi criado.

Uma outra coisa que não está explicita ali mas que precisamos levar em consideração, é que o nosso `src/root_command.sh` original foi completamente ignorado. Isso ocorre pois agora o nosso `src/bashly.yml` não especifica nenhum "root command". Temos apenas um subcomando chamado "number" (e é por isso que o Bashly criou um `src/number_command.sh`).

Se você olhar o arquivo gerado, verá um conteúdo já familiar (vimos isso no artigo anterior):

```bash
echo "# This file is located at 'src/number_command.sh'."
echo "# It contains the implementation for the 'rndm number' command."
echo "# The code you write here will be wrapped by a function named 'rndm_number_command()'."
echo "# Feel free to edit this file; your changes will persist when regenerating."
inspect_args
```

Nós não precisamos de nada disso. A única que precisamos é simplesmente mover todo o conteúdo do nosso código original para o novo arquivo:

```bash
mv src/root_command.sh src/number_command.sh
```

Agora basta um `bashly generate` e testar o programa usando o subcomando:

```
$ # se usarmos sem argumento, temos a mensgem de "usage"
$ ./rndm
rndm - Do random stuff

Usage:
  rndm COMMAND
  rndm [COMMAND] --help | -h
  rndm --version | -v

Commands:
  number   Prints a random number

$ # chamando via subcomando
$ ./rndm number
13790

$ ./rndm number
2884

$ ./rndm number -w
11598

$ ./rndm number -w --max 10
4
```

OK, parece está tudo funcionando conforme esperado. 👍

> **Lembre-se**: para regenerar o script automaticamente a cada alteração de arquivo, abra um novo terminal e execute:
> ```bash
> bashly generate --watch
> ```

Faça um commit e vamos explorar outros recursos relacionados à subcomandos.

### Fazendo um subcomando ser o padrão

Originalmente nosso gerador de números aleatórios era executado invocando `rndm`. Agora tornamos obrigatório que ele seja invocado via `rndm number`. Com isso quebramos a retrocompatibilidade do nosso programa.

Se por um acaso algum de nossos usuários está chamando nosso `rndm` em algum script dele, terá uma surpresa bem desagradável quando atualizar nosso programa e ver que o script dele está quebrando (por nossa causa).

Para evitar essa situação, vamos fazer com que o `rndm number` seja o subcomando _default_ a ser invocado quando chamarmos simplesmente `rndm`. Para isso basta adicionarmos `default: force` no nosso `src/bashly.yml`:

```yaml
name: rndm
help: Prints a random number
version: 0.0.2

commands:
  - name: number
    # 👇 única linha adicionada.
    default: force
    # daqui pra baixo tudo igual...
```

Agora pode testar sem passar o subcomando que você verá que voltamos a disponibilizar o nosso gerador de números aleatórios via `rndm`:

```
$ ./rndm --max 3
2

$ ./rndm
10157

$ ./rndm --web
4607
```

OK. Apenas uma única linha no nosso YAML e o problema foi resolvido.

Faça um commit e vamos continuar.

### Criando alias para o subcomando

Para facilitar a vida dos nossos usuários, vamos permitir que eles chamem nosso gerador via `rndm num` e também simplesmente via `rndm n`:

Para isso basta deixarmos claro no nosso YAML que queremos criar aliases:

```yaml
# src/bashly.yml
name: rndm
help: Prints a random number
version: 0.0.2

commands:
  - name: number
    default: force
    
    # basta adicionar um array de aliases
    alias:
      - num
      - n
      
    # daqui pra baixo tudo igual...
```

Confirmando que funciona:

```
$ ./rndm num
10157

$ ./rndm num --max 3
2

$ ./rndm n
26071

$ ./rndm n --web
4607
```

Bacaninha, né?

Agora chega de futucar o gerador de números aleatórios. Faça mais um commit e vamos partir para o gerador de senhas.

## Gerador de Senhas

Antes de partir pro código, vamos declarar no nosso YAML que queremos adicionar um subcomando:

```yaml
name: rndm
help: Do random stuff
version: 0.0.2

commands:
  - name: number
    # ... configurações do 'rndm number'

  # 👇 apenas o nome e uma descrição pra mostrar no help
  - name: password
    help: Generates a random password
```

Vamos dar uma olhadela no help como ficou:

```
$ ./rndm --help
rndm - Do random stuff

Usage:
  rndm COMMAND
  rndm [COMMAND] --help | -h
  rndm --version | -v

Commands:
  number     Prints a random number
  password   Generates a random password

Options:
  --help, -h
    Show this help

  --version, -v
    Show version number
```

Conforme esperado, o subcomando `password` está listado ali. Seguimos...

Ao regenerar observamos que o arquivo `src/password_command.sh` foi criado. É nele que colocaremos nosso código.

Explicando sucintamente como funcionará nosso gerador de senhas:

- teremos um sequência de caracteres a serem utilizados na senha, por exemplo: `abcdefghijklmnopqrstuvwxyz`.
- nesse exemplo temos 26 caracteres, portanto geramos um número aleatório entre 1 e 26 e pegamos o respectivo carácter da lista.
- repetimos esse processo até termos uma senha do tamanho desejado

A lista de caracteres usadas nesse exemplo foi só pra facilitar a explicação. O que nós queremos de verdade é um gerador de senhas que use letras minúsculas, maiúsculas e números (e posteriormente adicionaremos caracteres especiais).

Nessa primeira implementação vamos definir um tamanho de senha de 8 caracteres.

```bash
# src/password_command.sh

size=8
letters='abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'
numbers='1234567890'
chars="${letters}${numbers}"
password=''

while [[ ${#password} -lt $size ]]; do
  offset="$(generate_random_number "${#chars}")"
  password+="${chars:offset-1:1}"
done

echo "$password"
```

O código acima faz uso de alguns recursos do Bash que podem não ser tão amplamente conhecidos, então vamos relembrar:

- a notação `${#var}` é como o bash retorna o tamanho de uma string
- a notação `${var:N:1}` significa que queremos uma substring de `$var`, começando do enésimo carácter, e contendo apenas 1 carácter.

A função `generate_random_number` é exatamente aquela que criamos no artigo anterior, um gerador de números aleatórios que recebe como primeiro argumento o valor máximo.

Após um `bashly generate` vamos executar o script algumas vezes:

```
$ ./rndm password
riK8pYVZ

$ ./rndm password
sf6CZx91

$ ./rndm password
z9Oc2FX7

$ ./rndm password
G4iqVrWg
```

Legal! Funcionando conforme esperado!

### Movendo geração de senha para função

Agora eu já estou querendo que a lógica de geração de senha vá pra uma função específica. Portanto vamos fazer isso criando o arquivo `src/lib/password_functions.sh`:

```bash
# src/lib/password_functions.sh

generate_password() {
  local charset="$1"
  local size="$2"
  local password offset

  while [[ ${#password} -lt $size ]]; do
    offset="$(generate_random_number "${#chars}")"
    password+="${chars:offset-1:1}"
  done

  echo "$password"
}
```

Agora lá no nosso `src/password_command.sh` podemos penas chamar a função, assim:

```bash
# src/password_command.sh

size=8
letters='abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'
numbers='1234567890'
chars="${letters}${numbers}"

generate_password "$chars" "$size"
```
 
 Execute os testes aí do seu passo e confirme que tudo está funcionando. 
 
### Aliases para o `rndm password`

Estou achando que esse `rndm password` é um comando muito grande pra digitar. Vamos criar uns aliases:

```yaml
name: rndm
help: Do random stuff
version: 0.0.2

commands:
  - name: number
    # ... configurações do 'rndm number'

  - name: password
    help: Generates a random password
    alias:
      - passwd
      - pass
      - p
```

Testando:

```
$ ./rndm passwd
Fsgo1w4q

$ ./rndm pass
iIe2nj6Z

$ ./rndm p
aZSF8a8Z
```

Ótimo! Agora estou sentindo que é uma boa hora pra um commit e partirmos pra uma nova _feature_.

### Tamanho da senha

Vamos adicionar e configurar uma flag para que o usuário possa especificar o tamanho da senha a ser gerada:

```yaml
name: rndm
help: Do random stuff
version: 0.0.2

commands:
  - name: number
    # ... configurações do 'rndm number'

  - name: password
    help: Generates a random password
    alias:
      - passwd
      - pass
      - p

    # início da declaração de flags
    flags:
      # 👇 configuração da flag --size
      - long: --size
        short: -s
        arg: password_size
        help: Number of characters in the generated password
        default: "8"
        validate: positive_integer
```

O que fizemos aqui já aprendemos no artigo anterior, então nem vou me preocupar com muitas explicações.

Só com essas adições ao YAML já temos alguns benefícios que já podemos perceber:

1. Mensagem de help já mostra info sobre a nova opção 

```
rndm password - Generates a random password

Alias: passwd, pass, p

Usage:
  rndm password [OPTIONS]
  rndm password --help | -h

Options:
  --size, -s PASSWORD_SIZE
    Number of characters in the generated password
    Default: 8

  --help, -h
    Show this help
```

2. Já estamos fazendo validação de input (reaproveitando código que criamos anteriormente)

```
$ ./rndm p --size 0
validation error in --size, -s PASSWORD_SIZE:
The argument must be a positive integer. Given value: 0

$ ./rndm p --size -1
validation error in --size, -s PASSWORD_SIZE:
The argument must be a positive integer. Given value: -1

$ ./rndm p --size texto
validation error in --size, -s PASSWORD_SIZE:
The argument must be a positive integer. Given value: texto
```

3. Temos a variável `${args[--size]}` à nossa disposição.

Para fazer com que nosso código respeite a decisão do usuário referente ao tamanho da senha, basta pegar o valor passado como argumento e atribuir à variável `size`:

```bash
# src/password_command.sh

# 👇 única mudança
size="${args[--size]}"
# 👆

letters='abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'
numbers='1234567890'
chars="${letters}${numbers}"

generate_password "$chars" "$size"
```

Conferindo:

```
$ ./rndm pass --size 10
u37A7h8VUp

$ ./rndm pass --size 20
GgwdfpVMXvP770mA0TV3

$ ./rndm pass --size 30
XvUytaiCfYR5CM9l1HC0AZDkUvfU3s

$ ./rndm passwd
0yVGYWpx
```

Exatamente o que queremos!

Mais um commit e vamos para a próxima _feature_.

### Senhas numéricas

Pode ser que nosso usuário queira uma senha numérica, portanto vamos disponibilizar isso pra ele através da flag `--numeric`:

```yaml
name: rndm
help: Do random stuff
version: 0.0.2

commands:
  # ...
  - name: password
    # ...
    flags:
      # ...
      # 👇 linhas adicionadas
      - long: --numeric
        short: -n
        help: Generates a numeric password
```

Com essa configuração nós teremos a flag `${args[--numeric]}` disponível no nosso código, e nós vamos usá-la assim:

```bash
# src/password_command.sh

size="${args[--size]}"
numbers='1234567890'
letters='abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'

# 👇 verificando se usuário quer senha numérica
if [[ "${args[--numeric]}" ]]; then
  chars="${numbers}"
else
  chars="${letters}${numbers}"
fi
# 👆

generate_password "$chars" "$size"
```

Vamos dar uma conferida:

```
$ ./rndm password --numeric
61749459

$ ./rndm pass --numeric --size 6
321746

$ ./rndm pass -n
46261722

$ ./rndm pass -n -s 4
4397

$ ./rndm pass -ns 4
5613
```

Mais uma feature implementada. Vamos commitar e partir pra próxima!

### Senhas com apenas letras

Pode ser que o usuário também queira gerar uma senha apenas com letras, sem números. Vamos prover essa opção via `--alpha`.

```yaml
# ...
commands:
  # ...
  - name: password
    # ...
    flags:
      # ...
      # 👇 linhas adicionadas
      - long: --alpha
        short: -a
        help: Generates a password using only letters from the alphabet
```

Agora precisamos lidar com o `--alpha` no nosso código:

```bash
# src/password_command.sh

size="${args[--size]}"
numbers='1234567890'
letters='abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'

if [[ "${args[--numeric]}" ]]; then
  chars="${numbers}"

# 👇 se usuário passar '--alpha'...
elif [[ "${args['--alpha']}" ]]; then
  chars="${letters}"
  # 👆 vamos usar apenas letras do alfabeto
  
else
  chars="${letters}${numbers}"
fi

generate_password "$chars" "$size"
```

Testando:

```
$ ./rndm pass --alpha
nCtrNHgV

$ ./rndm pass -a
fGGrYHfi

$ ./rndm pass -a --size 10
WNXjLrLakm

$ ./rndm pass -as 10
iAGAdHekyO

$ ./rndm pass -as 20
lxafckXNyRetYnLIkApp

$ # o que será que acontece se usarmos isso:
$ ./rndm pass --alpha --numeric
85366290
```

Tava tudo funcionando legal, até que ali no último exemplo encontramos um problema para resolver: a opção `--alpha` não deveria ser permitida quando usamos `--numeric`. Ou seja, as opções `--numeric` e `--alpha` devem ser mutuamente exclusivas, e o nosso programa precisa avisar ao usuário quando ele comete este equívoco.

### Argumentos mutuamente exclusivos

Felizmente o Bashly nos fornece uma maneira muito fácil de especificar que argumentos são conflitantes. Basta declararmos isso usando `conflicts`, com um detalhe importante: **a configuração de `conflicts` precisa ser declarada nos dois dois lados da exclusividade**.

Aqui faremos isso:

```yaml
# ...
commands:
  # ...
  - name: password
    # ...
    flags:
      # ...
      - long: --numeric
        short: -n
        help: Generates a numeric password
        # 👇 linhas adicionadas
        conflicts:
          - --alpha
        # 👆 linhas adicionadas

      - long: --alpha
        short: -a
        help: Generates a password using only letters from the alphabet
        # 👇 linhas adicionadas
        conflicts:
          - --numeric
        # 👆 linhas adicionadas
```

Dessa vez não precisamos fazer coisa alguma com nosso código. Tudo será lindamente resolvido pelo Bashly.

Vamos ver se isso realmente funciona:

```
$ ./rndm pass --alpha --numeric
conflicting options: --numeric cannot be used with --alpha

$ ./rndm pass --numeric --alpha
conflicting options: --alpha cannot be used with --numeric

$ ./rndm pass -na
conflicting options: -a cannot be used with --numeric

$ ./rndm pass -an
conflicting options: -n cannot be used with --alpha
```

Perfeito! Vamos commitar e ver um outro caso de uso.
### Senhas com caracteres especiais

E se o nosso usuário quiser uma senha bem forte, incluindo caracteres especiais?

Vamos prover essa funcionalidade através da opção `--allow-symbols`. 

```yaml
name: rndm
help: Do random stuff
version: 0.0.2

commands:
  # ...
  - name: password
    # ...
    flags:
      # ...
      # 👇 linhas adicionadas
      - long: --allow-symbols
        short: -S
        help: Allow special characters in the generated password
```

Agora no nosso código vamos adicionar a lista de símbolos à lista de possíveis caracteres:

```bash
# src/password_command.sh

size="${args[--size]}"
numbers='1234567890'
letters='abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'

# 👇 lista de caracteres especiais
symbols='!@#$%&*()-_+={}[];:/?,.'

if [[ "${args[--numeric]}" ]]; then
  chars="${numbers}"
elif [[ "${args['--alpha']}" ]]; then
  chars="${letters}"
else
  chars="${letters}${numbers}"
fi

# 👇 se usuário passar '--allow-symbols'...
if [[ "${args['--allow-symbols']}" ]]; then
  chars+="${symbols}"
fi
# 👆 adicionamos os caracteres especiais na lista

generate_password "$chars" "$size"
```

Agora vamos testar várias maneiras de chamar o `--allow-symbols`:

```
$ ./rndm pass --allow-symbols
2E%3}6&_

$ ./rndm pass -S
E0OFLv@F

$ ./rndm pass -S --size 10
lbr#!udh?)

$ ./rndm pass -S --size 15
uLIT4SFPI]a/C$J

$ ./rndm pass -S -s 20
-0yL$,I5x%}lYffH_/@7

$ ./rndm pass -Ss 20
MvVS&!H8rCvIJEe;(Zjp

$ # se quiser uma senha com números e símbolos:
$ ./rndm pass --numeric --allow-symbols
1}!5,@]2
```

Acho que ficou bem legal nosso gerador de senhas. Vamos fazer mais um commit e ir finalizando o artigo.

## Finalizando

Neste artigo eu espero que uma coisa tenha ficado bem evidente: focamos mais nas _features_ da nossa aplicação do que em qualquer outra coisa.

Todo o rolê de lidar com subcomandos, fazer parsing de argumentos, detectar argumentos conflitantes... Toda essa complexidade colateral foi resolvido com algumas poucas linhas no nosso YAML. Essa é a beleza do Bashly! Ele te diz: "vai lá focar nas features que você quer entregar pro seu usuário, deixa que eu cuido do trabalho chato".

Só esse help lindão já é uma grande demonstração de algo que seria extremamente maçante e propenso a erros e esquecimentos, mas que é resolvido "de graça" pelo Bashly:

```
$ ./rndm password --help
rndm password - Generates a random password

Alias: passwd, pass, p

Usage:
  rndm password [OPTIONS]
  rndm password --help | -h

Options:
  --size, -s PASSWORD_SIZE
    Number of characters in the generated password
    Default: 8

  --numeric, -n
    Generates a numeric password
    Conflicts: --alpha

  --alpha, -a
    Generates a password using only letters from the alphabet
    Conflicts: --numeric

  --allow-symbols, -S
    Allow special characters in the generated password

  --help, -h
    Show this help
``` 

Vamos dar uma olhada também na estrutura do nosso projeto:

```
$ tree
.
├── rndm
└── src
    ├── bashly.yml
    ├── lib
    │   ├── password_functions.sh
    │   ├── random_number_functions.sh
    │   └── validations.sh
    ├── number_command.sh
    └── password_command.sh

2 directories, 7 files
```

Percebe-se que é um projetinho simples, porém com um acabamento e uma interface bem sólida.

## Principais aprendizados

- criar subcomandos
- definir um subcomando como o comando padrão
- criar aliases
- declarar argumentos mutuamente exclusivos (`conflict`)

## Referências

[Documentação do Bashly](https://bashly.dev/).