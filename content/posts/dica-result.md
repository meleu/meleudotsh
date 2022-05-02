---
title: "Uma dica simples que vai fazer você ganhar muito tempo quando estiver escrevendo shell scripts"
description: >
  Veja como um simples alias pode te ajudar a economizar muito tempo quando estiver escrevendo e testando trechos dos seus scripts.
tags:
  - fundamentos
date: 2020-01-11T15:51:17-03:00
cover:
  image: "img/dica-result.png"
  alt: "dica result"
---

Quando estamos escrevendo um shell script é muito comum precisarmos verificar se um teste resultará em verdadeiro ou falso. Fazer esses testes muitas vezes é tedioso, principalmente quando o comando não gera saída alguma (ou seja, não imprime nada na tela).

Veremos aqui como lidar com isso de maneira simples, fácil e muito prática.

## A origem da ideia

Geralmente eu costumo fazer essa checagem, assim:

```shell-session
$ comando && echo verdadeiro || echo falso
```

O `comando` no exemplo acima está representando o comando a ser executado. É só uma abstração, OK?

O símbolo `&&` é uma maneira do bash dizer "se o comando for finalizado com sucesso, execute o comando a seguir". Portanto isso aqui: `comando && echo verdadeiro`, significa "execute o comando e se ele terminar com sucesso, escreva 'verdadeiro' na tela".

Temos também o símbolo `||`, que significa "se o comando for finalizado sem sucesso, execute o comando a seguir". Portanto isso aqui: `comando || echo falso`, significa "execute o comando e se ele falhar, escreva 'falso' na tela".

Uma vez que entendemos a finalidade destes símbolos, podemos utilizá-los para fazer testes rápidos, como nos exemplos a seguir:

```shell-session
$ file='arquivo.txt'
$ [[ -f "$file" ]] && echo verdadeiro || echo falso
falso
$ user='meleu'
$ grep -q "$user" /etc/passwd && echo verdadeiro || echo falso
verdadeiro
```

Evidentemente que poderíamos alcançar o mesmo resultado utilizando `if-else`, mas isso não seria nada prático.

Aliás, sejamos honestos, escrever `&& echo verdadeiro || echo falso` toda hora também não é lá muito prático...


## Criando um alias

Para evitar a digitação excessiva, podemos criar um _alias_. Um alias é um "atalho" para um comando grande e/ou complexo.

Se você não conhece a técnica do alias muito provavelmente você já a está usando e nem sabia. Digite `alias` no seu terminal e veja o resultado.

Eis um exemplo típico:

```shell-session
$ alias
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias grep='grep --color=auto'
alias l='ls -CF'
alias la='ls -A'
alias ll='ls -alF'
alias ls='ls -F --color=auto'
```

Vamos pegar a última linha acima como exemplo.

Aquele alias nos diz que quando o usuário executa um `ls`, ele não está simplesmente executando o `ls` "puro". Ele está também visualizando a lista onde o nome dos arquivos aparecem coloridos (`--color=auto`) e com um caractere no final do nome (`-F`) que servem para ajudar a distinguir o tipo do arquivo (ex.: a cor azul e o caractere `/` significam diretório).

Uma vez entendido o que é e pra que serve um alias, vamos criar um para nos ajudar nos testes que mencionamos anteriormente:

```shell-session
$ alias result='echo verdadeiro || echo falso'
```

Pronto! Agora vamos ver como isso nos ajuda:

```shell-session
$ file='arquivo.txt'
$ [[ -f "$file" ]] && result
falso
$ user='meleu'
$ grep -q "$user" /etc/passwd && result
verdadeiro
```

Como você deve ter observado, uma vez definido o alias, basta adicionar `&& result` após o comando que você quer testar.

Agora sim, bem prático, né não?!

Só tem um porém: após o logoff o seu alias é esquecido. Para torná-lo "permanente" precisamos salvá-lo no nosso `.bashrc`.


## Salvando o alias no seu `.bashrc`

O arquivo `.bashrc` que fica localizado no seu diretório home, é basicamente script que é executado toda vez que você inicia o bash de maneira interativa. Portanto basta salvarmos nosso alias nesse arquivo que teremos ele a nossa disposição sempre que precisarmos.

Vá no seu diretório home (`/home/nome_do_usuario`) e procure o arquivo `.bashrc` (se não existir, basta criá-lo). Abra o arquivo no seu editor de texto de preferência.

Provavelmente já vai existir bastante coisa escrita nele. Se quiser tentar dar uma olhadela no conteúdo do arquivo e tentar entender o que está acontecendo lá, você pode acabar se deparando com algumas coisas legais e aprender coisa nova. Mas vamos concluir a nossa dica logo! Vá no final do arquivo e adicione a seguinte linha:

```
alias result='echo verdadeiro || echo falso'
```

E pronto! Agora você já pode testar aquele _one-liner_ maroto apenas adicionando `&& result` ao final da linha! ;)