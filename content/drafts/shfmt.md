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

Neste artigo vamos conhecer o `shfmt`, uma ferramenta que vai te ajudar a manter seu código com uma formatação consistente, e também para tornar legível algum código de outra pessoa que você queira examinar.

Veremos aqui o que é e como instalar o `shfmt`, alguns exemplos de uso e como configurar o seu editor (VSCode e vim) para formatar seu código assim que você salvar.

## Demonstração

Só pra deixar claro, quando eu digo formatação estou me referindo à indentação, declaração de funções, quebra de linhas de comandos longos... Enfim, coisas extremamente básicas mas que influenciam bastante na legibilidade do seu código.

Veja esse exemplo ilustrativo:
```bash
#!/bin/bash


 echo   'linha com formatação ruim'
   echo 'outra linha com indentação feiosa'


echo 'você pode não estar vendo...'
echo 'mas esta linha tem espaços em branco no final > '            

func(){ if [[ $# -eq 0 ]];then echo "sem argumentos">&2;return 1;fi; echo "Hello, $1"; }

main() {
  echo 'mais espaços desnecessários'
              echo 'indentação exagerada'
 }

main       "$@"
```

Com `shfmt` conseguimos converter aquilo 👆, nisso aqui 👇

```bash
#!/bin/bash

echo 'linha com formatação ruim'
echo 'outra linha com indentação feiosa'

echo 'você pode não estar vendo...'
echo 'mas esta linha tem espaços em branco no final > '

transfer() {
  if [[ $# -eq 0 ]]; then
    echo "sem argumentos" >&2
    return 1
  fi
  echo "Hello, $1"
}

main() {
  echo 'mais espaços desnecessários'
  echo 'indentação exagerada'
}

main "$@"
```


## Contexto pessoal

Desde que conheci o `shfmt` eu o uso para manter o meu próprio código com uma formatação consistente e alinhada com o meu *coding style*. Mas aonde ele me deixa mais feliz é quando preciso trabalhar num código originalmente escrito por uma pessoa sem experiência com programação shell.

Sabe como é né... O shell é uma linguagem muito poderosa e permissiva. A barreira de entrada é bem pequena e devido ao poder do shell você consegue rapidamente resolver o que você precisa resolver. Por um lado isso é bom (afinal, resolver problemas é uma coisa boa!), mas em contrapartida acaba gerando muito código bagunçado. Difícil de ler e de manter.

Pra agravar o cenário, geralmente quem escreve shell scripts, são SysAdmins. Historicamente um SysAdmin é uma pessoa treinada para manter o sistema estável, bem configurado e seguro. Focar em aprender técnicas avançadas de programação, *Clean Code*, e coisas do gênero normalmente não é uma prioridade para um SysAdmin.

Como resultado, é bastante comum encontrar por aí código bem difícil de ler/analisar/atualizar/refatorar.

Quando me deparo com um cenário desses, a primeiríssima coisa que faço é rodar o `shfmt` pra tornar o código minimamente legível (e na sequência eu rodo [shellcheck](/shellcheck)).


## Instalação

