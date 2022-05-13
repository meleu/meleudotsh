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

Neste artigo vamos conhecer o `shfmt`. Uma ferramenta que vai te ajudar a manter seu código com uma formatação consistente, e também para tornar legível algum código de outra pessoa que você queira examinar.

Veremos aqui o que é e como instalar o `shfmt`, alguns exemplos de uso e como configurar o seu editor (VSCode e vim) para formatar seu código assim que você salvar.

## Demonstração

Só pra deixar claro, quando eu digo formatação estou me referindo à indentação, declaração de funções, quebra de linhas de comandos longos... Enfim, coisas extremamente básicas mas que influenciam bastante na legibilidade do seu código.

O `shfmt` converte código como esse:
```bash
#!/bin/bash


 echo 'primeira linha com formatação bugada'
  echo 'linha 2 com indentação feiosa'


echo 'você pode não estar vendo...'
echo 'mas esta linha tem espaços em branco no final > '            

func(){ if [[ $# -eq 0 ]];then echo "sem argumentos">&2;return 1;fi; echo "Hello, $1"; }

main() {
  echo 'mais espaços desnecessários'
              echo 'indentação exagerada'
 }

main       "$@"
```

em uma versão bem mais agradável de se ler, como essa aqui:
```bash
#!/bin/bash

echo 'primeira linha com formatação bugada'
echo 'linha 2 com indentação feiosa'

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

Apesar de eu ser um grande fã do `shfmt`, tem uma coisa que me deixa intrigado: no [README do projeto](https://github.com/mvdan/sh) não existe uma instrução clara de como instalar. Mas nós vamos resolver isso bem rápido...

Uma das maneiras é simplesmente baixar o binário na [página de releases do projeto](https://github.com/mvdan/sh/releases).

Se você usa uma distro baseada em Debian/Ubuntu, verifique se o programa está disponível para você com `sudo apt install shfmt`.

Se sua distro possui Snap, você pode instalar com `snap install shfmt`.

Eu particularmente gosto de instalar essas ferramentinhas com desenvolvimento bem ativo, usando o [asdf-vm](https://asdf-vm.com), pois assim consigo facilmente brincar com versões mais atuais das ferramentas sem ficar instalando-as globalmente no meu sistema.

Não está no escopo desse artigo falar muito sobre o asdf-vm (isso fica pra um artigo futuro), mas vou passar a receitinha de bolo pra instalar o `shfmt`:

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

A primeira opção que eu gostaria de mostrar, é o `-d`/`--diff`. Simplesmente por ser uma opção onde a gente consegue ver um "diff" entre o nosso código original e o resultado após passar pelo `shfmt`.

Desta forma, quando formos testando as outras opções de formatação a gente vai usar junto com o `-d` e perceberemos claramente o que mudou.

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

🤔 Uhm... Vejamos...

O shfmt colocou uma indentação no `echo` dentro da função `hello`, e também tirou os espaços desnecessários na chamada da função.

Bacaninha né? Mas ainda assim tem como melhorar.

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
txt) echo "texto" ;;
mp3) echo "música" ;;
md) echo "markdown" ;;
*) echo "outro formato" ;;
esac

# depois do 'shfmt -ci'
case "${extension}" in
  txt) echo "texto" ;;
  mp3) echo "música" ;;
  md) echo "markdown" ;;
  *) echo "outro formato" ;;
esac
```

- `-sr`, `--space-redirects`: operadores de redirecionamento serão seguidos por um espaço.
```bash

```

### Espaço após redirecionamento

### Outras opções

- `-kp`, `--keep-padding`
- `-fn`, `--func-next-line`

## Pontos de atenção!

<https://github.com/mvdan/sh#caveats>

### Índices de arrays associativos

### Ambiguidade entre `$((` e `((`

### Algumas comandos embutidos são tratados como palavras chave





## Formatando um código "confuso"

aplicar nesse script aqui do transfer.sh
```bash
transfer(){ if [ $# -eq 0 ];then echo "No arguments specified.\nUsage:\n transfer <file|directory>\n ... | transfer <file_name>">&2;return 1;fi;if tty -s;then file="$1";file_name=$(basename "$file");if [ ! -e "$file" ];then echo "$file: No such file or directory">&2;return 1;fi;if [ -d "$file" ];then file_name="$file_name.zip" ,;(cd "$file"&&zip -r -q - .)|curl --progress-bar --upload-file "-" "https://transfer.sh/$file_name"|tee /dev/null,;else cat "$file"|curl --progress-bar --upload-file "-" "https://transfer.sh/$file_name"|tee /dev/null;fi;else file_name=$1;curl --progress-bar --upload-file "-" "https://transfer.sh/$file_name"|tee /dev/null;fi;}
```



## Instale shfmt no seu editor

### VSCode

### vim

### bônus: EditorConfig


