---
title: "Chega de commitar no repositório do trabalho com seus dados do github (e vice-versa)!"
description: >
  Depois de ler esse artigo você também nunca mais vai commitar no github com seu email do trabalho!
tags:
  - git
  - configuracoes
date: 2022-04-20T12:25:19-03:00
cover:
  image: "img/git-multiconfig.png"
  alt: git-multiuser
---

Você é programador, trabalha para mais de uma empresa e também contribui com projetos open source no github, gitlab, codeberg, etc.

Certamente você já passou por aquela situação de fazer um commit no repositório do trabalho usando username/email que usa nos seus projetos pessoais ou open source. Ou o que é mais delicado, você faz um commit num repositório público usando seu email da empresa. E agora o histórico de commit do seu projeto Open Source agora tem seu nome real e seu email da empresa... 😓

Você que já é calejado com git já sabe que podemos mandar um `git config user.name "MEU_USERNAME"` e um `git config user.email "MEU@EMAIL.COM"` para cada projeto individualmente. Mas se você trabalha com vários repositórios, isso vai se tornar uma tarefa tediosa e certamente vai chegar um momento que você vai esquecer de fazer isso.

Vamos ver nesse artigo uma maneira bem simples de evitar esses inconvenientes usando puramente alguns recursos do git.

## Passo a passo

Resumidamente, vamos fazer isso aqui:

1. ter um diretório específico para cada organização que você trabalha/contribui
2. ter um `gitconfig` específico para cada organização.
3. configurar seu `.gitconfig` global para que ele aplique o `gitconfig` específico de cada organização a todos os projetos armazenados dentro do diretório da respectiva organização.

Agora vamos detalhar estes passos.

### Passo 1: crie um diretório específico para cada organização

Para efeitos de ilustração vamos considerar que queremos configurar uma conta para o github e outra para gitlab.

Eu costumo guardar meus códigos em `~/src/`, portanto eu criaria um diretório para cada organização da seguinte forma:
```txt
# cria um diretório para cada organização
mkdir -p ~/src/github
mkdir -p ~/src/gitlab
```

### Passo 2: crie um gitconfig para cada organização

Agora vamos criar um arquivo `gitconfig` dentro de cada um dos diretórios que criamos anteriomente (eu prefiro deixar o arquivo visível, se quiser ocultar basta nomeá-los como `.gitconfig`).

Primeiramente vamos criar `~/src/github/gitconfig` com o seguinte conteúdo:
```
[user]
  email = SEU_EMAIL@EMPRESA.COM
  name = SEU_USERNAME
```

Faça a mesma coisa para o arquivo `~/src/gitlab/gitconfig`.


### Passo 3: customize seu `.gitconfig` global

Agora que já temos um diretório para cada organização e um `gitconfig` em cada um desses diretórios, precisamos dizer ao nosso `~/.gitconfig` global o que fazer com essas coisas.

Vamos portanto abrir o nosso `~/.gitconfig` (se o arquivo não existir, apenas crie-o) e adicionar o seguinte conteúdo **ao final do arquivo**:

```
# diretório onde ficarão os projetos
# hospedados no github
[includeIf "gitdir:~/src/github/**"]
  path = ~/src/github/gitconfig

# projetos hospedados no gitlab
[includeIf "gitdir:~/src/gitlab/**"]
  path = ~/src/gitlab/gitconfig
```

Se tivéssemos que traduzir aquela primeira configuração para o português, seria algo assim:

> Se o repositório git local estiver em qualquer subdiretório de `~/src/github/`, inclua as configurações presentes em `~/src/github/gitconfig`.

E prontinho! É só isso!

Agora você só precisa se certificar de clonar os repositórios nos diretórios corretos de cada organização. E assim o problema estará resolvido.

## Fontes

- `man git config` na parte entitulada "Includes"
    - também disponível online: <https://www.git-scm.com/docs/git-config#_includes>
