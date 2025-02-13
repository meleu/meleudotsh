---
title: Habilitando auto-suggestions no bash
description: Deixe o bash completar seu comando com uma sugestão baseada no seu histórico.
tags:
  - bash
  - configuracoes
date: 2025-02-13T11:00:00-03:00
cover:
  image: "img/blesh-cover.png"
  alt: blesh
---

O auto-suggestions é muito conhecido do pessoal que usa o [fish](https://fishshell.com/) ou [zsh](https://www.zsh.org/), e é algo que me ~~faz~~ fazia muita falta no bash. Trata-se de um recurso que, durante a digitação de um comando, já te mostra uma sugestão baseada no seu histórico de comandos utilizados.

Exemplo, se eu alguma vez já digitei o comando `cd ~/foo/bar`, só de digitar o `cd ~/f`, o auto-suggestion já vai me sugerir o resto do comando, assim:

![blesh auto-suggestion](/img/blesh-autosuggestion.png)

Esse simples recurso foi o que me fez passar os últimos 2 anos usando o zsh, depois de mais de duas décadas de bash. Neste período houveram muitos momentos onde eu esbarrava em algum problema de incompatibilidade que me deixava insatisfeito. Mas ainda assim eu permanecia no zsh só por causa do auto-suggestion, pois é algo que me faz ganhar muito tempo no dia a dia.

Acontece que recentemente descobri que existe uma maneira de ter esse recurso no bash e finalmente voltei para o meu velho amigo! 🫂

## Conheça o ble.sh

A única maneira que encontrei de ter auto-suggestion no bash é através do [ble.sh](https://github.com/akinomyoga/ble.sh) (eu pronuncio como _bléshi_).

Na real o ble.sh faz muito mais coisas que o auto-suggestions (aliás, ele chama esse recurso de auto-complete). Uma rápida lida no README do projeto lista recursos como:

- **Syntax highlighting**.
- **Auto-complete melhorado**. Além das sugestões baseadas no seu histórico de comandos, esse auto-complete também sugere possíveis candidatos ao comando que você está prestes a digitar (por exemplo, nomes de arquivo).
- **Edição em modo vim**.
- Outras features...

Caso essas features te empolguem, sugiro uma lida cuidadosa do README e no wiki do projeto. No meu caso eu estou interessado em apenas uma única coisa: o **auto-complete baseado no meu histórico**. Apenas isso e nada mais.

### Instalando o ble.sh

Existem instruções sobre como instalar tanto no [README](https://github.com/akinomyoga/ble.sh?tab=readme-ov-file#11-build-from-source) quanto [Wiki](https://github.com/akinomyoga/ble.sh/wiki/Manual-%C2%A71-Introduction#11-install--update). E uma coisa que pode ser confusa é que eles dão muitas opções de diferentes maneiras de instalar (acredito que todas elas funcionam, mas esse excesso de informação gera confusão).

Vou falar aqui como que **eu** fiz:

```bash
# vou num diretório onde quero clonar o repositório
git clone --recursive --depth 1 --shallow-submodules https://github.com/akinomyoga/ble.sh.git

# nem precisa entrar no diretório 'ble.sh'
# basta executar isso:
make -C ble.sh install PREFIX=~/.local
```

Esses 👆 comandos serão suficientes para colocar os arquivos do ble.sh em subdiretórios do `~/.local/`.

Agora vou no meu `~/.bashrc`  e adiciono algumas linhas:

```bash
# ~/.bashrc

# 🗣 OBS!: no topo do meu ~/.bashrc eu verifico se
# estou em uma sessão interativa. Se não tiver, já saio.
[[ "$-" != *i* ]] && return

# ... outras configurações aqui...

# Isso 👇 aqui foi adicionado.

# iniciando o ble.sh
# https://github.com/akinomyoga/ble.sh
source ~/.local/share/blesh/ble.sh --noattach
[[ ! ${BLE_VERSION-} ]] || ble-attach
```

Agora basta um `exec bash` que o seu ambiente shell será recarregado.

Use o seu shell um pouco e observe que o ble.sh já está em ação, te oferecendo vários recursos.

### Configurando o ble.sh apenas para auto-suggestion

Como eu disse, apesar do ble.sh ter muitos recursos, a única coisa que eu estou interessado é o **auto-complete baseado no meu histórico**. Como esse tipo de coisa deve ser muito comum, o mantenedor do projeto já deixou no README uma seção chamada [Disable features](https://github.com/akinomyoga/ble.sh#22-disable-features), mostrando como desabilitar as features que vem habilitadas por padrão.

Um bom lugar para colocar tais configurações é o `~/.blerc`. Aqui eu mostro como eu deixei para o meu caso de uso (observe que o que estamos fazendo é atribuindo valores vazios à algumas variáveis):

```bash
# dicas do README
# https://github.com/akinomyoga/ble.sh?tab=readme-ov-file#22-desabilita-features
#############################################################################
# desabilita syntax highlighting
bleopt highlight_syntax=

# desabilita highlighting baseado em filenames
bleopt highlight_filename=

# desabilita highlighting baseado em tipos de variável
bleopt highlight_variable=

# desabilita ambiguous completion
bleopt complete_ambiguous=

# desabilita menu-complete com TAB
bleopt complete_menu_complete=

# desabilita menu filtering (ex.: sugerir arquivos)
bleopt complete_menu_filter=

# desabilita marcador de EOF (ex.: "[ble: EOF]")
bleopt prompt_eol_mark=''

# desabilita marcador de erro (ex.: "[ble: exit %d]")
bleopt exec_errexit_mark=

# desabilita marcador de exit (ex.: "[ble: exit]")
bleopt exec_exit_mark=

# desabilita outros marcadores (ex.: "[ble: ...]")
bleopt edit_marker=
bleopt edit_marker_error=
```

Além destas configs, para desabilitar as features que eu não preciso/quero. Eu também adiciono essa config pra deixar o meu auto-suggestion (aqui chamado de auto-complete) com uma cor um pouco mais sútil:

```bash
# deixa o auto-complete com uma cor mais sútil
ble-face auto_complete='fg=240,underline,italic'
```

## Conclusão

Depois de mais de 20 anos usando bash eu fiquei os últimos 2 anos usando zsh... (será coincidência que eu tenha ficado esse mesmo tempo sem postar aqui no meleu.sh? 🤔)

A única coisa que me fazia usar o zsh era o auto-suggestion, que a meu ver é uma _killer feature_ que me ajuda muito no dia a dia.

Agora, graças ao auto-complete do ble.sh estou de volta pro meu velho amigo **bash**! 🥰
