# meleu.sh

Repositório por trás do <https://meleu.sh>

## Rodando localmente

```bash
# tem que ter o hugo instalado
brew install hugo

# clonar com o submodule do PaperMod
git clone --recursive git@github.com:meleu/meleudotsh.git
cd meleudotsh

# iniciando o servidor
hugo server
```

## Atualizar PaperMod

Dicas retiradas [da documentação oficial](https://adityatelange.github.io/hugo-PaperMod/posts/papermod/papermod-installation/) "Expand Method 2 - Git Submodule (recomended)"

```bash
# caso tenha esquecido de clonar com --recursive
git submodule update --init --recursive

# efetivamente atualizar o PaperMod theme
git submodule update --remote --merge
```
