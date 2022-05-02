---
title: "Use shellcheck e encontre bugs no seu código antes mesmo de executá-lo."
description: >
  "Com o shellcheck você vai se livrar de muitos aborrecimentos imprevistos com bugs que você jamais conseguiria prever!"
tags:
  - boas-praticas
  - ferramentas
date: 2022-05-02T15:11:31-03:00
cover:
  image: "img/shellcheck.png"
  alt: shellcheck
draft: true
---

Nós amamos programar, não é mesmo (se você está lendo esse blog, eu imagino que sim). Mas convenhamos, escrever um código e ficar continuamente alternando entre escrever-salvar-testar... Tem hora que enche o saco!

A ferramenta que vou mostrar neste artigo vai nos ajudar a minimizar essas trocas de contexto, que tanto nos incomodam e quebram o ritmo da nossa escrita.

O shellcheck é um *linter* para shell scripts. Um *linter*, segundo o [wikipedia](https://en.wikipedia.org/wiki/Lint_(software)) é:

> uma ferramenta de análise estática de código usada para alertar erros de programação, bugs, erros estilísticos e construções suspeitas.

Explicando de maneira mais simples: é uma ferramenta que vai análisar o seu código em busca de problemas e vai te alertar sobre o que ele encontrar.


## Um contexto pessoal

O shellcheck é mais uma daquelas ferramentas que mudam a vida das pessoas. Mudou a minha vida e tenho certeza que mudará a sua, caso programe em shell com frequência.

Eu sei que isso soa muito sensacionalista e exagerado... mas é sério, não consigo descrever o shellcheck de outra forma. Vou explicar o motivo explicando um pouco do meu trabalho

Parte do meu dia a dia é escrevendo/mantendo esteiras de integração contínua escritas em bash. O ciclo escrever-salvar-testar não é tão simples quanto "salvar o arquivo, ir no terminal e executar o script". No meu trabalho esse ciclo geralmente envolve:

1. fazer o commit das minhas alterações para um repositório git
2. ir em outro repositório disparar a pipeline que vai executar o meu código
3. esperar a pipeline instanciar o container que executará o meu código
4. olhar o log em busca de erros

Imagina esse ciclo pra você ver no log que você cometeu um erro de digitação num nome de variável... 🤦

De tão deprimente as vezes dá vontade de ficar trancado no banheiro, chorando encolhido e ouvindo Linkin Park.

Espero que essa historinha sirva pra explicar o motivo de tantos artigos deste blog terem esse viés de "antecipe os problemas antes de colocar seu código em produção". E também o por que de eu falar coisas como "isso mudou minha vida"...

Acredite em mim, não é sensacionalismo. 😇



## Conhecendo o shellcheck

Você pode ver o shellcheck em ação agora mesmo!

Copie esse código bem bobinho aqui (selecione e dê `ctrl-c`):

```bash
hello() {
  echo Hello $name
}

hello $@
```

Notamos que é um código bash perfeitamente válido. Não tem erro de sintaxe algum. Mas agora vamos colar esse código nesse site aqui: <https://www.shellcheck.net/>

Ao colar o código, você verá na parte de baixo da tela algo assim:

![](Pasted%20image%2020220502160928.png)

Observe que o shellcheck está nos apontando vários problemas que temos no nosso script.

- Linha 1:
    - erro: não atribuímos um shebang ao nosso script (num [artigo anterior](/shebang) aprendemos por que isso nunca deve acontecer no seu script)
- Linha 2:
    - alerta: a variável `$name` está sendo referenciada mas não foi atribuído valor algum a ela.
    - informação: em volta de variáveis devemos usar [aspas duplas sempre](/aspas-sempre)
- Linha 5:
    - erro: mais uma vez esquecemos das aspas...


### Instalação

Obviamente que você não ficar copiando e colando seu código numa página da web o tempo todo quando tiver codando.

Vamos instalar o shellcheck na nossa máquina.

Se você está usando uma distro baseada no Debian:
```
sudo apt-get install shellcheck
```

Outros métodos também podem ser encontrados [no README do shellcheck no github](https://github.com/koalaman/shellcheck#installing) (eu particularmente instalei com o [asdf-vm](https://asdf-vm.com/))

### Executando o shellcheck

Não tem mistério, basta fazer `shellcheck script.sh`.

Vamos ver com mais um script ilustrativo:

```bash
#!/usr/bin/env bash
# remove-spaces.sh
#
# remove espaços do nome dos arquivos .mp3

directory="$1"

for file in "$directory/*.mp3"; do
  mv -v "${file}" "${file// /_}"
done
```

Mais uma vez, um script perfeitamente válido. O bash não vai reclamar de nada ao tentar interpretar esse script.

Vamos então executar o shellcheck e ver o que ele vai dizer:

```
$ shellcheck remove-spaces.sh

In remove-spaces.sh line 6:
  for file in "$directory/*.mp3"; do
              ^----------------^ SC2066 (error): Since you double quoted this, it will not word split, and the loop will only run once.

For more information:
  https://www.shellcheck.net/wiki/SC2066 -- Since you double quoted this, it ...

```

Eita! Parece que exageramos no alcance das aspas duplas ali...

Indo direto ao ponto, o que o shellcheck está tentando nos dizer é que como o `*` está dentro das aspas, aquele padrão não vai expandir para o nome dos arquivos `.mp3` que temos naquele diretório. Ele tentará procurar um arquivo chamado literalmente `*.mp3` (asterisco ponto mp3).

No README do shellcheck tem uma [galeria de código ruim](https://github.com/koalaman/shellcheck#gallery-of-bad-code), com uma lista de códigos problemáticos que muitas vezes podem ser construções válidas para o shell, mas que não é exatamente o que você quer.


### ShellCheck wiki

Uma das coisas mais fantásticas do shellcheck não é nem o fato dele ficar jogando na nossa cara que nosso script está todo bugado. O mais legal é que ele nos diz **por que** o código está bugado e também **como** corrigir.

Observe a última linha daquele teste que fizemos:

```
For more information:
  https://www.shellcheck.net/wiki/SC2066 -- Since you double quoted this, it ...
```

Se dermos uma olhadinha [naquele link](https://www.shellcheck.net/wiki/SC2066), veremos o motivo pelo qual essa construção é problemática e também o que devemos fazer para obter o resultado que queremos com um código mais robusto.

No nosso exemplo aqui, bastaria usar `"$directory"/*.mp3` (fechar as aspas logo após o nome da variável).

Eu *preciso* enfatizar que o shellcheck wiki é uma fonte valiosíssima de conhecimento sobre shell scripting, mas primeiro vamos falar rapidinho sobre o que fazer se você discordar do shellcheck

### E se eu discordar do shellcheck?

Podem haver situações onde não concordamos com o diagnóstico do shellcheck e queremos ignorar certas coisas que ele considera problemática.

Um exemplo clássico pra mim é essa estratégia que eu sempre uso para saber em que diretório o meu script está armazenado:

```bash
#!/usr/bin/env bash

# maneira "infalível" de saber o diretório do seu script
export SCRIPTS_DIR="$(
  cd "$(dirname -- "${BASH_SOURCE[0]}")" && pwd
)"
```

Agora vejamos o que o shellcheck tem a nos dizer:

```
In clean-tmp.sh line 4:
export SCRIPTS_DIR="$(
       ^---------^ SC2155 (warning): Declare and assign separately to avoid masking return values.

For more information:
  https://www.shellcheck.net/wiki/SC2155 -- Declare and assign separately to ...
```

Ele está falando que eu deveria declarar `SCRIPTS_DIR` primeiro, e depois atribuir um valor.

Este alerta serve para chamar a atenção para situações como essa:

```
# o export finaliza com sucesso
# independente do que está dentro do $()
$ export var=$(comando invalido) \
  && echo sucesso \
  || echo falha
comando: command not found
sucesso
```

OK, isso realmente pode ser problemático. Mas eu tenho certeza que no caso específico daquele meu código eu não terei problemas. Portanto eu quero que o shellcheck ignore essa "regra".

Como podemos ver no output do shellcheck, a regra é identificada pelo ID `SC2155`. Vamos então desabilitá-la.

Uma maneira de fazer isso é ir na linha acima do problema e adicionar um comentário assim:
```bash
# maneira "infalível" de saber o diretório do seu script
# shellcheck disable=SC2155
export SCRIPTS_DIR="$(
  cd "$(dirname -- "${BASH_SOURCE[0]}")" && pwd
)"
```

Pronto! Agora se você rodar o shellcheck de novo ele não vai mais reclamar disso.

Outras maneiras de ignorar essas regras podem ser vistas [nesta página](https://github.com/koalaman/shellcheck/wiki/Ignore). As que eu uso na prática geralmente são essas 3:

1. inserindo um comentário pra desabilitar a regra na linha acima do código problemático.
2. criando um arquivo `.shellcheckrc` na raiz do projeto (requer shellcheck 0.7+)
3. inserindo um comentário pra desabilitar a regra no topo do arquivo (logo após o shebang).


### Coisas que aprendi com o shellcheck wiki

Quero voltar a falar do wiki para reforçar o quanto ele é valioso! O conteúdo dele é muito bem escrito, encoraja boas práticas e vai te mostrar várias armadilhas presentes quando programamos em shell.

Garanto que você aprenderá muito simplesmente ao rodar o shellcheck com algum script antigo que você escreveu aí e ler as páginas que ele recomenda que você leia.

Vou exemplificar aqui 3 coisinhas que eu aprendi e que provavelmente passariam despercebidas por anos até que algum bug sério me fizesse aprender da pior maneira.

Isso não é nem a ponta do iceberg. Mas serve pra ilustrar como que podemos evitar problemas que demorariam horas para encontrarmos a raiz.

Para isso, vamos a mais um script ilustrativo:

```bash
#!/usr/bin/env bash
# clean-tmp.sh

# maneira "infalível" de saber o diretório do seu script
# shellcheck disable=SC2155
export SCRIPTS_DIR="$(
  cd "$(dirname -- "${BASH_SOURCE[0]}")" && pwd
)"

# função (meramente ilustrativa)
# para limpar arquivos temporários
cleanTmpDir() {
  cd "${SCRIPTS_DIR}/tmp"
  rm *
  cd ..
}

main() {
  # ... considere que algo útil aconteceu aqui...
  echo "limpando arquivos temporários"
  cleanTmpDir
}

main "$@"
```

Vejamos o que o shellcheck diz:

```
$ shellcheck clean-tmp.sh 

In clean-tmp.sh line 13:
  cd "${SCRIPTS_DIR}/tmp"
  ^---------------------^ SC2164 (warning): Use 'cd ... || exit' or 'cd ... || return' in case cd fails.

Did you mean: 
  cd "${SCRIPTS_DIR}/tmp" || exit


In clean-tmp.sh line 14:
  rm *
     ^-- SC2035 (info): Use ./*glob* or -- *glob* so names with dashes won't become options.


In clean-tmp.sh line 15:
  cd ..
  ^---^ SC2103 (info): Use a ( subshell ) to avoid having to cd back.

For more information:
  https://www.shellcheck.net/wiki/SC2164 -- Use 'cd ... || exit' or 'cd ... |...
  https://www.shellcheck.net/wiki/SC2035 -- Use ./*glob* or -- *glob* so name...
  https://www.shellcheck.net/wiki/SC2103 -- Use a ( subshell ) to avoid havin...
```

Agora vamos analisar estes alertas

#### Sempre certifique-se que um `cd` terminou com sucesso

```
In clean-tmp.sh line 13:
  cd "${SCRIPTS_DIR}/tmp"
  ^---------------------^ SC2164 (warning): Use 'cd ... || exit' or 'cd ... || return' in case cd fails.

Did you mean: 
  cd "${SCRIPTS_DIR}/tmp" || exit
```

Após ler [o wiki SC2164](https://www.shellcheck.net/wiki/SC2164) percebi que ele está me alertando que eu deveria checar se o `cd` foi bem sucedido.

O motivo de uma coisa aparentemente tão boba ser levado é sério pode ser visto na nossa função `cleanTmpDir`. Lá temos um `cd` e na sequência temos um `rm -rf`.

Se o `cd` por acaso não for bem sucedido, provavelmente faremos um `rm -rf` num lugar onde não queremos.

Mas aqui tem uma coisa... Eu sou meio chatinho com a ~~beleza~~ legibilidade do meu código, e não quero ficar enchendo meu script de `|| return` depois de cada `cd`...

Felizmente temos uma solução bem limpa quanto a isso. Basta seguir a recomendação que fiz lá no artigo para deixar o seu [bash-rigoroso](/bash-rigoroso) e adicionar um `set -e` no início do seu script.

Isso fará com que o script se encerre sempre que um comando terminar com uma falha sem tratamento.

E o mais bacana, é que se colocar um `set -e` no nosso script, o shellcheck já entende que o problema foi resolvido e nem vai reportá-lo mais.


#### Sempre especifique o caminho relativo dos arquivos referenciados por `*coringas*`

```
In clean-tmp.sh line 14:
  rm *
     ^-- SC2035 (info): Use ./*glob* or -- *glob* so names with dashes won't become options.
```

Ao ler [o wiki SC2035](https://www.shellcheck.net/wiki/SC2035) eu percebi que o perigo aqui é que se um arquivo com um nome, por exemplo, `-rf` isso será considerado como uma opção do `rm`, e não como um nome de arquivo.

Obviamente, isso poderá trazer consequências indesejadas e bastante desagradáveis.

Agora pare um pouquinho pra pensar em que momento que você iria se dar conta desse potencial problema?

Bem... Eu não sou nenhum gênio, e fiquei muito feliz de receber esse alerta sem sentir dor alguma. 🙂

A solução que eu usaria aqui é simplesmente colocar o caminho relativo: `./`. Desta forma aquela linha ficaria assim:
```bash
  rm ./*
```

#### Se precisar usar `cd`, faça isso num `(subshel)`

```
In clean-tmp.sh line 15:
  cd ..
  ^---^ SC2103 (info): Use a ( subshell ) to avoid having to cd back.
```

#### Ao usar `rm -rf ${dir}/*`, evite qualquer possibilidade de `$dir` expandir para `/*`



```
In clean-tmp.sh line 15:
  cd ..
  ^---^ SC2103 (info): Use a ( subshell ) to avoid having to cd back.
```




## Indo além...

Nesta seção coloco um conteúdo um pouquinho mais avançado mostrando algumas utilizações que faço do shellcheck.

Se você está apenas iniciando no shellscript, não se preocupe se não entender tudo...


### integrando shellcheck com o vim usando Syntastic



### impedindo commit de código bugado com git hook


### uma pipeline para checagem de código

