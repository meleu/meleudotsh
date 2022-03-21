---
title: "O que o #! realmente faz?"
date: 2019-12-21T00:24:42-03:00
description: >
  O que exatamente acontece quando executamos um arquivo que começa com '#!' - também conhecido como shebang.
tags:
  - fundamentos
  - boas-praticas
cover:
  image: "img/shebang.png"
  alt: "shebang"
---

Para tornar o primeiro post deste blog bem simbólico, vamos falar sobre a primeira coisa que devemos colocar em um shell script: o `#!` (vulgarmente chamado de [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) ou hashbang).

## Como o `#!` funciona?

O `#!` shebang serve para dizer ao kernel qual será o interpretador a ser utilizado para executar os comandos presentes no arquivo.

Quando executamos um arquivo que começa com `#!`, o kernel abre o arquivo e pega tudo que está escrito após o shebang até o final da linha. Em seguida ele tenta executar um comando com o conteúdo desta string adicionando como parâmetro o nome do próprio arquivo.

Portanto se você tem arquivo executável chamado `meuscript.sh`, contendo um shell-script e começando com `#!/bin/bash`, quando você executa esse arquivo o kernel vai executar `/bin/bash meuscript.sh`.

Nos exemplos a seguir você verá isso muito claramente.

Vamos começar com o clássico `hello.sh`:
```
#!/bin/bash
echo "Hello World!"
```

Quando é digitado no shell:
```
$ ./hello.sh
```

O kernel vai ver o `#!` e vai ler o que vem a seguir até o final da linha, neste caso `/bin/bash`. E então o que é executado é equivalente a você fazer isso no shell:
```
$ /bin/bash hello.sh
```

## Demonstrações

### com o `cat`

Vejamos agora um exemplo usando `#!/bin/cat`. O nome do arquivo é `hashbangcat` 
```txt
#!/bin/cat
Todo o conteúdo deste arquivo será
exibido quando ele for executado
(inclusive o '#!/bin/cat').
```

O que é lido após o hashbang: `/bin/cat`
Nome do arquivo: `hashbangcat`

Portanto será executado: `/bin/cat hashbangcat`

Veja:
```txt
$ ./hashbangcat
#!/bin/cat
Todo o conteúdo deste arquivo será
exibido quando ele for executado
(inclusive o '#!/bin/cat').
```

### com o `echo`

Agora um exemplo que deixa bastante claro que as coisas acontecem exatamente como estou dizendo. Vamos usar `#!/usr/bin/echo`. O seguinte arquivo se chama `hashbangecho`:
```txt
#!/usr/bin/echo
O conteúdo deste arquivo *não* será
exibido quando ele for executado.
```
Agora vamos executar esse arquivo:
```
$ ./hashbangecho
./hashbangecho
```

A saída do comando foi o nome do arquivo pois o que realmente foi executado foi `/usr/bin/echo ./hashbangecho`.

Se passarmos parâmetros, eles também serão repassados. Conforme veremos no exemplo a seguir.


### com o `ls`

Este arquivo se chama `hashbangls.sh`:

```txt
#!/bin/ls
O conteúdo aqui não importa.
```

Agora ele sendo executado:

```txt
$ ./hashbangls.sh
./hashbangls.sh
$ ./hashbangls.sh notfound
/bin/ls: cannot access 'notfound': No such file or directory
./hashbangls.sh
$ ./hashbangls.sh -l
-rwxr-xr-x 1 meleu meleu 41 Dec 20 14:42 ./hashbangls.sh
```


## Dúvidas Comuns

### Por que algumas pessoas usam `#!/usr/bin/env bash`?

Provavelmente você já viu alguns scripts começando com `#!/usr/bin/env bash` onde normalmente você costuma ver `#!/bin/bash`. O motivo disso é que acredita-se que isso aumenta a portabilidade do seu script (mesmo que isso possa ser questionável).

O comando `env`, se usado sem argumento algum, imprime uma (grande) lista com todas as variáveis de ambiente (**env**ironment variables). Mas se o `env` é usado seguido de um comando, ele executa aquele comando em uma outra instância do shell.

🤔 - **OK, mas como que isso influencia na portabilidade?!**

