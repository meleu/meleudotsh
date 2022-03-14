---
title: "Consulte os números de casos e mortes causadas pelo Coronavirus diretamente do terminal"
description: >
  Escreveremos um script para obter, diretamente no terminal, os números atualizados de casos e mortes causadas pelo COVID-19 (doença causada pelo Coronavirus). Usaremos alguns truques de curl, jq, arrays associativos e mais. Stay at /home!
tags:
  - links
  - codigo
  - jq
date: 2020-03-17T16:19:38-03:00
---

O mundo está em pânico! O tal do Coronavirus está se alastrando rapidamente pelo mundo. Mesmo que exista uma turma tentando minimizar o alarde e nos convencer que não tem necessidade de tanto pânico, dizendo que existem/existiram outras doenças matando muito mais... O melhor é se prevenir!

Uma das recomendações que estão sendo dadas é para que as pessoas não saiam de casa. E já que vamos ficar em casa (stay at `/home`), que tal um pouco de nerdeza para distrair?

Neste artigo vamos criar um script que consultará uma API REST para obter o número de casos oficialmente reportados de COVID-19 (doença causada pelo Coronavirus).

Eis algums comandos/técnicas que iremos usar:

- `curl` para obter os dados;
- `jq` para tratar o JSON;
- array associativo para armazenar as informações tratadas pelo `jq`;
- redirecionamento para `while read` para alimentar o array associativo.


Vamos começar...


## Primeiro Passo: obtendo os dados com `curl`

Eu conheci essa API lá no github, o repositório fica em https://github.com/novelcovid/api

O endpoint que iremos acessar é o `https://corona.lmao.ninja/all` para pegar um resumo dos dados globais. E também usaremos o `https://corona.lmao.ninja/countries/[nome-do-pais]` para consultas por país.

Vamos usar o `curl` para ver o que esses endpoints nos retornam:
```shell-session
$ curl --silent 'https://corona.lmao.ninja/all'
{"cases":186993,"deaths":7477,"recovered":80842,"updated":1584448250501}
$ 
$ curl --silent 'https://corona.lmao.ninja/countries/brazil'
{"country":"Brazil","cases":234,"todayCases":0,"deaths":0,"todayDeaths":0,"recovered":2,"critical":18}
```

Legal, parecem ser informações úteis. Mas assim com esse JSON cru, todo coladinho fica meio ruim de visualizar...


## Segundo Passo: consumindo o JSON com `jq`

Um comando bem útil para tratar de dados JSON na linha de comando é o `jq` (se não tiver ele na sua máquina, trate de instalar, pois ele é muito útil!).

Vejamos como ele pode "embelezar" aquele JSON e torná-lo mais legível:
```shell-session
$ curl --silent 'https://corona.lmao.ninja/all' | jq '.'
{
  "cases": 186993,
  "deaths": 7477,
  "recovered": 80842,
  "updated": 1584448250501
}

$ curl --silent 'https://corona.lmao.ninja/countries/brazil' | jq '.'
{
  "country": "Brazil",
  "cases": 234,
  "todayCases": 0,
  "deaths": 0,
  "todayDeaths": 0,
  "recovered": 2,
  "critical": 18
}

```

Bem melhor, não é mesmo?

Neste exemplo usamos apenas o `'.'`, que é uma maneira simples de dizer ao `jq` que você só quer embelezar a saída (pretty-print) de uma entrada JSON. Porém através desse argumento, que precisa estar entre aspas, podemos fazer coisas muito mais poderosas com o `jq`.

Vamos aqui usar alguns comandos mais avançados do `jq` para coletar os dados que nos interessam e posteriormente salvá-los em um array...

Primeiro vamos fazer com que cada propriedade do JSON seja convertido em um objeto com o par key-value (chave-valor). É, isso parece meio confuso, mas com um exemplo tudo fica mais claro.

O que nós queremos é que, por exemplo, `"country": "Brazil"` vire isso:
```json
  {
    "key": "country",
    "value": "Brazil"
  }
```

E para alcançar tal objetivo, vamos usar o `to_entries` do próprio `jq`.
```shell-session
$ curl --silent 'https://corona.lmao.ninja/all' | jq '. | to_entries'
[
  {
    "key": "cases",
    "value": 187332
  },
  {
    "key": "deaths",
    "value": 7477
  },
  {
    "key": "recovered",
    "value": 80843
  },
  {
    "key": "updated",
    "value": 1584448850495
  }
]
```

