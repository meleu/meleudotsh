---
title: Mantenha a consistência na formatação do seu código com shfmt
description: >
  Além de manter o seu código com uma formatação consistente, o shfmt também pode tornar aquele código obscuro que você achou por aí em algo mais legível.
tags:
  - boas-praticas
  - ferramentas
date: 2022-05-12T15:14:44-03:00
cover:
  image: "img/shfmt.png"
  alt: shfmt
draft: true
---

Neste artigo vamos conhecer o `shfmt`. Uma ferramenta que vai te ajudar a manter seu código com uma formatação consistente, e também pode ser usado para tornar legível algum código de outra pessoa que você queira examinar.

Mostrarei como instalar, alguns exemplos de uso e como configurar o seu editor (VSCode e vim) para formatar seu código assim que você salvar.

## Contexto pessoal

Desde que conheci o `shfmt` eu o uso para manter o meu próprio código com uma formatação consistente com as minhas preferências. Mas aonde ele me deixa mais feliz é quando preciso trabalhar num código escrito por uma pessoa sem experiência com programação shell.

Sabe como é né... O shell é uma linguagem muito poderosa e permissiva. A barreira de entrada é bem pequena e devido ao poder do shell você consegue rapidamente resolver o que você precisa resolver. Por um lado isso é bom (afinal, resolver problemas é uma coisa boa!), mas em contrapartida acaba gerando muito código bagunçado. Difícil de ler e de manter.

Pra agravar o cenário, geralmente quem escreve shell scripts, são SysAdmins. Historicamente um SysAdmin é uma pessoa treinada para manter o sistema estável, bem configurado e seguro. Focar em aprender técnicas avançadas de programação, *Clean Code*, e coisas do gênero normalmente não é uma prioridade para um SysAdmin.

Como resultado, não é muito raro encontrar por aí código bem difícil de analisar/atualizar/refatorar.

Quando me deparo com um cenário desses, a primeiríssima coisa que faço é rodar o `shfmt` pra tornar o código minimamente legível (e na sequência eu rodo [shellcheck](/shellcheck)).


## Instalando o shfmt

Apesar de eu ser um grande fã do `shfmt`, tem uma coisa que me deixa intrigado: no [README do projeto](https://github.com/mvdan/sh) não tem uma instrução clara de como instalar.

Mas vamos ultrapassar essa barreira que você vai ver que vai valer a pena


## Formatando uma "maçaroca" de código 




## Instale shfmt no seu editor

### VSCode

### vim



aplicar nesse script aqui do transfer.sh
```bash
transfer(){ if [ $# -eq 0 ];then echo "No arguments specified.\nUsage:\n transfer <file|directory>\n ... | transfer <file_name>">&2;return 1;fi;if tty -s;then file="$1";file_name=$(basename "$file");if [ ! -e "$file" ];then echo "$file: No such file or directory">&2;return 1;fi;if [ -d "$file" ];then file_name="$file_name.zip" ,;(cd "$file"&&zip -r -q - .)|curl --progress-bar --upload-file "-" "https://transfer.sh/$file_name"|tee /dev/null,;else cat "$file"|curl --progress-bar --upload-file "-" "https://transfer.sh/$file_name"|tee /dev/null;fi;else file_name=$1;curl --progress-bar --upload-file "-" "https://transfer.sh/$file_name"|tee /dev/null;fi;}
```
