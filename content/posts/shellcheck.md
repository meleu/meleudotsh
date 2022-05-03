---
title: "Use shellcheck e livre-se dos bugs no seu código antes mesmo de executá-lo"
description: >
  "Com o shellcheck você vai se livrar de muitos aborrecimentos e imprevistos com bugs que você jamais conseguiria prever!"
tags:
  - boas-praticas
  - ferramentas
date: 2022-05-02T15:11:31-03:00
cover:
  image: "img/shellcheck.png"
  alt: shellcheck
draft: true
---

Nós amamos programar, não é mesmo? (se você está lendo esse blog, eu imagino que sim). Mas convenhamos, quando chegamos naquele ponto onde fazer uma simples alteração envolve ficar continuamente alternando entre escrever-salvar-testar... Tem hora que enche o saco!

A ferramenta que vou mostrar neste artigo vai nos ajudar a minimizar essas constantes trocas de contexto, que tanto nos incomodam e quebram o ritmo da nossa escrita.

O shellcheck é um *linter* para shell scripts. Segundo a Wikipedia um [*linter*]](https://en.wikipedia.org/wiki/Lint_(software)) é:

> uma ferramenta de análise estática de código usada para alertar erros de programação, bugs, erros estilísticos e construções suspeitas.

Explicando de maneira mais simples: é uma ferramenta que vai análisar o seu código em busca de problemas e vai te alertar sobre o que ele encontrar.


## Um contexto pessoal

O shellcheck é mais uma daquelas ferramentas que mudam a vida das pessoas. Mudou a minha vida e caso você programe em shell, tenho certeza que mudará a sua também.

Eu sei que isso soa muito sensacionalista e exagerado... mas é sério, não consigo descrever o shellcheck de outra forma. Vou explicar o motivo falando um pouco do meu trabalho

Parte do meu dia a dia é escrevendo/mantendo esteiras de integração contínua escritas em bash. O ciclo escrever-salvar-testar não é tão simples quanto "salvar o arquivo, ir no terminal e executar o script". No meu trabalho esse ciclo geralmente envolve:

1. escrever e salvar minhas alterações
2. fazer o commit das minhas alterações para um repositório git
3. ir em outro repositório disparar a pipeline que vai executar o meu código
4. esperar a vez do *job* onde está o meu script ser executado
5. esperar o container ser instanciado
6. finalmente meu código será executado e eu poderei olhar o log em busca de problemas.

Imagina fazer todo esse processo pra no final das contas chegar a conclusão que você cometeu um erro de digitação num nome de variável... 🤦

De tão deprimente as vezes dá vontade de ficar trancado no banheiro, chorando encolhido e ouvindo Linkin Park.

Espero que essa historinha sirva pra explicar o motivo de tantos artigos deste blog começarem com esse discurso de "isso mudou minha vida" e de "antecipe os problemas antes de colocar seu código em produção".

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

![](shellcheck-web.png)

Observe que mesmo sendo um código perfeitamente válido, o shellcheck está nos apontando vários problemas:

- Linha 1:
    - erro: não atribuímos um shebang ao nosso script (num [artigo anterior](/shebang) aprendemos por que isso nunca deve acontecer no seu script)
- Linha 2:
    - alerta: a variável `$name` está sendo referenciada mas não foi atribuído valor algum a ela.
    - informação: em volta de variáveis devemos usar [aspas duplas sempre](/aspas-sempre)
- Linha 5:
    - erro: mais uma vez esquecemos das aspas...


### Instalação

Obviamente que quando você estiver codando, não vai ficar copiando e colando seu código numa página da web o tempo todo. Portanto vamos instalar o shellcheck na nossa máquina.

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

```txt
$ shellcheck remove-spaces.sh

In remove-spaces.sh line 6:
  for file in "$directory/*.mp3"; do
              ^----------------^ SC2066 (error): Since you double quoted this, it
will not word split, and the loop will only run once.

For more information:
  https://www.shellcheck.net/wiki/SC2066 -- Since you double quoted this, it ...

```

Eita! Parece que exageramos no alcance das aspas duplas ali...

Indo direto ao ponto, o que o shellcheck está tentando nos dizer é que como o `*` está dentro das aspas, aquele padrão não vai expandir para o nome dos arquivos `.mp3` que temos naquele diretório. Ele tentará procurar um arquivo chamado literalmente `*.mp3` (asterisco ponto mp3).

No README do shellcheck tem uma [galeria de código ruim](https://github.com/koalaman/shellcheck#gallery-of-bad-code), com uma lista de códigos problemáticos que podem até ser construções válidas para o shell, mas que não é exatamente o que você quer.


### ShellCheck wiki

Uma das coisas mais fantásticas do shellcheck não é nem o fato dele ficar jogando na nossa cara que nosso script está cheio de problemas. O mais legal é que ele nos diz **por que** o código é problemático e também **como** melhorar.

Observe a última linha da saída do shellcheck que executamos anteriormente:

```txt
For more information:
  https://www.shellcheck.net/wiki/SC2066 -- Since you double quoted this, it ...
```

Se dermos uma olhadinha [naquele link](https://www.shellcheck.net/wiki/SC2066), veremos o motivo pelo qual essa construção é problemática e também o que devemos fazer para obter o resultado que queremos com um código mais robusto.

No nosso exemplo aqui, bastaria usar `"$directory"/*.mp3` (fechar as aspas logo após o nome da variável).

Eu *preciso* enfatizar que o wiki do shellcheck é uma fonte valiosíssima de conhecimento sobre shell scripting.

Faça esse teste: rode o shellcheck no menor script que você tem aí rapidamente a mão e gaste um tempinho lendo os alertas e as páginas que o shellcheck recomenda.

Tenho certeza que você vai, tipo 🤯 "Wow! Nunca tinha parado pra pensar nisso!"

Bom, pelo menos foi isso que aconteceu comigo.

Fazendo esse simples exercício de rodar o shellcheck em alguns dos meus scripts eu aprendi coisas como:

- Devemos sempre verificar se um `cd` terminou com sucesso ([SC2164](https://www.shellcheck.net/wiki/SC2128)) - dica: deixar o seu [bash mais rigoroso](/bash-rigoroso) já resolve esse problema.

- Em um `rm`, devemos tomar muito cuidado com variáveis que fazem referência a um diretório, pois isso pode trazer consequências catastróficas. Exemplo: em um `rm -rf "$tmpdir"/*`, se a variável `$tmpdir` estiver vazia, isso vai virar um `rm -rf /*`. Seguindo a orientação do shellcheck ([SC2115](https://www.shellcheck.net/wiki/SC2115)), deveríamos usar `${tmpdir:?}`, pois isso fará o comando falhar se a variável estiver vazia.

- Devemos sempre deixar explícito o caminho relativo quando queremos referenciar arquivos usando o `*` asterisco ([SC2155](https://www.shellcheck.net/wiki/SC2155)). Isso serve para evitar que um nome de arquivo se torne uma opção para o comando. Exemplo: imagine o transtorno causado por um arquivo chamado `-rf` quando você usando o comando `rm *`.

- Um macetinho útil para quando fazemos um `cd` em nosso script: execute em `( subshell )`. Desta forma você não precisa fazer um `cd ..` ao terminar o que foi fazer naquele diretório ([SC2103](https://www.shellcheck.net/wiki/SC2103)).


Estes 👆 foram apenas alguns exemplos de coisas que aprendi rodando o shellcheck em scripts antigos. Gaste um tempinho fazendo isso e não se arrependerá.


### E se eu discordar do shellcheck?

Podem haver situações onde não concordamos com o diagnóstico do shellcheck e queremos ignorar certas coisas que ele considera problemática.

Um exemplo clássico pra mim é essa estratégia que eu sempre uso para saber em que diretório o meu script está armazenado:

```bash
#!/usr/bin/env bash
# example.sh

# maneira infalível de saber o diretório do seu script
export SCRIPTS_DIR="$(
  cd "$(dirname -- "${BASH_SOURCE}")" && pwd
)"
```

Agora vejamos o que o shellcheck tem a nos dizer:

```txt
$ shellcheck example.sh

In example.sh line 5:
export SCRIPTS_DIR="$(
       ^---------^ SC2155 (warning): Declare and assign separately to avoid
masking return values.


In example.sh line 6:
  cd "$(dirname -- "${BASH_SOURCE}")" && pwd
                    ^------------^ SC2128 (warning): Expanding an array without
an index only gives the first element.

For more information:
  https://www.shellcheck.net/wiki/SC2128 -- Expanding an array without an ind...
  https://www.shellcheck.net/wiki/SC2155 -- Declare and assign separately to ...
```

No primeiro alerta ele está falando que eu deveria declarar `SCRIPTS_DIR` primeiro, e depois atribuir um valor.

Este alerta serve para chamar a atenção para situações como essa:

```txt
$ # o export vai finalizar com sucesso,
$ # independente do que está dentro do $()
$
$ export var=$(comando invalido) && echo sucesso || echo falha
comando: command not found
sucesso
```

OK, isso realmente pode ser problemático. Mas eu tenho certeza que no caso específico daquele meu código eu não terei problemas. Portanto eu quero que o shellcheck ignore essa "regra".

Como podemos ver no output do shellcheck, a regra é identificada pelo ID `SC2155`. Vamos então desabilitá-la.

Uma maneira de fazer isso é ir na linha acima do problema e adicionar um comentário assim:
```bash
# shellcheck disable=2155
```

Pronto! Agora se você rodar o shellcheck de novo ele não vai mais reclamar disso.

Só que ainda temos o outro alerta, me avisando eu estou usando um array, `${BASH_SOURCE}`, sem especificar o indíce, e que isso vai me retornar apenas o primeiro elemento.

Entendo que esse é um alerta útil quando usamos arrays em construções do tipo `for var in "${myArray}"`, onde eu estou esperando obter todos os valores do array. Mas neste meu uso específico aqui, eu tenho certeza que o primeiro elemento do `$BASH_SOURCE` é exatamente o que eu quero. Portanto, basta ignorarmos a regra `SC2128`, adicionando o número desta regra naquele mesmo comentário, separando com uma vírgula.

```bash
# shellcheck disable=2155,2128
export SCRIPTS_DIR="$(
  cd "$(dirname -- "${BASH_SOURCE}")" && pwd
)"
```

Pronto! Agora o shellcheck não vai mais encrencar com esse macetinho bastante útil.


#### Outras maneiras de ignorar regras do shellcheck

[Nesta página](https://github.com/koalaman/shellcheck/wiki/Ignore) da documentação mostra várias maneiras de ignorar certas regras. Geralmente eu uso uma dessas aqui:

1. criando um arquivo `.shellcheckrc` na raiz do projeto (requer shellcheck 0.7+)
3. inserindo um comentário pra desabilitar a regra no topo do arquivo (logo após o shebang).
2. inserindo um comentário pra desabilitar a regra na linha acima do código problemático.



## Indo além

Nesta seção coloco um conteúdo um pouquinho mais avançado, mostrando várias situações onde eu uso o shellcheck e como isso torna minha vida mais feliz. 🙂

Se você está apenas iniciando no shellscript, não se preocupe se não entender tudo... Só de você se preocupar em rodar um `shellcheck` nos seus scripts eu te garanto que você está no caminho correto!


### Integrando shellcheck com o o seu editor

Já etendemos que o shellcheck é bem bacana e nos ajuda a antecipar muitos problemas. Mas se nos atentarmos um pouquinho vamos perceber que mais uma vez acabaremos entrando na repetição do ciclo escrever-salvar-testar.

Mesmo que o relatório do shellcheck seja completinho e isso vá minimizar a quantidade de ciclos escrever-salvar-testar, ainda assim podemos melhorar. Podemos integrar o shellcheck no nosso editor de texto!

O [README do shellcheck](https://github.com/koalaman/shellcheck#in-your-editor) lista links para várias maneiras de fazer isso em diversos editores, vou mostrar aqui apenas 3 deles.

**Pré-requisito**: você precisa ter o shellcheck já devidamente instalado na sua máquina.

#### VSCode

Não tem complicação alguma, simplesmente abra o VSCode, vá na seção de plugins, procure por ShellCheck, e instale.

![](shellcheck-vscode.png)

E pronto! Agora você será alertado sobre problemas no seu código a medida que está escrevendo.


#### vim / Syntastic

**OBS.:** Se você não usa vim, pode pular essa parte.

Apesar de existirem várias maneiras de integrar shellcheck com o vim, vou focar aqui no uso do [Syntastic](https://github.com/vim-syntastic/syntastic), pois foi o que eu achei que fez mais sentido pra mim.

Eu costumo administrar os plugins do meu vim usando o [vim-plug](https://github.com/junegunn/vim-plug) (explicar como instalar/utilizar o vim-plug está fora do escopo desse texto, confira a documentação).

Instalando o syntastic no meu `~/.vimrc`:

```vim
" isso só vai funcionar se você tiver
" o vim-plug devidamente instalado
call plug#begin()

" syntastic para integrar vim com shellcheck
Plug 'vim-syntastic/syntastic', { 'for': 'sh' }

call plug#end()

" configurações recomendadas pelo Syntastic
" https://github.com/vim-syntastic/syntastic#settings
set statusline+=%#warningmsg#
set statusline+=%{SyntasticStatuslineFlag()}
set statusline+=%*
let g:syntastic_always_populate_loc_list = 1
let g:syntastic_auto_loc_list = 1
let g:syntastic_check_on_open = 1
let g:syntastic_check_on_wq = 0
```

Depois de salvar o arquivo basta mandar um `:PlugInstall` (que é um comando do vim-plug).

Essa config vai ser suficiente para que quando você salvar o seu script, os erros já apareçam numa janelinha na parte de baixa da tela:

![](shellcheck-vim.png)

Pra você ir de uma mensagem de erro para a seguinte, use o comando `:lnext`, e pra voltar `:lprevious`.

Como ficar digitando isso é meio tedioso, eu criei um atalhozinho no meu `~/.vimrc`

```vim
" eu uso espaço como meu "leaderkey"
let mapleader = "\<Space>"

" pula para o erro seguinte/anterior
" (útil para usar com syntastic/shellcheck)
map <leader>ne :lnext<cr>     " ne = Next Error
map <leader>pe :lprevious<cr> " pe = Previous Error
```

Agora quando eu salvo o arquivo e shellcheck me mostra os erros, eu posso alterna entre eles usando `<space>ne` e `<space>pe` (eu memorizo como `ne` significando "Next Error", e `pe` para "Previous Error").

Como eu disse, essas configurações só fazem sentido para quem já está familiarizado com vim e gerenciamento de plugins.


#### Geany

O companheiro [Blau Araujo](https://twitter.com/blau_araujo) me informou que o Geany já tem uma integração com o shellcheck e é automaticamente disponível se você abre um arquivo de shell script.

Para fazer o Geany analisar seu script, basta ir no menu Construir -> Lint.


### Usando git hook para impedir commit de código problemático

Se você trabalha sozinho num projeto, integrar o shellcheck no seu editor pode ser suficiente. Mas se você trabalha com mais pessoas, precisamos tomar alguns cuidados adicionais.

Uma coisa que eu costumo fazer nos projetos que trabalho é criar um scriptzinho pra rodar o shellcheck em todos os arquivos `*.sh` do repositório. E aí oriento o pessoal a utilizar esse script como um git hook de pre-commit.

Um git hook de pre-commit nada mais é que um script que será executado imediatamente após você enviar um comando `git commit`, porém antes desse commit ser efetivamente realizado. Se o script falhar, o commit **não** é realizado.

No diretório onde está o seu repositório local, você tem um diretório chamado `.git/`. Dentro deste diretório tem um outro chamado `hooks/`. É ali que ficam os git hooks.

O script abaixo, deve ser colocado dentro do arquivo `.git/hooks/pre-commit` e deve ser executável:

```bash
#!/usr/bin/env bash
# git-hook-pre-commit.sh
########################
#
# Esse script previne que código shell problemático
# seja commitado (arquivos .sh). Ele deve ser utilizado
# como um git hook de pre-commit.
#
# Dependência: git e shellcheck
#
# Se você usa uma distro debian-based:
# $ sudo apt install shellcheck
#
############################################################
#
# COMO UTILIZAR
###############
#
# No seu repositório local haverá um diretório '.git',
# você deve colocar o conteúdo deste arquivo dentro de
# '.git/hooks/pre-commit' e atribuir permissão de eXecução.
#
# $ cat git-hook-pre-commit.sh > .git/hooks/pre-commit
# $ chmod a+x .git/hooks/pre-commit
############################################################

main() {
  git ls-files \
    --exclude='*.sh' \
    --ignored \
    --cached \
    -z \
    | xargs -0r shellcheck
}

main
```

Pronto! Fazendo isso corretamente, o próprio git agora vai te informar se o seu código está problemático toda hora que você fizer um `git commit`.


### Crie uma pipeline para checagem de código

Imagine que você está trabalhando num repositório com mais outras pessoas.

Você se importa com a qualidade do seu código, instala shellcheck na sua máquina, integra com seu editor, usa um git hook de pre-commit pra não enviar código bugado pro repositório... Mas aí pode acontecer um commit de um colaborador que ~~está cagando pra tudo isso~~ ainda não aprendeu a importância de mantermos um código livre de bugs e potenciais problemas.

Para que possamos ter esse controle no nosso repositório também, podemos criar uma pipeline que varre nosso código em busca de problemas.

Como eu trabalho com GitLab CI, o job ficaria assim:

```yaml
stages:
  - shellcheck
  # - outros estágios...

shellcheck:
    image: koalaman/shellcheck-alpine:latest
    stage: shellcheck
    script:
    - apk add --upgrade git
    - git ls-files --exclude='*.sh' --ignored -c -z | xargs -0r shellcheck
```

Aí agora quando entro na página principal projeto no gitlab, já bato o olho na bolinha que aparece ali pra mostrar o status da última pipeline. Aí posso ver como que é bom trabalhar com uma equipe que só tem fera:

![](shellcheck-pipeline-ok.png)

E de vez em quando vejo situações como essa:

![](shellcheck-pipeline-fail.png)

No [wiki do shellcheck](https://github.com/koalaman/shellcheck/wiki) tem algumas dicas para integrar a ferramenta em várias outras plataformas além do gitlab. Se você trabalha com outro tipo de esteira, vale a pena uma conferida.

