---
title: "cheat.sh: obtendo exemplos e macetes dos mais diversos comandos"
description: >
  Descubra como o cheat.sh vai facilitar sua vida te mostrando rapidamente vários macetes e exemplos de uso dos mais diversos comandos. Podendo inclusive ser acessado diretamente da linha de comando.
date: 2019-12-27T05:00:13-03:00
showToc: false
tags:
  - links
cover:
  image: "img/cheat-sh.png"
  alt: "exemplo de output do cheat.sh"
---

Recentemente conheci um site muito maneiro para os amantes da linha de comando: https://cheat.sh

E o que mais gostei dele é que conseguimos ter uma interação prazeirosa simplesmente acessando via `curl`.

Abra o seu terminal e experimente você mesmo com o seguinte comando:

```shell-session
curl cheat.sh
```

Trata-se de um site com um repositório com diversos macetes de comandos, linguagens de programação, algoritmos, etc.

Por exemplo, imaginemos que queremos ver alguns macetes do comando `sudo`. Basta executarmos um `curl cheat.sh/sudo`. Veja só:

```shell-session
$ curl cheat.sh/sudo
# sudo
# Execute a command as another user.

# List of an unreadable directory:
sudo ls /usr/local/scrt

# To edit a file as user www:
sudo -u www vi /var/www/index.html

# To shutdown the machine:
sudo shutdown -h +10 "Cya soon!"

# To repeat the last command as sudo:
sudo !!

# Save a file you edited in vim
:w !sudo tee > /dev/null %

# Make sudo forget password instantly
sudo -K

# List your sudo rights
sudo -l

# Add a line to a file using sudo
echo "foo bar" | sudo tee -a /path/to/some/file

# run root shell
sudo -i

# to disable password for sudo for user superuser add
# superuser ALL=(ALL) NOPASSWD:ALL
# in /etc/sudoers
```

Se preferir, também é possível ter a mesma informação acessando via browser. Veja você mesmo: [https://cheat.sh/sudo](https://cheat.sh/sudo)

Você pode também instalar um script específico para acessar o site. Veja as instruções de instalação diretamente no [README do repositório github dele](https://github.com/chubin/cheat.sh#command-line-client-chtsh).

Uma vez instalado o script (chamado de `cht.sh`), vc pode fazer consultas mais "inteligentes".

Imaginemos que você quer lembrar como fazer um _parse_ de JSON em JavaScript. Basta executar o comando a seguir que você verá algumas dicas:

```shell-session
cht.sh js parse json
```

![cheat-sh-json](/img/cheat-sh-json.png)

Certamente uma ferramenta que qualquer _CLI lover_ vai gostar de, no mínimo, experimentar!

## Fontes

- https://cheat.sh/
- https://github.com/chubin/cheat.sh