Como você pode observar, o comando/filtro passado para o `jq` foi `'. | to_entries'`. É importante que você observe duas coisas:

1. O comando/filtro está entre 'aspas simples'.
2. Tem um `|` pipe dentro das aspas simples.

O `jq` permite que você envie a saída de um comando/filtro para outro usando `|` pipes, mas para isso **é importante que eles estejam dentro das aspas**. Caso contrário o shell vai interpretar que o `|` pipe está sendo enviado para ele (o shell).

Continuando...

Nós queremos usar o par key-value para posteriormente salvar esse conteúdo em um array. Vamos prosseguir (fique atento às explicações nos comentários entre os comandos abaixo):
```shell-session
$ curl --silent 'https://corona.lmao.ninja/all' \
    | jq '. | to_entries'
[
  {
    "key": "cases",
    "value": 187404
  },
  {
    "key": "deaths",
    "value": 7478
  },
  {
    "key": "recovered",
    "value": 80848
  },
  {
    "key": "updated",
    "value": 1584450050381
  }
]

$ # to_entries retornou um array de objetos.
$ # Vamos pegar só o que está dentro do array usando '.[]'
$ curl --silent 'https://corona.lmao.ninja/all' \
    | jq '. | to_entries | .[]'
{
  "key": "cases",
  "value": 187404
}
{
  "key": "deaths",
  "value": 7478
}
{
  "key": "recovered",
  "value": 80848
}
{
  "key": "updated",
  "value": 1584450050381
}

$ # agora vamos pegar somente o valor de cada
$ # propriedade, colocá-las em apenas uma linha
$ # e separar os valores com um '=' (igual)
$ curl --silent 'https://corona.lmao.ninja/all' \
    | jq -r '. | to_entries | .[] | .key + "=" + .value'
jq: error (at <stdin>:0): string ("cases ") and number (187404) cannot be added

$ # Whooops! o sinal de + serve para concatenar strings,
$ # mas se algum operando é um número, o + é um operador aritmético.
$ # Portanto vamos converter esse número para string.
$ curl --silent 'https://corona.lmao.ninja/all' \
    | jq -r '. | to_entries | .[] | .key + "=" + (.value|tostring)'
cases=188146
deaths=7497
recovered=80848
updated=1584450650499
```

Prontinho! Essa saída está perfeita para o nosso propósito!


## Terceiro Passo: salvando os dados em um array associativo

Para armazenar as informações fornecidas pelo `jq`, vamos usar um array associativo.

