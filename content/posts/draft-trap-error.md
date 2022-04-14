---
title: "Como detectar detectar precisamente onde seu script está quebrando"
description: >
  Use o 'trap' e poupe tempo quando precisar debugar o seu script. Você saberá a linha exata onde o problema ocorre.
tags:
  - draft
  - trap
  - boas-praticas
date: 2022-04-07T07:09:29-03:00
cover:
  image: "img/trap-error.png"
  alt: draft-trap-error
draft: true
---

Esse artigo é uma continuação do artigo anterior sobre como deixar o [bash mais rigoroso](bash-rigoroso).

No artigo anterior aprendemos como fazer o nosso script falhar o mais rápido possível e entedemos qual é a grande vantagem disso. Neste artigo veremos como obter uma indicação bem direta e precisa de onde o nosso script falhou.

## Recapitulando...

Só pra lembrar, no artigo anterior entendemos que devemos iniciar nossos scripts com:

```bash
set -euo pipefail
```

Pois queremos que o script:

- `set -e`: seja interrompido assim que ele falhar
- `set -u`: não tolere variáveis sem um valor explicitamente definido
- `set -o pipefail`: o "exit status" de uma pipeline seja o status do primeiro comando que falhar (ou sucesso)


### Como o `trap` funciona


### Variáveis úteis fornecidas pelo bash


### O código