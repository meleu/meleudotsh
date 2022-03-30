---
title: "Como criar um gráfico de barras no terminal listando os países com mais ocorrências de morte pelo Coronavirus"
description: >
  Escreveremos um script que mostra, no terminal, um gráfico em barras com o ranking dos países com mais casos de morte por COVID-19 (Coronavirus).Usaremos curl, jq, e criaremos o gráfico com o termgraph.
tags:
  - codigo
  - links
  - jq
  - curl
date: 2020-03-27T16:21:54-03:00
cover:
  image: "img/coronavirus-ranking.png"
  alt: "output de covid-ranking.sh"
---

Recentemente descobri uma ferramenta bem bacaninha que serve para gerar simples e belos gráficos de barra diretamente no terminal. A ferramenta se chama `termgraph` e seu repositório no github é [https://github.com/mkaz/termgraph](https://github.com/mkaz/termgraph).

E aproveitando a enorme oferta de dados disponível na web sobre a epidemia do Coronavírus, achei que esse seria um bom caso de uso para aprendermos a usar a ferramenta.

Vamos obter uma lista com as 10 nações com maior número de mortes por COVID-19 e gerar um gráfico de barras mostrando esse ranking.

Vamos nessa!


## Primeiro passo: instalando e usando o `termgraph`

Para instalar é muito simples:
```shell-session
pip3 install termgraph
```

E pronto. Agora digite um `termgraph -h` só para se certificar que a instalação foi feita com sucesso.

O formato dos dados a serem enviados ao `termgraph` pra ele gerar o gráfico de barras é absurdamente simples. Trata-se apenas de um arquivo onde cada linha contém um rótulo, seguido de espaço(s) (ou vírgula), seguido do(s) número(s) a serem usados como valores para criar o gráfico.

Veja o exemplo abaixo:

```shell-session
$ cat data/ex1.dat
# Example Data Set 1
2007    183.32
2008    231.23
2009     16.43
2010     50.21
2011    508.97
2012    212.05
2014    1.0

$ # observe acima! cada linha começa com o rótulo
$ # seguido do valor referente aquele rótulo.
$ # divinamente simples!!
$
$ termgraph data/ex1.dat

2007: ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 183.32
2008: ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 231.23
2009: ▇ 16.43
2010: ▇▇▇▇ 50.21
2011: ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 508.97
2012: ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 212.05
2014: ▏ 1.00 

```

Bem legal, não acha?!

Pois agora o que temos que fazer é obter os dados que queremos para fazer o ranking.


## Segundo passo: obtendo os dados com `curl`

Assim como no meu [artigo anterior](/coronavirus/), mais uma vez vamos usar a API que é documentada nesse repositório: <https://github.com/disease-sh/API>

Lá podemos ver que o endpoint a ser acessado para obtermos uma lista ordenada pelo número de mortes é `https://disease.sh/v3/covid-19/countries?sort=deaths`.

Se você fizer o teste aí no seu terminal, vai perceber que sua tela vai receber uma enxurrada de dados no formato JSON. Abaixo eu mostro apenas o início dos dados:

```shell-session
$ curl 'https://disease.sh/v3/covid-19/countries?sort=deaths'
[{"country":"Italy","countryInfo":{"_id":380,"lat":42.8333,...
```

Para simplificar os nossos testes iniciais e evitar de ficar acessando a API desnecessariamente, vamos salvar esse JSON num arquivo:

```shell-session
$ curl 'https://disease.sh/v3/covid-19/countries?sort=deaths' > corona-deaths.json
```

Agora esses dados JSON brutos, precisam ser lapidados. E mais uma vamos usar o `jq` para isso:


## Terceiro passo: garimpando dados com `jq`

O `jq` uma ferramenta bem útil para tratar dados JSON diretamente pela linha de comando. Se não tiver instalado na sua máquina, instale-o. Geralmente o gerenciador de pacotes oficial da sua distro (`apt-get`, `pacman`, etc.) já vai ter um pacote chamado `jq`, portanto creio que você não terá problemas com isso.

Conforme também vimos no meu [artigo anterior](http://meleu.sh/coronavirus/), o `jq` dá uma embelezada no JSON tornando-o mais legível. No entanto se usarmos o tradicional `jq '.'` serão tantas linhas exibidas na tela que dificilmente conseguiremos visualizar as primeiras entradas da lista.

Portanto vamos começar a fazer alguns truques para limpar esses dados. Não está no escopo desse texto, explicar cada opção, irem apenas mostrar a "receitinha do bolo". Mas na [documentação oficial](https://stedolan.github.io/jq/manual/) você pode consultar todas essas opções.

### 1. Pegar somente as 10 primeiras entradas

Lembre-se, no passo anterior, usamos o `curl` para salvar os dados num arquivo chamado `corona-deaths.json`.



```shell-session
$ jq '.[:10]' corona-deaths.json
[
  {
    "country": "Italy",
    "countryInfo": {
      "_id": 380,
      "lat": 42.8333,
      "long": 12.8333,
      "flag": "https://disease.sh/assets/flags/it.png",
      "iso3": "ITA",
      "iso2": "IT"
    },
    "cases": 80589,
    "todayCases": 0,
    "deaths": 8215,
    "todayDeaths": 0,
    "recovered": 10361,
    "active": 62013,
    "critical": 3612,
    "casesPerOneMillion": 1,
    "deathsPerOneMillion": 136
  },
  .
  .
  .
]
$ # mostrando apenas a primeira entrada para
$ # economizar esse espaço...
```

Como podemos ver, o JSON começa com `[` e termina com `]`. Isso significa que os dados estão dentro de um array. Vamos agora pegar esses dados "individualmente", sem estarem encapsulados num array.

### 2. Pegando os dados de dentro do array

```shell-session
$ jq '.[:10][]' corona-deaths.json
{
  "country": "Italy",
  "countryInfo": {
    "_id": 380,
    "lat": 42.8333,
    "long": 12.8333,
    "flag": "https://disease.sh/assets/flags/it.png",
    "iso3": "ITA",
    "iso2": "IT"
  },
  "cases": 80589,
  "todayCases": 0,
  "deaths": 8215,
  "todayDeaths": 0,
  "recovered": 10361,
  "active": 62013,
  "critical": 3612,
  "casesPerOneMillion": 1,
  "deathsPerOneMillion": 136
},
.
.
.
$ 
```

OK. Só que ainda tem muitos dados que não nos interessam ali.

### 3. Pegando apenas as propriedades que nos interessam

Aqui nós vamos matar dois coelhos com uma cajadada só.

Nós precisamos apenas do nome do país e do número de mortes. Precisamos também exibir essas informações naquele formato usado pelo `termgraph`: `rótulo número`.

Alcançamos esse objetivo usando o recurso de [string interpolation do jq](https://stedolan.github.io/jq/manual/#Stringinterpolation-%5C%28foo%29)
```shell-session
$ jq '.[:10][] | "\(.country) \(.deaths)"' corona-deaths.json
"Italy 8215"
"Spain 4858"
"China 3292"
"Iran, Islamic Republic of 2378"
"France 1696"
"USA 1307"
"UK 759"
"Netherlands 546"
"Germany 304"
"Belgium 289"
```

Opa! Parece que deu certo, mas... Aquela linha do Iran parece que vai ser problemática...

Como a ocorrência de vírgula e espaços é uma coisa que pode ocorrer no nome dos países, vamos usar um outro delimitador para separar rótulos dos números e depois a gente dá um jeito de dizer ao `termgraph` que o delimitador é diferente do default.

Para o nosso propósito aqui, vamos usar o `;` ponto-e-vírgula (espero que não apareça país algum com ponto-e-vírgula no nome!):

```shell-session
$ jq '.[:10][] | "\(.country);\(.deaths)"' corona-deaths.json
"Italy;8215"
"Spain;4858"
"China;3292"
"Iran, Islamic Republic of;2378"
"France;1696"
"USA;1307"
"UK;759"
"Netherlands;546"
"Germany;304"
"Belgium;289"
```

Tem outro detalhezinho que precisamos ajeitar antes de mandar esses dados para o `termgraph`: as "aspas".

Observe que cada linha começa e termina com aspas. O `termgraph` não vai entender isso muito bem. Portanto precisamos removê-las, e vamos usar o `tr` para isso:

```shell-session
$ jq '.[:10][] | "\(.country);\(.deaths)"' corona-deaths.json \
  | tr -d \"
Italy;8215
Spain;4858
China;3292
Iran, Islamic Republic of;2378
France;1696
USA;1307
UK;759
Netherlands;546
Germany;304
Belgium;289
```

Vamos salvar esse conteúdo em um arquivo para mandar pro `termgraph`:

```shell-session
$ jq '.[:10][] | "\(.country);\(.deaths)"' corona-deaths.json \
  | tr -d \" > input.dat
```

### 4. Mandando os dados para o `termgraph`

Vamos ver se esse gráfico vai ficar bonito mesmo.

Lembremos que optamos por usar o `;` ponto-e-vírgula como delimitador, portanto precisamos avisar ao `termgraph` sobre isso usando a opção `--delim`:

```shell-session
$ # OBSERVAÇÃO: o ponto-e-vírgula precisa estar entre aspas
$ # caso contrário o shell vai achar que é o fim de um comando
$ termgraph --delim ';' input.dat

Italy                    : ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 8215.00
Spain                    : ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 4858.00
China                    : ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 3292.00
Iran, Islamic Republic of: ▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 2378.00
France                   : ▇▇▇▇▇▇▇▇▇▇ 1696.00
USA                      : ▇▇▇▇▇▇▇ 1307.00
UK                       : ▇▇▇▇ 759.00
Netherlands              : ▇▇▇ 546.00
Germany                  : ▇ 304.00
Belgium                  : ▇ 289.00

```

Epa! Bem legal, né não?

Mas... Pode parecer bobagem, mas aquele `.00` no final está me incomodando... O `termgraph` sempre considera que o número é um valor com casas decimais, mas como não é possível que tenhamos, por exemplo 3.72 mortes, eu vou remover aquilo com o `sed`:

```shell-session
$ termgraph --delim ';' input.dat | sed 's/\.00$//'

Italy                    : ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 8215
Spain                    : ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 4858
China                    : ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 3292
Iran, Islamic Republic of: ▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 2378
France                   : ▇▇▇▇▇▇▇▇▇▇ 1696
USA                      : ▇▇▇▇▇▇▇ 1307
UK                       : ▇▇▇▇ 759
Netherlands              : ▇▇▇ 546
Germany                  : ▇ 304
Belgium                  : ▇ 289
```

Agora sim! Temos todo os insumos necessários para montar o nosso script!

## Último passo: o script

```
#!/usr/bin/env bash
# covid-ranking.sh
##################
#
# Exibe um gráfico de barras com o ranking dos países com
# maior número de mortes causadas pelo COVID-19 (Coronavirus).
#

readonly URL='https://disease.sh/v3/covid-19/countries?sort=deaths'

readonly DEPENDENCIES=(curl jq termgraph)

checkDependencies() {
  local errorFound=0

  for command in "${DEPENDENCIES[@]}"; do
    if ! which "$command" > /dev/null ; then
      echo "ERRO: não encontrei o comando '$command'" >&2
      errorFound=1
    fi
  done

  if [[ "$errorFound" != "0" ]]; then
    echo "---IMPOSSÍVEL CONTINUAR---"
    echo "Esse script precisa dos comandos listados acima" >&2
    echo "Instale-os e/ou verifique se estão no seu \$PATH" >&2
    exit 1
  fi
}

main() {
  checkDependencies

  curl --silent "$URL" \
    | jq '.[:10][] | "\(.country);\(.deaths)"' \
    | tr -d \" \
    | termgraph --delim ';' --title 'Países com maior casos de mortes por COVID-19' \
    | sed 's/\.00$//'
}

main "$@"
```

Agora vejamos o script em ação:

```shell-session
$ chmod a+x covid-ranking.sh 
$ ./covid-ranking.sh 

# Países com maior casos de mortes por COVID-19

Italy      : ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 8215
Spain      : ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 4934
China      : ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 3292
Iran       : ▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 2378
France     : ▇▇▇▇▇▇▇▇▇▇ 1696
USA        : ▇▇▇▇▇▇▇▇ 1382
UK         : ▇▇▇▇ 759
Netherlands: ▇▇▇ 546
Germany    : ▇ 304
Belgium    : ▇ 289

```

**CURIOSIDADE**: nos testes finais antes da publicação deste artigo eu observei que o nome do "Iran" não está mais retornando com a vírgula e aqueles outros dizeres. Observe isso na saída acima! Ainda bem que apareceu antes e pudemos contornar um possível problema no código que só apareceria no futuro! :)

## Passo(s) Extra(s): aprimore o script

Sugestões de melhorias que você pode implementar nesse script como exercício:

- Veja [na documentação da API](https://disease.sh/docs/) como obter os dados ordenados por outros critérios (exemplo: `cases`, `todayCases`, `recovered`, etc.)
- Uma vez que tenha aprendido a ordenar por outros critérios, gerar gráficos do ranking dos países com mais casos, com mais casos registrados hoje, com mais casos recuperados. Você pode fazer isso por exemplo adicionando uma opção de linha de comando chamada `--sort-by`
- Adicione a opção da linha de comando `--max`, onde o usuário poderá especificar o número de entradas no ranking (exemplo: apenas os top-5 ou os top-20).
- Use sua criatividade.


## Fontes

- <https://github.com/disease-sh/api> - repositório com a descrição de como usar a api rest.
- <https://stedolan.github.io/jq/manual/> - manual do `jq`
- <https://jqplay.org/> - ótima ferramenta para treinar `jq` e ajudar a montar seu comando/filtro.

