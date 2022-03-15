---
title: "Como gerar uma URL curta no bitly direto da linha de comando"
description: >
  Com o curl e uma conta no bitly.com podemos facilmente criar um script para termos nosso próprio "URL shortener" diretamente na linha de comando.
tags:
  - codigo
  - links
  - jq
date: 2020-02-15T16:17:04-03:00
cover:
  image: "url-shortener.png"
  alt: "função shortener()"
---

Um conhecido serviço de "encurtamento de URL" que existe na web é o [https://bitly.com](https://bitly.com). Neste site podemos criar versões mais curtas para URLs grandes. Se você já se deparou com links do tipo `bit.ly/AlgumaCoisa`, sabe do que estou falando, certo?

Neste artigo vamos usar a API do bitly.com através do `curl` e assim criar o nosso próprio "URL shortener" para usar direto da linha de comando.

Se você não conhece o `curl`, trata-se de um comando bastante poderoso e útil, que serve para transferir dados de/para servidores usando diversos protocolos (HTTP, HTTPS, FTP, TELNET, e muitos outros). Neste artigo veremos apenas uma pequena fração de todo o poder do `curl`.  

## Primeiro Passo: criar uma conta no bitly.com

Para podermos usar a API do bitly.com, vamos precisar criar uma conta. O processo é bastante simples e direto, vá no site [https://bitly.com](https://bitly.com) e clique no link "Sign up" no canto superior direito.

Uma vez criada a conta e confirmado o seu registro via email, você poderá acessar o seu "dashboard". É uma tela parecida com essa:

![bitlyDashboard](http://meleu.sh/content/images/2020/02/bitlyDashboard.png)

## Segundo Passo: obter um _access token_

Ali no canto superior direito (onde na minha imagem está escrito "Meleu Zord") você vai clicar e vai aparecer um menu. Escolha "Profile Settings" e em seguida escolha "Generic Access Token". Você verá algo parecido com isso:

![genericAccessToken](http://meleu.sh/content/images/2020/02/genericAccessToken.png)

Digite sua senha naquele campo onde está escrito "PASSWORD" e tecle enter. Você receberá seu _access token_, que é uma string bem loucona tipo isso: `ac4b1cabb9264079c4e9f2532804f81364c09b12` (este é um token inválido e fictício).


## Teiceiro Passo: acessando a API do bitly com o `curl`

Lendo a [documentação do bitly.com](https://dev.bitly.com/v4_documentation.html) é possível entender como usar a API, mas aqui já vou te dar tudo mastigadinho.


### Autenticação ao Portador

A autenticação feita na API do bitly.com é através de um método conhecido como "_bearer authentication_". Trata-se de um esquema de autenticação envolvendo tokens criptografados chamados de _bearer tokens_. Geralmente estes tokens são gerados e fornecidos pelo servidor, e foi exatamente o que fizemos no segundo passo acima.

A palavra inglesa _bearer_ em português significa "portador", portanto podemos interpretar esse tal de _bearer authentication_ como "dê acesso ao portador deste token".

Para usarmos esta metodologia de autenticação através do `curl`, fazemos da seguinte forma:

```
curl \
  --header "Authorization: Bearer ac4b1cabb9264079c4e9f2532804f81364c09b12"
```

Essa linha de comando ainda não está pronta...


### Passando dados no formato JSON

A versão mais atual da API do bitly (versão 4) espera receber dados no formato JSON. E neste JSON precisamos passar a propriedade `long_url` contendo a URL que queremos encurtar.

No exemplo abaixo vamos encurtar a URL `https://meleu.sh/elemento-presente-no-array/` (muita atenção nas aspas!):
```
curl \
  --header "Authorization: Bearer ac4b1cabb9264079c4e9f2532804f81364c09b12" \
  --header "Content-Type: application/json" \
  --data '{ "long_url": "https://meleu.sh/elemento-presente-no-array/" }'
```

Calma que o comando ainda não está pronto! Precisamos dizer ao `curl` para onde vamos enviar estas informações...


### Acessando o endpoint `shorten`

O endpoint da API responsável por encurtar URLs é o `shorten`, e ele é acessado através desta URL: `https://api-ssl.bitly.com/v4/shorten`.

Portanto o comando completo (e pronto para ser usado) vai ficar parecido com isso (lembre-se que o token abaixo é inválido, use o seu próprio token):
```
curl \
  --header "Authorization: Bearer ac4b1cabb9264079c4e9f2532804f81364c09b12" \
  --header "Content-Type: application/json" \
  --data '{ "long_url": "https://meleu.sh/elemento-presente-no-array/" }' \
  https://api-ssl.bitly.com/v4/shorten
```

Se tudo der certo com o comando acima, receberemos como resposta um JSON parecido com isso (se o seu der errado, lembre-se de reconferir o seu token):
```
{"created_at":"2020-02-16T01:16:54+0000","id":"bit.ly/2StrCQC","link":"http://bit.ly/2StrCQC","custom_bitlinks":[],"long_url":"https://meleu.sh/elemento-presente-no-array/","archived":false,"tags":[],"deeplinks":[],"references":{"group":"https://api-ssl.bitly.com/v4/groups/bK311XKuDpS"}}
```

Uma zona, não é mesmo? Vamos ajeitar esse output usando o `jq`.


## Quarto Passo: pegando somente o link curto

O `jq` é um comandinho que serve para processar dados no formato JSON.

Não faz parte do escopo deste artigo explicar o que é JSON nem os detalhes de como usar o `jq` (talvez num próximo artigo). Então vou apenas mostrar aqui a "receitinha de bolo" de como obter o que queremos.

Para tornar aquele "tijolão" de JSON que obtivemos como resposta, basta mandarmos esse conteúdo para o `jq` da seguinte maneira:
```
curl \
  --header "Authorization: Bearer ac4b1cabb9264079c4e9f2532804f81364c09b12" \
  --header "Content-Type: application/json" \
  --data '{ "long_url": "https://meleu.sh/elemento-presente-no-array/" }' \
  https://api-ssl.bitly.com/v4/shorten \
  | jq '.'
```

Isso vai gerar uma saída mais legível, desse tipo:
```
{
  "created_at": "2020-02-16T01:16:54+0000",
  "id": "bit.ly/2StrCQC",
  "link": "http://bit.ly/2StrCQC",
  "custom_bitlinks": [],
  "long_url": "https://meleu.sh/elemento-presente-no-array/",
  "archived": false,
  "tags": [],
  "deeplinks": [],
  "references": {
    "group": "https://api-ssl.bitly.com/v4/groups/bK311XKuDpS"
  }
}
```

Bem mais legível, certo?

Como podemos observar, a propriedade que nos interessa é a `link`, portanto vamos pegar somente ela. Desta forma:

```
curl \
  --header "Authorization: Bearer ac4b1cabb9264079c4e9f2532804f81364c09b12" \
  --header "Content-Type: application/json" \
  --data '{ "long_url": "https://meleu.sh/elemento-presente-no-array/" }' \
  https://api-ssl.bitly.com/v4/shorten \
  | jq '.link'
```
Isso vai gerar simplesmente:
```
"http://bit.ly/2StrCQC"
```

## Quinto Passo: lidando com URLs inválidas

E se o usuário quiser encurtar uma URL inválida?

Poderíamos criar uma RegEx para verificar isso de antemão, mas vamos pensar melhor: com certeza um o a API do bitly.com sabe lidar com isso!

Vamos ver o que acontece se passarmos uma url inválida:
```
curl \
  --header "Authorization: Bearer ac4b1cabb9264079c4e9f2532804f81364c09b12" \
  --header "Content-Type: application/json" \
  --data '{ "long_url": "IssoNãoÉUmaURL" }' \
  https://api-ssl.bitly.com/v4/shorten \
  | jq '.'
```
Isso vai gerar essa saída:
```
{
  "message": "INVALID_ARG_LONG_URL",
  "resource": "bitlinks",
  "description": "The value provided is invalid.",
  "errors": [
    {
      "field": "long_url",
      "error_code": "invalid"
    }
  ]
}
```

Neste JSON retornado, não existe a propriedade `link`. Além disso o conteúdo da propriedade `description` parece bastante apropriado para mostrar ao usuário quuando ele mandar uma URL inválida.

Portanto vamos usar o recurso `if-then-else` do `jq` e mostrar o `link` se estiver tudo certo, e mostrar `description` se tiver dado algum problema:
```
curl \
  --header "Authorization: Bearer ac4b1cabb9264079c4e9f2532804f81364c09b12" \
  --header "Content-Type: application/json" \
  --data '{ "long_url": "IssoNãoÉUmaURL" }' \
  https://api-ssl.bitly.com/v4/shorten \
  | jq 'if .link == null then .description else .link end'
```
Isso vai gerar:
```
The value provided is invalid.
```

Perfeito! Acho que já temos tudo que precisamos para transformar isso num script!


## Sexto Passo: transformando tudo isso num script

```
#!/usr/bin/env bash
# shortener.sh - encurtador de URLs

# TODO: o token abaixo é inválido! use o seu token!
readonly BITLY_TOKEN='ac4b1cabb9264079c4e9f2532804f81364c09b12'
readonly BITLY_ENDPOINT='https://api-ssl.bitly.com/v4/shorten'

shortener() {
  local long_url="$1"
  while [[ -z "$long_url" ]]; do
    read -r -p "Digite a url: " long_url
  done
  curl -s \
    --header "Authorization: Bearer ${BITLY_TOKEN}" \
    --header "Content-Type: application/json" \
    --data "{\"long_url\":\"${long_url}\"}" \
    "${BITLY_ENDPOINT}" \
    | jq -r 'if .link == null then .description else .link end'
}

shortener "$@"
```

## Fontes

- A mente brilhante por trás dessa solução é o colega [Robson Alexandre](https://github.com/robsonalexandre), que agraciou os participantes do nosso [grupo de telegram de shell script](https://t.me/shellscript_x) com uma versão que usa a versão 3 da API do bitly. Peguei o script dele como inspiração e atualizei aqui para usar a versão 4 da API.
- manpage do `curl`
- manpage do `jq`
- Usando `if-then-else` no `jq`: https://stedolan.github.io/jq/manual/v1.6/#ConditionalsandComparisons
- Página para praticar `jq`: https://jqplay.org/