No [README do projeto](https://github.com/mvdan/sh) não existe uma instrução muito clara de como instalar o `shfmt`. Mas nós vamos resolver isso bem rápido...

Se você usa uma distro baseada em Debian/Ubuntu, verifique se o programa está disponível para você com `sudo apt install shfmt`.

Se sua distro possui Snap, você pode instalar com `snap install shfmt`.

Uma outra maneira é simplesmente baixar o binário na [página de releases do projeto](https://github.com/mvdan/sh/releases).

Eu particularmente gosto de instalar essas ferramentinhas que tem um desenvolvimento bem ativo, usando o [asdf-vm](https://asdf-vm.com), pois assim consigo facilmente experimentar versões mais atuais das ferramentas sem ficar instalando-as globalmente no meu sistema.

Não está no escopo desse artigo falar muito sobre o asdf-vm (fica pra um artigo futuro), mas se você tem ele aí, aqui está a receitinha de bolo pra instalar o `shfmt`:

```bash
# disponibilizando o shfmt para ser instalado via asdf
asdf plugin add shfmt

# instalando a versão mais recente
asdf install shfmt latest

# definindo a versão que será disponibilizada no PATH
asdf global shfmt latest

# conferindo qual foi a versão instalada
shfmt --version
```

> Observação: no momento da escrita desse artigo a versão atual do shfmt é 3.5.0


## Opções de formatação

A primeira opção que eu gostaria de mostrar, é o `-d`/`--diff`. Simplesmente por ser uma opção onde a gente consegue ver um "diff" entre o nosso código original e o resultado após passar pelo `shfmt`. Isso vai te mostrar claramente o que mudou.

Vejamos com esse codiguinho aqui:
```bash
#!/bin/bash
hello() {
echo "Hello, $1"
}
hello    "$@"
```

Se a gente rodar o `shfmt -d hello.sh`, obteremos o seguinte diff (obs.: o arquivo original **NÃO** será alterado):
```diff
--- hello.sh.orig
+++ hello.sh
@@ -1,6 +1,6 @@
 #!/bin/bash
 hello() {
-echo "Hello, $1"
+        echo "Hello, $1"
 }
 
-hello    "$@"
+hello "$@"
```

Podemos observar que o shfmt colocou uma indentação no `echo` dentro da função `hello`, e também tirou os espaços desnecessários na chamada da função.

A formatação default ficou bacaninha, né? Mas vamos ver como customizar para um estilo que mais nos agrade.

Por exemplo, eu não curti muito aquela indentação com tab. Nos meus projetos eu costumo utilizar 2 espaços para indentação.

Para isso vamos utilizar a opção `-i`/`--indent` (junto com o `-d` pra vermos o diff)
```bash
shfmt -d -i 2 hello.sh
```

No diff, a gente vai observar o seguinte:
```diff
--- hello.sh.orig
+++ hello.sh
@@ -1,6 +1,6 @@
 #!/bin/bash
 hello() {
-echo "Hello, $1"
+  echo "Hello, $1"
 }
 
-hello    "$@"
+hello "$@"
```

Ah... Agora sim! Indentação com 2 espaços. 🙂

Não vou ficar demonstrando aqui o que cada opção faz (afinal, eu te mostrei o macetinho do `--diff` exatamente para você fazer suas experimentações aí). Mas vou listar aqui as opções que eu costumo usar:

- `-bn`, `--binary-next-line`: para que os operadores binários `&&`, `||` e até o `|` sejam colocados no começo da próxima linha.
```bash
# antes do 'shfmt -bn'
comando1 && 
  comando2 &&
  comando3

# depois do 'shfmt -bn'
comando1 \
  && comando2
  && comando3
```

- `-ci`, `--ci-indent`: para adicionar indentação em cada opção dentro de um `case`.
```bash
# antes do 'shfmt -ci'
case "${extension}" in
.txt)
  echo "texto"
  ;;
.mp3)
  echo "música"
  ;;
.md)
  echo "markdown"
  ;;
*)
  echo "outro formato"
  ;;
esac

# depois do 'shfmt -ci'
case "${extension}" in
  .txt)
    echo "texto"
    ;;
  .mp3)
    echo "música"
    ;;
  .md)
    echo "markdown"
    ;;
  *)
    echo "outro formato"
    ;;
esac
```

- `-sr`, `--space-redirects`: operadores de redirecionamento serão seguidos por um espaço.
```bash
# antes do 'shfmt -sr'
grep meleu /etc/passwd >myInfo.txt

# depois do 'shfmt -sr'
grep meleu /etc/passwd > myInfo.txt
```

As outras opções de formatação eu não costumo usar, mas recomendo que você experimente um pouco e veja se faz sentido pra você.

## Pontos de atenção!

Apesar de ser uma ferramenta extremamente útil, existem alguns casos onde precisamos ficar atento pois o `shfmt` não é capaz de lidar.

Isso está listado no [README do projeto](https://github.com/mvdan/sh#caveats) e nós vamos dar uma analisada aqui.

### Índices de arrays associativos precisam de aspas

Quando estiver usando um array associativo, o `shfmt` vai precisar que você coloque os índices entre aspas. Caso contrário ele terá problemas para formatar índices com espaços ou sinais aritméticos.

Imagine um array desse tipo aqui:
```bash
$ # para o bash isso aqui é perfeitamente válido
$ declare -A array=([indice-1]=um [indice-2]=dois)

$ # passando esse código para o shfmt
$ echo 'declare -A array=([indice-1]=um [indice-2]=dois)' \
  | shfmt
declare -A array=([indice - 1]=um [indice - 2]=dois)
$ # espaços indevidos aqui 👆    e aqui  👆
```

O problema é que o parser do `shfmt` acredita que ali dentro daqueles `[colchetes]` tem uma expressão aritmética, e aí tenta formatar essa expressão pra ficar bonitinha.

Inclusive, se o nosso índice tiver espaços, o shfmt vai reclamar que não estamos passando uma expressão válida:
```bash
$ # expressão válida no bash
$ declare -A array=([indice 1]=um)

$ # mas inválida para o shfmt
$ echo 'declare -A array=([indice 1]=um)' | shfmt
<standard input>:1:27: not a valid arithmetic operator: 1
```

**Solução**: basta usarmos aspas que o parser do `shfmt` vai saber que estamos passando uma string (e não uma expressão aritmética):
```bash
$ echo 'declare -A array=(["indice 1"]=um ["indice-2"]=dois)' \
  | shfmt
declare -A array=(["indice 1"]=um ["indice-2"]=dois)

```


### Ambiguidade entre `$((` e `((`

Recapitulando aqui 3 *features* do bash:

1. `$((expressão))`: executa uma expressão aritmética.
    - ex.: `echo "Você nasceu em $((thisYear - age))"`
2. `$(comando)`: executa o `comando` e devolve o output daquele comando.
    - ex.: `echo "Você está no diretório $(pwd)"`
3. `(comando)`: executa um `comando` em subshell.
    - ex.: `(cd /tmp; rm -f files*)`

Se combinarmos a técnica do item 2 com a do item 3, podemos ter uma situação do tipo
```bash
# isso é perfeitamente válido em bash
echo "output dos comandos: $((comando1); (comando2))"
# o shfmt vai confundir isso 👆
```

Como podemos notar, aquele comecinho ali com `$((` vai confundir o `shfmt`.
```txt
$ echo '$((comando1); (comando2))' | shfmt
<standard input>:1:1: reached ) without matching $(( with ))
```

De fato, essa notação é apontada no próprio [padrão POSIX](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html#tag_18_06_03) como uma notação ambígua. E recomenda que seja usado um espaço para separar o `$(` e o `(`.

```txt
$ # com espaços, fica deboas 👍
$ echo '$( (comando1); (comando2) )' | shfmt -i 2
$(
  (comando1)
  (comando2)
)

```


### Declarações "exóticas" não são suportadas

Em bash você consegue declarar duas variáveis e atribuir o mesmo valor a ambas em uma única linha, usando essa notação aqui:
```bash
declare {var1,var2}=valor
```

Essa notação não é suportada pelo `shfmt`, pois ele vai achar que você está querendo declarar uma variável chamada `{var1,var2}`:
```txt
$ echo 'declare {var1,var2}=valor' | shfmt
<standard input>:1:9: invalid var name
```

No meu caso isso não se torna exatamente um problema, pois eu não uso nem jamais usaria tal notação (pois acredito que o código deve ser o mais claro e explícito possível).


### A opção `--binary-next-line` não se aplica a `[[ testes ]]`

Todos os "probleminhas" listados acima são perfeitamente aceitáveis pra mim. Tenho pra mim que eles até encorajam boas práticas.

Mas esse aqui é o único "probleminha" do `shfmt` que me incomoda (mas nem por isso parei de usá-lo em TODOS os meus scripts).

Esse problema não está na [lista de *caveats* do README](https://github.com/mvdan/sh#caveats), mas existe [uma issue aberta sobre isso](https://github.com/mvdan/sh/issues/813).

O lance é que a opção `-bn`,`--binary-next-line` não se aplica às expressões dentro dos `[[ colchetes ]]`.



Expressões longas como essa não são muito comuns nos meus códigos, mas de vez em quando aparecem.










## Formatando um código "confuso"

aplicar nesse script aqui do transfer.sh
```bash
transfer(){ if [ $# -eq 0 ];then echo "No arguments specified.\nUsage:\n transfer <file|directory>\n ... | transfer <file_name>">&2;return 1;fi;if tty -s;then file="$1";file_name=$(basename "$file");if [ ! -e "$file" ];then echo "$file: No such file or directory">&2;return 1;fi;if [ -d "$file" ];then file_name="$file_name.zip" ,;(cd "$file"&&zip -r -q - .)|curl --progress-bar --upload-file "-" "https://transfer.sh/$file_name"|tee /dev/null,;else cat "$file"|curl --progress-bar --upload-file "-" "https://transfer.sh/$file_name"|tee /dev/null;fi;else file_name=$1;curl --progress-bar --upload-file "-" "https://transfer.sh/$file_name"|tee /dev/null;fi;}
```



## Instale shfmt no seu editor

### VSCode

### vim

### bônus: EditorConfig


