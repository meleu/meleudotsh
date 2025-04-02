---
title: draft-pup
description:
tags:
  - draft
  - unix-moderno
date: 2022-04-24T11:42:36-03:00
cover:
  image: "img/draft-pup.png"
  alt: draft-pup
draft: true
---

> [!important]
> Mudar para htmlq

Falar sobre o `pup`

https://github.com/ericchiang/pup

usar como exemplo cotação do dolar

```bash
# dolarhoje():  gets, from dolarhoje.com, the dollar value
#               compared with brazilian real
# dependencies:
# - lynx: sudo apt install lynx
# - pup: https://github.com/ericchiang/pup
dolarhoje() {
  local currency="$1"
  local htmlFile='/tmp/dolarhoje.html'

  curl --fail -sL "dolarhoje.com/${currency}" \
    | pup 'div#cotacao' > "${htmlFile}" 2> /dev/null \
    && lynx -dump "${htmlFile}" \
    | sed 's/\(_\|hoje\)//g'
}
```

