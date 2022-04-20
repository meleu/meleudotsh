---
title: "Chega de commitar no repositório do trabalho com seus dados do github (e vice-versa)!"
description: >
  Depois de ler esse artigo você nunca mais vai commitar no github com seu email do trabalho!
tags:
  - draft
  - git
date: 2022-04-17T22:25:19-03:00
cover:
  image: "img/git-multiconfig.png"
  alt: git-multiuser
draft: true
---

Você é programador, trabalha para mais de uma empresa e também contribui com projetos open source no github, gitlab, codeberg, etc.

Certamente você já passou por aquela situação de fazer um commit no repositório do trabalho usando username/email que usa nos seus projetos pessoais ou open source. Ou o que é mais delicado, você faz um commit num repositório público usando seu email da empresa. E agora o histórico de commit do seu projeto Open Source agora tem seu nome real e seu email da empresa... 😓

Vamos ver nesse artigo uma maneira bem simples de evitar esses inconvenientes usando puramente alguns recursos do git.

## Resumo da solução

Resumidamente, vamos fazer isso aqui:

1. ter um diretório específico para cada organização que você trabalha/contribui
2. ter um `gitconfig` específico para cada organização.
3. configurar seu `.gitconfig` global para que ele aplique o `gitconfig` específico de cada organização a todos os projetos armazenados dentro do diretório da respectiva organização.


## Passo 1: crie um diretório específico para cada organização

Para efeitos de ilustração vamos considerar que queremos configurar uma conta para o github e outra para gitlab.

Eu costumo guardar meus códigos em `~/src/`, portanto eu criaria um diretório para cada organização da seguinte forma:
```txt
# cria um diretório para cada organização
mkdir -p ~/src/github
mkdir -p ~/src/gitlab
```

## Passo 2: crie um gitconfig para cada organização

Agora vamos criar um arquivo `gitconfig` dentro de cada um dos diretórios que criamos anteriomente (eu prefiro deixar o arquivo visível, se quiser ocultar basta nomeá-los como `.gitconfig`).

Primeiramente vamos criar `~/src/github/gitconfig` com o seguinte conteúdo:
```ini
[user]
  email = SEU_EMAIL@EMPRESA.COM
  name = SEU_USERNAME
```

Faça a mesma coisa para o arquivo `~/src/gitlab/gitconfig`.


## Passo 3: customize seu `.gitconfig` global

Once each directory has its own gitconfig, now we must setup the global `.gitconfig` to apply them at specific situations using `includeIf`:

```
# add this to your ~/.gitconfig

[includeIf "gitdir:~/src/github/**"]
  path = ~/src/github/gitconfig

[includeIf "gitdir:~/src/gitlab/**"]
  path = ~/src/gitlab/gitconfig
```

Once it's done, your configuration in `~/src/client1/gitconfig` will be applied to all cloned repositories inside that directory (same for `client2`).

