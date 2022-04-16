---
title: "Como detectar precisamente onde seu script está quebrando"
description: >
  Segredinhos obscuros do bash que permitirão que você poupe muito tempo quando precisar caçar bugs. Garanto que isso vai mudar sua vida.
tags:
  - trap
  - boas-praticas
date: 2022-04-15T07:09:29-03:00
cover:
  image: "img/trap-err.png"
  alt: capturando erros no bash
---

Esse artigo é uma continuação do artigo anterior sobre como deixar o [bash mais rigoroso](/bash-rigoroso).

No artigo anterior aprendemos como fazer o nosso script falhar o mais rápido possível e entedemos qual é a grande vantagem disso. Neste artigo veremos como obter uma indicação bem direta e precisa de onde o nosso script falhou.

Desde que comecei a usar isso nos meus scripts (principalmente nas minhas esteiras de Integração Contínua) minha vida é muito mais feliz. Isso mudou minha vida e espero que mude a sua também! 🙏


## Recapitulando...

Só pra lembrar, no artigo anterior entendemos que devemos iniciar nossos scripts com:

```bash
set -euo pipefail
```

Pois queremos que o script:

- `set -e`: seja interrompido assim que ele falhar
- `set -u`: não tolere variáveis sem um valor explicitamente definido
- `set -o pipefail`: o "exit status" de uma pipeline seja o status do primeiro comando que falhar (ou sucesso)

A partir desse conhecimento vamos avançar um pouco mais...


## Não quero ler tudo isso! Me diz logo o que tenho que fazer!

A partir de agora use isso no início de todos os seus scripts:
```bash
set -Eeuo pipefail

trap 'echo "${BASH_SOURCE}:${LINENO}:${FUNCNAME:-}"' ERR
```

O tempo que você vai economizar com isso você pode usar para ler os artigos deste site. 😇

## Variáveis úteis definidas pelo bash

O `bash` por padrão já define muitas variáveis com utilidades específicas. Pro nosso propósito aqui iremos utilizar as seguintes:

- `BASH_SOURCE`: o nome do arquivo onde está o seu script (na verdade esta variável é um array, mas aqui nós vamos abstrair isso)
- `LINENO`: linha exata aonde esta variável está sendo referenciada.
- `FUNCNAME`: nome da função onde esta variável está sendo referenciada.

Vamos a um script ilustrativo pra explicar melhor:
```bash
#!/usr/bin/env bash
# scriptinfo.sh

# falhe rápido:
# https://meleu.sh/bash-rigoroso
set -euo pipefail

echo "--> informações do script <--"
echo "BASH_SOURCE='${BASH_SOURCE}'"
echo "LINENO='${LINENO}'"
echo "FUNCNAME='${FUNCNAME}'"
echo 
echo "fim!"
```

Executando:
```txt
$ ./scriptinfo.sh 
--> informações do script <--
BASH_SOURCE='./scriptinfo.sh'
LINENO='10'
./scriptinfo.sh: line 11: FUNCNAME: unbound variable
```

Oops!! 😳

OK, entendemos que `BASH_SOURCE` trouxe o nome do script, e que o `LINENO` trouxe o número exato da linha do script onde ele foi referenciado. Mas e aquele erro ali na linha 11?

