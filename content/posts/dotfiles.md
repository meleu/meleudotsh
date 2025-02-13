---
title: "Uma maneira limpa e inteligente de gerenciar seus dotfiles em um repositório git"
description: >
  Veja como alguns segredinhos do git podem tornar o gerenciamento de seus dotfiles uma coisa simples e limpa. Não instale ferramenta alguma, apenas use o git.
tags:
  - dotfiles
  - git
  - configuracoes
date: 2022-04-28T13:32:13-03:00
cover:
  image: "img/dotfiles.png"
  alt: "configuração dos meus dotfiles"
---

Neste artigo conheceremos uma maneira limpa de gerenciar seus dotfiles usando apenas o git.

Não será necessária ferramenta adicional alguma, você não precisará instalar nada, nem criar links simbólicose e nem escrever script algum. Tudo que precisamos é pura e simplesmente o git.

Como resultado da técnica descrita aqui, você terá um repositório dotfiles refletindo exatamente a estrutura de diretórios e localização dos arquivos que deve estar em seu homedir.

## Não quero ler isso tudo! Me diz logo o que tenho que fazer!

Se você já tem sólidos conhecimento sobre como o git funciona e não quer ler todas as explicações que dou, pode pular direto para o [resumo](#resumo).

## Motivação

Armazenar seus dotfiles em um repositório git (e deixá-lo open source em um repositório público) proporciona as seguintes vantagens:

- **backup "na nuvem"**: Todas aquelas suas customizações minuciosas e caprichadas que você fez estaram facilmente acessíveis de outras máquinas que você for começar usar.
- **aprendizado**: bisbilhotando 👀 os dotfiles de outras pessoas você aprende vários truquezinhos bacanas que facilitam sua vida. E se você deixa seus dotfiles open-source, a comunidade também vai aprender quais são os seus truques.

Caso queira bisbilhotar os meus dotfiles, aqui está: <https://github.com/meleu/.dotfiles>

## ⚠ ATENÇÃO

Se você salva qualquer tipo de informação sensível em seus dotfiles (ex.: senha, tokens, emails, etc.), tome cuidado para não tornar essas informações públicas.

Se por um acaso você acidentalmente commitou alguma informação sensível, recomendo uma lida nesse artigo do site do github: [Remover dados confidenciais do repositório](https://docs.github.com/pt/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository).



## Iniciando seu repositório `dotfiles`

### Crie seu repositório remoto

O primeiro passo é criar o seu repositório remoto. Eu recomendo fortemente que você deixe seu repositório público. Desta forma você pode compartilhar seu conhecimento com a comunidade e com os seus colegas, pode ensinar e aprender muito.

Vá no seu [github](https://github.com/), [gitlab](https://gitlab.com/), ou seu webservice de git favorito e crie um repositório chamado `dotfiles`.

**Observção**: crie seu diretório completamente vazio (sem README, sem LICENSE, etc.)


### Crie seu repositório local

Neste exemplo vou considerar que você criou o repositório no github. Portanto, se você está usando outros serviços, faça as adaptações necessárias nos comandos a seguir.

```bash
# só pra garantir...
username='COLOQUE_SEU_USERNAME_AQUI'

# vá para o home e crie o diretório "dotfiles"
cd ~
mkdir dotfiles

# entre no diretório e inicie um repositório git
cd ~/dotfiles
git init

# vá no github e pegue o endereço SSH do seu 
# repositório 'dotfiles'. Copie aquele endereço
# e configure o seu repo local.
git remote add origin git@github.com:${username}/dotfiles.git
```

### Segredo #1: customize seu `worktree`

Esse é o "pulo do gato" que vai tornar desnecessário que você instale qualquer software, crie links simbólicos, ou escreva scripts...

O que vamos fazer aqui é dizer para o nosso repositório local, que o diretório que ele tem que monitorar está em outro lugar.

Vamos fazer isso da seguinte forma:

```bash
# certifique-se que está no diretório do seu repo
cd ~/dotfiles

# diga ao git pra monitorar os arquivos no seu homedir
git config core.worktree '../../'
```

Motivo pelo qual estamos usando `../../`: pois esse caminho é relativo ao diretório `~/dotfiles/.git`. Portanto, se queremos apontar para o homedir, temos que subir dois níveis.

Isso fará com que o seu repo `dotfiles` passe a monitorar **TODOS os arquivos do seu homedir**. 😱

Para contornar isso, vamos para o segundo segredo...


### Segredo #2: ignore tudo!

Nosso arquivo `.gitignore` terá apenas um byte: `*`

```bash
# ignore a p*rr@ toda
echo '*' > ~/.gitignore
```

Note que o `.gitignore` PRECISA estar no seu homedir.

Agora que você está gitignorando tudo, basta você adicionar os arquivos que você quer usando a Força ~~Jedi~~ com `-f`:

```bash
# esses comandos precisam ser executados de
# dentro do diretório ~/dotfiles/
cd ~/dotfiles

# observações:
# - como estamos gitignorando tudo, use '-f'
# - como estamos em ~/dotfiles, precisamos do '../'
git add -f ../.bashrc
git add -f ../.vimrc
# etc...

# commit & push
git commit -m 'upando meus dotfiles'
git push
```

Lembre-se:

- para usar os comandos `git` no seu repositório, você precisa estar dentro do `~/dotfiles/`
- para adicionar arquivos novos, você precisa usar `git add -f`
- você precisará referenciar os arquivos levando em consideração que o `worktree` foi customizado. No nosso caso aqui, você terá que usar algo assim `git add -f ../${arquivo}`
- uma vez que o arquivo é adicionado, você não precisa mais usar o `-f` no `git add` futuramente.

E pronto! Seus dotfiles já estão num repositório git!


## Recuperando seus dotfiles em uma outra máquina

Para recuperar os nossos dotfiles de maneira limpa, vamos usar o terceiro segredo. Esse é o segredinho que torna desnecessárias todas aquelas ferramentas de gerenciar links simbólicos ou escrita de scripts para colocar os dotfiles nos lugares corretos.

### Segredo #3: git clone sem checkout

Por padrão, quando clonamos um repositório remoto, o git automaticamente já faz checkout dos arquivos e coloca no diretório criado ao clonar o repo remoto.

Mas como nós queremos fazer checkout dos arquivos diretamente no nosso homedir, vamos fazer o seguinte:
```bash
# certifique-se que está no homedir
cd ~

# use o --no-checkout para não fazer checkout
# dos arquivos dentro diretório que será criado
git clone --no-checkout git@github.com:${username}/dotfiles.git

# entre no repositório recém criado
cd ~/dotfiles

# diga ao git pra monitorar os arquivos no seu homedir
git config core.worktree '../../'

# agora fazemos o checkout dos arquivos
# diretamente no nosso homedir
# ⚠ atenção! ⚠ arquivos locais serão sobrescritos
git reset --hard origin/master
```

**ATENÇÃO**: ao usar o `git reset --hard`, seus dotfiles locais da máquina serão sobrescritos com o conteúdo que está vindo do seu repositório remoto. Geralmente é exatamente isso que queremos, mas de qualquer forma achei por bem ressaltar. 😇

E pronto! É só isso! Agora você já pode compartilhar seus dotfiles com seus colegas, amigos e com a comunidade em geral.


## Resumo

```bash
# crie um repo remoto chamado "dotfiles"
# (tanto faz se é github, gitlab, etc.)

# só pra garantir...
username='COLOQUE_SEU_USERNAME_AQUI'

# vá para o home e crie o diretório "dotfiles"
cd ~
mkdir dotfiles

# entre no diretório e inicie um repositório git
cd ~/dotfiles
git init

# vá no github e pegue o endereço SSH do seu 
# repositório 'dotfiles'. Copie aquele endereço
# e configure o seu repo local.
git remote add origin git@github.com:${username}/dotfiles.git

# diga ao git pra monitorar os arquivos no seu homedir
git config core.worktree '../../'

# ignore a p*rr@ toda
echo '*' > ~/.gitignore

# observações:
# - como estamos gitignorando tudo, use '-f'
# - como estamos em ~/dotfiles, precisamos do '../'
git add -f ../.bashrc
git add -f ../.vimrc
# etc...

# commit & push
git commit -m 'upando meus dotfiles'
git push
```

Recuperando os arquivos em outra máquina:
```bash
# certifique-se que está no homedir
cd ~

# use o --no-checkout para não fazer checkout
# dos arquivos dentro diretório que será criado
git clone --no-checkout git@github.com:${username}/dotfiles.git

# entre no repositório recém criado
cd ~/dotfiles

# diga ao git pra monitorar os arquivos no seu homedir
git config core.worktree '../../'

# agora fazemos o checkout dos arquivos
# diretamente no nosso homedir
# ⚠ atenção! ⚠ arquivos atuais serão sobrescritos
git reset --hard origin/master
```


## Bônus

Se você quer uma maneira rápida de sincronizar seus dotfiles com o repositório remoto, eis a função que eu uso:
```bash
# .files(): sincroniza seus dotfiles com o repo remoto.
# OBS.: o uso de (parênteses) no lugar de {chaves} é 
#       intencional e serve para que não seja necessário
#       fazer um `cd` pra voltar para o diretório anterior
.files() (
  local dotfilesDir="${HOME}/dotfiles"
  local gitStatus

  cd "${dotfilesDir}"

  gitStatus="$(git status --porcelain)"

  if [[ -z "${gitStatus}" ]]; then
    warn "dotfiles: nothing to update"
    return 0
  fi

  git add --all \
    && git status \
    && git commit -m "update $(date +'%Y-%m-%d %R'): ${gitStatus}" \
    && git pull --rebase \
    && git push
)
```

Eu coloco essa função no meu `~/.bash_functions`, que é chamado pelo meu `~/.bashrc`. Desta forma, quando quero sincronizar com o repo remoto eu simplesmente executo `.files` (isso mesmo, "ponto faious").

Se quer mais inspiração. Confira meus dotfiles: <https://github.com/meleu/.dotfiles>


## Fontes

- gitignore "a p\*rr@ toda": <https://drewdevault.com/2019/12/30/dotfiles.html>
- truquezinho do worktree: <https://www.wangzerui.com/2017/03/06/using-git-to-manage-system-configuration-files/>
- vários artigos sobre gerenciamento de dotfiles: <https://dotfiles.github.io/>
- Documentação do github sobre como [Remover dados confidenciais do repositório](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository)
