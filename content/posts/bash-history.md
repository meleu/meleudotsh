---
title: Melhorando seu .bash_history
description: >
  Deixe o seu .bash_history um pouco mais inteligente.
tags:
  - configuracoes
  - dotfiles
date: 2022-03-30T10:40:00-03:00
cover:
  image: "img/bash-history.png"
  alt: "configuração do bash history no .bashrc"
---

Neste artigo veremos algumas configurações interessantes para se fazer no seu ambiente de forma a melhorar o seu `.bash_history` e o output do comando `history`.

## Motivação

Eu geralmente uso o `tmux` com várias sessões de bash abertas, e como raramente eu desligo o computador, muitas vezes essas sessões duram dias. Normalmente o bash grava o histórico da sessão atual no `.bash_history` apenas ao final da sessão.

O problema disso é que muitas vezes eu quero consultar o histórico de comandos em busca de algo que fiz dias atrás, mas o como eu não encerrei a sessão onde o comando foi executado, esse histórico ainda não foi salvo no meu `.bash_history`.

Pois bem, resolvi investir um tempo estudando configs relacionadas ao bash history e nesse artigo mostro o resultado dos meus estudos.

> A propósito, caso queira bisbilhotar as configs que eu uso, dê uma conferida no [meu `.bashrc`](https://github.com/meleu/homedir/blob/master/.bashrc).


## Configurações com variáveis de ambiente

Vamos ver aqui as configurações que são feitas através de variáveis de ambiente.

### `HISTCONTROL`

Controla como os comandos serão gravados no histórico.

As opções são as seguintes:

- `ignorespace` - ignora comandos começando com espaço
- `ignoredups` - ignora comandos duplicados (da sessão atual)
- `ignoreboth` - equivalente a usar as duas opções acima
- `erasedups` - elimina entradas duplicadas de todo o histórico (não apenas da sessão atual)

Caso queira usar mais de uma opção no `HISTCONTROL`, elas devem ser separadas com `:` dois-pontos. Exemplo: `HISTCONTROL='ignoreboth:erasedups'`

**Minha preferência:**

```bash
export HISTCONTROL='ignoreboth'
```

Eu uso `ignoreboth`, que é equivalente a `ignorespace:ignoredups`.

Desta forma eu tenho as duas features:

- quando eu não quero armazenar um comando no meu history (por exemplo quando tem alguma credencial) eu adiciono um espaço em branco antes do comando.
- não armazeno comandos repetidos que foram executados em uma mesma sessão.

Eu não curto o `erasedups` pois em alguns momentos eu gosto de olhar no meu history algo que fiz no passado e ter o comando em ordem me ajuda a fazer uma "reconstituição dos fatos". Com o `erasedups`, os comandos do passado iriam sumir caso eu os repetisse.


### `HISTIGNORE`

Nesta variável podemos adicionar uma lista, separada por `:` dois-pontos, de comandos que não queremos armazenar no nosso `.bash_history`.

**Minha preferência:**

```bash
export HISTIGNORE='ls:ls -lah:history:pwd:htop:bg:fg:clear'
```


### `HISTTIMEFORMAT`

Configura um timestamp para ser exibido quando vc usa o comando history.

Veja este exemplo:

```
$ history 3
 2426  cat /etc/passwd
 2427  vim ~/.bash_history 
 2428  history 3
 
$ export HISTTIMEFORMAT="%F %T "

$ history 3
 2428  2022-03-29 17:32:25 history 3
 2429  2022-03-29 17:32:36 export HISTTIMEFORMAT="%F %T$ "
 2430  2022-03-29 17:32:40 history 3
```

Você pode ver as opções possíveis no `man strftime`.

**Minha preferência:**

```bash
export HISTTIMEFORMAT="%F %T$ "
```

Eu gosto de deixar esse `$` após o timestamp pois me ajuda na hora de mandar um `history | grep something | cut -d$ -f2-`.


### `PROMPT_COMMAND`

Essa não é uma variável exclusiva de configuração do history, mas vamos usá-la para esse propósito aqui.

O `PROMPT_COMMAND` nada mais é do que um comando (ou lista de comandos) que são executados antes do prompt ser exibido. E o que vamos fazer aqui é executar o comando `history -a` para salvar o history atual em disco após cada comando.

**Minha preferência:**

```bash
export PROMPT_COMMAND='history -a'
```

Observação: esta variável também pode ser um array. Veja `man bash` para mais detalhes.


### `HISTSIZE` e `HISTFILESIZE`

A manpage do bash não deixa muito claro qual é a diferença dessas duas variáveis, mas através de experimentações aqui eu cheguei a conclusão que:

- `HISTSIZE` define quantos comandos estarão visíveis no output do seu comando `history`.
- `HISTFILESIZE` define quantos comandos serão armazenados no seu arquivo `~/.bash_history` (removendo os mais antigos quando esse valor é atingido).

Em qualquer uma dessas variáveis, um valor negativo significa "ilimitado".

**Minha preferência:**

```bash
export HISTSIZE=10000
export HISTFILESIZE=20000
```

Pra ser sincero não tenho nenhum dado concreto para justificar esses valores. Apenas uso e acho razoável.


## Configurações com `shopt`

Agora veremos duas opções de `shopt` que estão relacionadas com o bash history.


### `histappend`

Faz o bash acrescentar o history da sessão atual ao `~/.bash_history`. Se essa opção estiver desativada, o arquivo será sobrescrito (o conteúdo anterior será perdido).

**Minha preferência:**

Obviamente, eu deixo essa opção habilitada.

```bash
shopt -s histappend
```


### `cmdhist`

Um comando com múltiplas linhas é salvo como se fosse apenas um comando.

Veja na prática:

```
$ # habilitando o cmdhist

$ shopt -s cmdhist

$ echo meleu \
> | tr [:lower:] [:upper:]
MELEU

$ history 3
 2444  shopt -s cmdhist
 2445  echo meleu | tr [:lower:] [:upper:]
 2446  history 3
 
$ # 👆 observe a linha 2445 acima

$ # desabilitando o cmdhist

$ shopt -u cmdhist

$ echo meleu \
> | tr [:lower:] [:upper:]
MELEU

$ history 5
 2447  # desabilitando o cmdhist
 2448  shopt -u cmdhist
 2449  echo meleu \
 2450  | tr [:lower:] [:upper:]
 2451  history 5

$ # 👆 observe as linhas 2449 e 2450 acima
```

**Minha preferência:**

```bash
shopt -s cmdhist
```

## Configuração pra colocar no seu `.bashrc`

Se você curtiu as minhas preferências listadas acima, basta colocar o seguinte conteúdo no seu `.bashrc`:

```bash
# History Options
#################################
# explicação destas opções em
# https://meleu.sh/bash-history

export HISTCONTROL='ignoreboth'
export HISTIGNORE='ls:ls -lah:history:pwd:htop:bg:fg:clear'
export HISTTIMEFORMAT="%F %T$ "
export PROMPT_COMMAND='history -a'
export HISTSIZE=10000
export HISTFILESIZE=20000

shopt -s histappend
shopt -s cmdhist
```

Caso tenha curiosidade de saber como é o meu `.bashrc`, dê uma olhada no meu [dotfiles no github](https://github.com/meleu/dotfiles/blob/master/.bashrc).


## Fontes

- man bash (procure por `HISTCONTROL`, `HISTIGNORE`, `HISTTIMEFORMAT`, etc.)
- <https://linuxhint.com/bash_command_history_usage/>
- <https://gist.github.com/ckabalan/7d374ceea8c2d9dd237d763d385cf2aa>