Esse erro aconteceu porque lá no começo do script dissemos ao bash que queremos que ele quebre sempre que encontrar uma variável sem um valor definido (`set -u`, aprendemos essa técnica no [artigo anterior](/bash-rigoroso/#não-permita-variáveis-não-declaradas)). E como estamos chamando a variável `FUNCNAME` fora de uma função, ela está vazia.

Para contornar isso atribuindo uma string vazia como valor default mas também vamos adicionar uma função ao script só pra ver a variável funcionando de verdade:
```bash
#!/usr/bin/env bash
# script2.info

# falhe rápido:
# https://meleu.sh/bash-rigoroso
set -euo pipefail

echo "--> informações de fora da função <--"
echo "BASH_SOURCE='${BASH_SOURCE}'"
echo "LINENO='${LINENO}'"
echo "FUNCNAME='${FUNCNAME:-}'"
echo

main() {
  echo "--> informações de dentro da função <--"
  echo "BASH_SOURCE='${BASH_SOURCE}'"
  echo "LINENO='${LINENO}'"
  echo "FUNCNAME='${FUNCNAME:-}'"
  echo
  echo "fim!"
}

main "$@"
```

Executando:
```txt
$ ./scriptinfo2.sh 
--> informações de fora da função <--
BASH_SOURCE='./scriptinfo2.sh'
LINENO='10'
FUNCNAME=''

--> informações de dentro da função <--
BASH_SOURCE='./scriptinfo2.sh'
LINENO='17'
FUNCNAME='main'

fim!
```

Bacana, não é mesmo?! 🤓

Agora que entendemos a utilidade dessas variáveis, vamos dar uma pausa pra falar do trap...


## O comando `trap`

O `trap` serve para "capturar" um sinal que o bash acabou de receber e executar algum comando quando esse sinal for capturado.

A sintaxe dele é assim:
```bash
trap COMANDO SINAL
```

Não está no escopo desse artigo entrar no detalhe de como *signal handling* funciona, portanto vamos a uma breve explicação com exemplos.

### Lista de sinais

Primeiro vamos ver a lista de sinais com o comando `trap -l`:
```txt
$ trap -l
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP
 6) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1
11) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM
16) SIGSTKFLT   17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP
21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU     25) SIGXFSZ
26) SIGVTALRM   27) SIGPROF     28) SIGWINCH    29) SIGIO       30) SIGPWR
31) SIGSYS      34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX
```

Essa lista nos mostra o número do sinal e o seu nome. Ao referenciarmos estes sinais no `trap` podemos usar o número, o nome, ou o nome sem o prefixo `SIG`. Exemplo: `2`, `SIGINT` e `INT` tem o mesmo significado.

### Capturando o Ctrl-c

Quando você tecla `Ctrl-c`, por exemplo, pra cancelar a execução de um script, o bash recebe o sinal `SIGINT`.

Vejamos o seguinte script:

```bash
#!/usr/bin/env bash
# ctrlc.sh

# o trap vai "capturar" o SIGINT, e ao invés de
# interromper o script, vai executar um comando
trap 'echo "Você pressionou <Ctrl-c>!"' INT

echo "🥱 - Que soninho... Vou dormir por 10 minutos."
echo "Se precisar de mim, pressione <Ctrl-c>"
echo "😴"

sleep 600

echo "🥴 - Acordei!"
```

Executando:
```txt
$ ./ctrlc.sh 
🥱 - Que soninho... Vou dormir por 10 minutos.
Se precisar de mim, pressione <Ctrl-c>
😴
^CVocê pressionou <Ctrl-c>!
🥴 - Acordei!
```

Observe que como o `trap` capturou o `SIGINT` gerado pelo `Ctrl-c`, executou um `echo` e o script prosseguiu. Ou seja, o `Ctrl-c` não interrompeu a execução do script.

### Ponto de atenção

Lembre-se da sintaxe do trap:
```bash
trap COMANDO SINAL
```

O `COMANDO` pode ser qualquer comando válido, mas ele **precisa** estar inteiro no primeiro argumento do trap.

Observe com atenção a linha do `trap` no script de exemplo:
```bash
trap 'echo "Você pressionou <Ctrl-c>!"' SIGINT
```

Veja como que o `echo` e todos os seus argumentos estão dentro de `'aspas simples'`.

Como sei que esse papo de aspas é um assunto meio tortuoso pra quem não usa o shell com frequência, fica aqui o alerta.


## Os Segredos™

Estou classificando o conhecimento que estou mostrando aqui como "segredo", não porque eles são realmente secretos. Mas porque eles estão meio que espalhados pela documentação. E quando eu consegui "ligar os pontos" o sentimento de [epifania](https://pt.wikipedia.org/wiki/Epifania) foi grande!


### Segredo #1: `set -e` cria um novo sinal

Essa informação meio que passa despercebida lá no help do trap (aqui traduzida por mim e mostrando apenas a parte que nos interessa):

```txt
$ help trap
trap: trap [-lp] [[arg] signal_spec ...]

   (...)
   Um SIGNAL_SPEC de ERR significa que é pra executar
   ARG cada vez que uma falha de um comando fizer
   o shell sair quando a opção -e está ativa.
   (...)
```

Pois é... Está escrito de maneira meio confusa. E não é por conta da minha tradução ser precária. Está escrito de maneira esquisita no original também...

O help está mencionando um sinal chamado `ERR`, mas se olharmos com atenção a lista de sinais no output do `trap -l` não tem nenhum `SIGERR`.

Pois é! O lance é que `set -e` faz o bash criar um sinal chamado `ERR` que será lançado quando o script encontrar algum comando que termine com um status diferente de zero.

Ou seja, o `set -e` faz o bash interromper o script assim encontra uma falha e em seguida lança o sinal `ERR`.

Exemplo bobo:
```bash
#!/usr/bin/env bash

set -euo pipefail

trap 'echo "Oops! Quebrei!"' ERR

comando invalido

echo "o script vai quebrar no comando inválido acima"
echo "portanto isso aqui não será executado"
```

Executando:
```txt
$ ./trap-bobo.sh 
./trap-bobo.sh: line 7: comando: command not found
Oops! Quebrei!
```

Como eu disse, o `set -e` faz o bash (1) interromper o script e (2) lançar o sinal `ERR`.


### Segredo #2: executando um comando na linha onde o `ERR` é lançado

Esse segredo é uma das chaves para alcançar o objetivo que queremos. Continue comigo...

Quando temos uma situação do tipo:
```bash
# requer 'set -e'
trap 'echo "ERR capturado, abortando!"' ERR
```

O sinal `ERR` será lançado quando qualquer comando do script terminar com uma falha (ou seja, status code diferente de zero).

A grande sacada aqui é que esse `echo` que estamos passando para o `trap` será executado como se estivesse na linha onde o `ERR` foi capturado!

Agora se você coloca nesse `echo` uma referência a variável `LINENO` que mencionamos anteriormente... 🤯

```bash
# requer 'set -e'
trap 'echo "ERR capturado na linha ${LINENO}!"' ERR
```

É sério... Quando eu percebi isso eu quase chorei de emoção. 🥲


### Tentando juntar tudo isso

Vamos juntar o conhecimento que adquirimos no [artigo anterior](/bash-rigoroso) com o que foi foi exposto aqui e vamos logo ao truque que vai mudar a sua vida:

```bash
# OBS: isso só funciona se usarmos 'set -e'
trap 'echo "${BASH_SOURCE}:${LINENO}:${FUNCNAME:-}"' ERR
```

Vamos ver essa belezura em ação com esse exemplo ilustrativo:

```bash
#!/usr/bin/env bash
# find-user.sh
# Encontra um usuário dentro do /etc/passwd
# e imprime o nome em maiúsculo.

set -euo pipefail

trap 'echo "ERRO EM: ${BASH_SOURCE}:${LINENO}:${FUNCNAME:-}"' ERR

# se não passar um usuário, use um default
username="${1:-usuário inválido}"

grep "${username}" /etc/passwd \
  | cut -d: -f1 \
  | tr [:lower:] [:upper:]

# pra fingir que aqui teria mais comando,
# vamos colocar esse echo
echo fim
```

Executando:
```txt
$ ./find-user.sh meleu
MELEU
fim

$ ./find-user.sh 
ERRO EM: ./find-user.sh:15:

```

Bacana... Recebemos o nome do script e a linha onde o erro ocorreu. A nome da função veio em branco mas era de se esperar, pois o erro não correu dentro de função alguma.

Acontece que eu sou um cara rigoroso com meu estilo de codificação. E uma das coisas que eu pratico nos meus códigos da vida real é colocar tudo dentro de uma função. Portanto eu vou refatorar o exemplo acima pra ficar assim:

```bash
#!/usr/bin/env bash
# find-user.sh
# Encontra um usuário dentro do /etc/passwd
# e imprime o nome em maiúsculo.

set -euo pipefail

trap 'echo "ERRO EM: ${BASH_SOURCE}:${LINENO}:${FUNCNAME:-}"' ERR

main() {
  # se não passar um usuário, use um default
  local username="${1:-usuário inválido}"

  grep "${username}" /etc/passwd \
    | cut -d: -f1 \
    | tr [:lower:] [:upper:]
}

main "$@"
```

Agora vamos executar:
```txt
$ ./find-user.sh meleu
MELEU

$ ./find-user.sh

$ # 😳 como assim?

$ ./find-user.sh UsuarioQualquer

$ # 😕 cadê o trap em ação?!
```

Quebrei a cara! O `trap` não fez o que eu esperava que ele fizesse... 😔

Mas não vamos nos abalar! Vamos conhecer mais um segredo

### Segredo #3: fazendo o `trap` ser herdado pelas funções

Esse segredo está "escondido" no `help set`. Lá vemos o seguinte (tradução minha):
```txt
-E  Se ligado, o trap com ERR será herdado pelas funções.
```

Opa! Parece ser exatamente o que queremos! Agora sim vamos juntar os pontos.


## Juntando tudo

Agora que já temos tudo que precisamos vamos refatorar meu script adicionando a opção `set -E`:
```bash
#!/usr/bin/env bash
# find-user.sh
# Encontra um usuário dentro do /etc/passwd
# e imprime o nome em maiúsculo.

set -Eeuo pipefail

trap 'echo "ERRO EM: ${BASH_SOURCE}:${LINENO}:${FUNCNAME:-}"' ERR

main() {
  # se não passar um usuário, use um default
  local username="${1:-usuário inválido}"

  grep "${username}" /etc/passwd \
    | cut -d: -f1 \
    | tr [:lower:] [:upper:]

  # pra fingir que aqui teria mais comando,
  # vamos colocar esse echo
  echo fim
}

main "$@"
```

Executando:
```txt
$ ./find-user.sh meleu
MELEU
fim

$ ./find-user.sh 
ERRO EM: ./find-user.sh:16:main

```

🥳🎉 Yeah!! É exatamente isso que queremos!!!

## Conclusão

Juntando as peças desse quebra cabeça envolvendo `help trap`, `help set`, trechos do `man bash` e experimentações, chegamos a seguinte conclusão:

1. use `set -euo pipefail` pelos motivos explicados no [artigo anterior](/bash-rigoroso).
2. `set -e` faz o bash:
    - interromper execução do script quando qualquer comando terminar com status diferente de zero.
    - lançar o sinal `ERR`.
3. o `trap` pode ser usado para capturar esse `ERR`.
4. quando o `trap` captura o `ERR` ele executa o comando que foi passado pra ele como se estivesse na linha onde o `ERR` foi lançado.
5. a variável `LINENO` mostra a linha do script onde ela está sendo invocada.
6. consequência de 4. e 5.:`trap 'echo "${LINENO}"' ERR` vai mostrar a número da linha onde o `ERR` foi lançado.
7. use `set -E` para o que o `trap 'comando...' ERR` seja herdado pelas funções.

Convertendo isso em código:
```bash
# use isso nos seus scripts:
set -Eeuo pipefail

trap 'echo "ERRO EM: ${BASH_SOURCE}:${LINENO}:${FUNCNAME:-}"' ERR
```

Obviamente que o `echo` pode ser incrementado. Por exemplo, colocando a mensagem em vermelho com caracteres de escape ANSI. Use sua criatividade! 😉


## Fontes

- `help set`
- `help trap`
- `man bash`
- experimentações...
- Eu tive a ideia de usar `$BASH_SOURCE:$LINENO:$FUNCNAME` quando eu estava [lendo sobre debugging no BashGuide do Greg's Wiki](https://mywiki.wooledge.org/BashGuide/Practices#Activate_Bash.27s_Debug_Mode)