Quando você usa `#!/bin/bash` você está explicitamente dizendo que o executável `bash` está no diretório `/bin/`. Isso parece ser o padrão em todas as distribuições Linux, mas existem outros "sabores" de Unix onde isso pode não acontecer. Podem ocorrer casos, por exemplo, onde o `bash` pode estar presente em `/usr/bin/`. E nesse caso, se você tentar executar um script que tenha `#!/bin/bash`, o kernel não encontraria o arquivo `/bin/bash` e você iria receber um erro do tipo: `bad interpreter: No such file or directory`.

Por outro lado, se você executa `run bash`, o comando `env` vai procurar pelo `bash` em seu `$PATH`, e então vai executar o primeiro `bash` que encontrar nos diretórios listados no `$PATH`. Normalmente o `bash` está em `/bin/`, mas também pode haver um caso em que seu script está sendo executado em um sistema que tem o `bash` em outro diretório (como `/usr/bin/` ou `/home/user/bin/`).

Portanto, para que o seu script possa ser usado nos mais diversos ambientes, algumas pessoas recomendam usar a técnica do `#!/usr/bin/env bash`.

🤔 - **Mas espera aí! O que garante que o `env` sempre estará em `/usr/bin/`?**

A resposta é simples: **Não há garantia alguma!** 😇

A recomendação é baseada no que é comumente visto nos sistemas Unix. Mas ter o `bash` em `/bin/` também é bastante comum.

O importante de você entender aqui é: as duas opções funcionam na maioria dos casos porém nenhuma das duas é uma bala de prata que vai resolver tudo.

Se você quer minha opinião, aí vai: as duas soluções são igualmente boas, escolha qualquer uma e seja feliz.

- Se é um projetinho pessoal seu: use o jeito que mais te agrada.
- Se é um projeto que já está em andamento: siga o que já é praticado pelo projeto (sem nenhuma neura de não ser o seu método favorito - lembre-se: as duas soluções são boas o suficiente).

🤨 - **Tá bom meleu, mas qual das duas soluções você usa?**

Eu costumo usar o `#!/usr/bin/env bash`.

Principal motivo: quando eu estava contribuindo com o projeto [RetroPie](https://github.com/RetroPie/RetroPie-Setup), esse era o método utilizado lá.

Pode ser que eu use `#!/bin/bash` aqui e acolá... Já cheguei a conclusão que as duas soluções são igualmente boas. Mas eu tendo a preferir utilizar o método com o `env`.

Existe ainda alguns casos especiais, **que não estão relacionados especificamente ao bash**, onde o useo do `#!/usr/bin/env` é uma opção melhor: quando você quer executar um script Python ou usando NodeJS.

Vamos ver um exemplo bobo em NodeJS. Se eu quero executar um script simplesmente chamando o nome do arquivo, eu poderia criar um arquivo assim:

```js
#!/usr/bin/node
console.log('Hello World from NodeJS');
```

O problema aqui é que eu estou partindo do princípio que o `node` está em `/usr/bin/`. Mas acontece que eu geralmente instalo o Node usando o [Node Version Manager](https://github.com/nvm-sh/nvm) (ao invés de usar o node disponibilizado pelos `apt-get` da vida). Portanto meu node fica nesse diretório aqui:

```txt
$ which node
/home/meleu/.nvm/versions/node/v14.15.1/bin/node
```

Eu jamais iria iniciar o meu script com um shebang tipo `#!/home/meleu/.nvm/versions/node/v14.15.1/bin/node`!

Portanto uma solução bem interessante é usar `#!/usr/bin/env node`.


### E se eu não usar `#!` algum?

Recomendo fortemente que você **nunca** escreva/execute um script sem um shebang `#!`.

Como eu disse, o shebang diz ao seu kernel qual interpretador é para ser usado para executar os comandos presentes no arquivo. Se você executa um script sem especificar o interpretador, o shell vai disparar uma outra instância de si mesmo e tentar executar o script nessa nova instância. Isso significa que ele executará seja lá o que for que estiver escrito no arquivo, mesmo se for um script feito para ksh, zsh, dash, fish, node, python ou qualquer outra linguagem...

Portanto **sempre use um shebang nos seus scripts!**.


## Fontes

- http://mywiki.wooledge.org/BashProgramming#Shebang
- https://wiki.bash-hackers.org/scripting/basics#the_shebang
- `man env`
- [Aqui está um email de Dennis Ritchie](https://www.in-ulm.de/~mascheck/various/shebang/4.0BSD_newsys_sys1.c.html) de 1980, falando sobre esses "caracters mágicos".
- [Node.js shebang](https://alexewerlof.medium.com/node-shebang-e1d4b02f731d)

