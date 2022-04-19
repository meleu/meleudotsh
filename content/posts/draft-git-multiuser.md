---
title: "Chega de commitar no repositório do trabalho com seus dados do github (e vice-versa)!"
description: >
  Depois de ler esse artigo você nunca mais vai commitar no github com seu email do trabalho!
tags:
  - draft
  - git
date: 2022-04-27T22:25:19-03:00
cover:
  image: "img/git-multiuser.png"
  alt: git-multiuser
draft: true
---

## Motivação

Você é programador, trabalha para mais de um cliente e também contribui com projetos open source no github, gitlab, codeberg, etc.

Certamente você já passou por aquela situação de fazer um commit no github usando seu username/email da empresa. Ou então fez um commit no repositório do trabalho usando username/email que usa pros seus projetos pessoais ou open source.

Vamos ver nesse artigo uma maneira de evitar esses inconvenientes usando puramente alguns recursos do git.


###





Você trabalha para mais de um cliente e ainda contribui com 

## gitconfigs to be applied to specific directories

If you work for multiple clients - or if you work for a company and contribute to open source projects - you probably already faced the situation where you made a git commit with the wrong account. Now the github commit history has your real name and your work email... :/

A good solution for this is to:

1. have a specific dir for a specific customer's source code and
2. have a specific gitconfig to be applied to all git repositories inside that directory.

Let's just do that:

```bash
# creating specific dirs
mkdir -p ~/src/client1
mkdir -p ~/src/client2

# gitconfig for client1
echo "
[user]
  email = meleu.dev@client1.com
  name = meleu" > ~/src/client1/gitconfig

# gitconfig for client2
echo "
[user]
  email = meleu.dev@client2.com
  name = meleu" > ~/src/client2/gitconfig
```

Once each directory has its own gitconfig, now we must setup the global `.gitconfig` to apply them at specific situations using `includeIf`:

```
# add this to your ~/.gitconfig

[includeIf "gitdir:~/src/client1/**"]
  path = ~/src/client1/gitconfig

[includeIf "gitdir:~/src/client2/**"]
  path = ~/src/client2/gitconfig
```

Once it's done, your configuration in `~/src/client1/gitconfig` will be applied to all cloned repositories inside that directory (same for `client2`).