Um array associativo é aquele que você acessa os elementos através de um identificador, geralmente chamado de chave (ou _key_). Exemplo: se você atribui `carro[cor]='prata'`, a partir daí você pode fazer `echo "${carro[cor]}"` (você pode aprender mais sobre arrays de uma maneira geral [neste artigo](https://debxp.org/cbpb/aula06) do amigo Blau Araujo.)

Para alimentar o nosso array, vamos usar um redirecionamento para `while read`. O trecho de código a seguir faz esse serviço:
```
# covid.sh - ainda um protótipo...

# deixando claro que data é um array associativo
declare -A data

json2array() {
  local jsonData="$1"

  while IFS== read -r key value; do
    data[$key]="$value"
  done < <(jq -r '. | to_entries | .[] | .key + "=" + (.value|tostring)' <<<"$jsonData")
}
```

Em [um outro artigo](http://meleu.sh/percorrer-arquivo/) eu explico em detalhes essa técnica de redirecionar conteúdo para um `while read`. Aqui eu vou explicar de maneira breve:

- `IFS==`: como vimos no tópico acima, aquele comando `jq` vai me fornecer um conteúdo parecido com isso: `cases=188146`. Portanto, se eu setar o `IFS` como `=` sinal de igual, ele vai separar `cases` e `188146` em dois argumentos.

- `read -r key value`: como o read receberá dois argumentos, o primeiro será `$key` e o segundo será `$value`.

- `data[$key]="$value"`: como `data` é um array associativo, aqui teremos algo do tipo `data[cases]="188146"`.

Vamos executar a função acima de maneira interativa e ver se tudo está correto:
```shell-session
$ # "carregando" a função nessa instância do shell
$ . covid.sh
$
$ # vamos colocar o JSON em uma variável:
$ myJson="$(curl --silent 'https://corona.lmao.ninja/all')"
$
$ # agora sim vamos passar o JSON para alimentar o array:
$ json2array "$myJson"
$
$ # nada aconteceu... mas o array já está populado! :)
$ echo "casos: ${data[cases]}"
casos: 188433
$ echo "mortes: ${data[deaths]}"
mortes: 7500
```

Massa! Agora temos o que precisamos para escrever um scriptzinho que irá nos mostrar os números globais e do Brasil! :)


## Último Passo: o script

Nesse script nós vamos consultar os números referentes aos dados globais e do Brasil.

```
#!/usr/bin/env bash
# covid.sh
##########
#
# Imprime números a respeito do COVID-19 (doença causada pelo Coronavirus).
#

readonly API_URL='https://corona.lmao.ninja'

# deixando claro que data é um array associativo
declare -A data

json2array() {
  local jsonData="$1"

  # verificando se é um JSON válido
  jq empty <<<"$jsonData" 2>/dev/null || return 1

  while IFS== read -r key value; do
    data[$key]="$value"
  done < <(jq -r '. | to_entries | .[] | .key + "=" + (.value|tostring)' <<<"$jsonData")
}

main() {
  local json
  
  # primeiro dados globais:
  json="$(curl --silent "${API_URL}/all")"

  if ! json2array "$json"; then
    echo "ERRO: falha ao obter os dados de $API_URL" >&2
    exit 1
  fi

  echo -n "
=========================================
CASOS DE COVID-19 OFICIALMENTE REPORTADOS
==========================================

Última atualização: $( date -d @$((data[updated] / 1000)) )

Dados Globais
^^^^^^^^^^^^^
Número de casos..............: ${data[cases]}
Número de mortes.............: ${data[deaths]}
Pacientes curados............: ${data[recovered]}
"

  # obtendo dados do Brasil
  json="$(curl --silent "${API_URL}/countries/brazil")"

  if ! json2array "$json"; then
    echo "ERRO: falha ao obter os dados de $API_URL" >&2
    exit 1
  fi

  echo -n "
Dados Referentes ao Brasil
^^^^^^^^^^^^^^^^^^^^^^^^^^
Número de casos..............: ${data[cases]}
Casos registrados hoje.......: ${data[todayCases]}
Número de mortes.............: ${data[deaths]}
Mortes registradas hoje......: ${data[todayDeaths]}
Pacientes curados............: ${data[recovered]}
Pacientes em situação crítica: ${data[critical]}
"
}

main "$@"
```

Agora vamos o script em ação:
```shell-session
$ ./covid.sh 

=========================================
CASOS DE COVID-19 OFICIALMENTE REPORTADOS
==========================================

Última atualização: Ter Mar 17 11:30:50 -03 2020

Dados Globais
^^^^^^^^^^^^^
Número de casos..............: 188623
Número de mortes.............: 7511
Pacientes curados............: 80874

Dados Referentes ao Brasil
^^^^^^^^^^^^^^^^^^^^^^^^^^
Número de casos..............: 301
Casos registrados hoje.......: 67
Número de mortes.............: 1
Mortes registradas hoje......: 1
Pacientes curados............: 2
Pacientes em situação crítica: 18

```


## Passo Extra: aprimore o script!

Sugestões de melhorias que você pode implementar neste script como exercício:

- Possibilidade de fornecer o nome do país como parâmetro na linha de comando, e então obter os dados referente ao país informado.

- Usar o `printf` para imprimir os números de uma maneira mais "legível". Exemplo: `188.623` no lugar de `188623`.

- Acessar o endpoint `https://corona.lmao.ninja/countries` e imprimir os 5 países com maior número de casos/mortes.

- Use sua criatividade...


## Fontes

- https://github.com/javieraviles/covidAPI - repositório com a descrição de como usar a API.
- https://github.com/novelcovid/api - repositório com a descrição de como usar a api rest.
- https://stedolan.github.io/jq/manual/ - manual do `jq`
- https://jqplay.org/ - ótima ferramenta para treinar `jq` e ajudar a montar seu comando/filtro.
- https://debxp.org/cbpb/aula06 - uma aula do amigo [Blau Araujo](https://debxp.org/user/blau) sobre arrays no bash.
