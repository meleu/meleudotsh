---
title: "Configuração do LazyVim pós-instalação"
description: >
  Ajeitando o LazyVim pra ficar do meu jeito logo após uma nova instalação.
date: 2025-02-09T17:00:00-03:00
tags:
  - neovim
cover:
  image: "img/lazyvim-meleush.png"
  alt: "LazyVim dashboard"
---

Depois de aproximadamente 2 décadas usando [Vim](https://www.vim.org/), em 2023 eu me rendi ao [Neovim](https://neovim.io/).

Cheguei a gastar alguns dias tentando fazer aquele setup bacanudo e super customizado pra deixá-lo como uma IDE, mas percebi que isso **NÃO** é uma tarefa simples (tanto de se fazer quanto de se manter no longo prazo). Por fim decidi me render a uma "distribuição" neovim, especificamente a [LazyVim](https://lazyvim.org/).

Nesse artigo mostro como configuro o LazyVim logo após uma instalação, para deixá-lo de um jeito que eu fico mais produtivo.

## Avisos

- **NÃO** tenho aqui a intenção de evangelizar o uso do vim ou propagandear o quão produtivo você pode ser quando o usa.
- **NÃO** falo sobre instalação do neovim e/ou LazyVim! Para isso você terá que seguir as orientações da [documentação oficial](https://lazyvim.org/).
- **NÃO** descrevo detalhadamente o que cada plugin faz.
- Vou falar apenas das configurações que fiz logo após uma instalação do LazyVim.
- No momento da escrita desse artigo o LazyVim está na versão 14.11.
- Enquanto eu estiver usando o LazyVim no meu dia a dia, pretendo ir atualizando esse artigo quando julgar necessário.

## Keybindings úteis para saber logo de cara

### Arquivos e buffers

- `<leader><leader>` picker de arquivos (ele tem fuzzy finding)
- `<leader>,` - picker de buffers abertos
- `<leader>bd` - fecha o buffer atual

### Notificações

Pra quem já está acostumado com o vim/neovim "clássico" vai estranhar as notificações. Algumas vezes são informações úteis, outras vezes pode ser ruído. De qualquer forma acho útil já chegar sabendo como revisitar essas notificações quando necessário:

- `<leader>sna` - mostra todas notificações
- `<leader>snt` - mostra todas as notificações no "picker"
- `<leader>snl` - mostra a última notificação
- `<leader>snd` - limpa todas notificações visíveis na tela

## Primeiras configurações

**OBSERVAÇÃO**: sempre que eu me referir a algum caminho de arquivo aqui, assuma que é um caminho relativo ao `~/.config/nvim/`.

### manter meu `.vimrc`

Como uso o Vim "original" há 2 décadas, meu `.vimrc` já tem umas coisinhas. Para carregar essas configurações eu coloco isso no final do `lua/config/options.lua`

```lua
-- load my own "old" configs written in VimScript
if vim.fn.filereadable("~/.vimrc") then
  vim.cmd("source ~/.vimrc")
end
```

Claro que eu poderia converter tudo pra lua, mas até o momento não é algo que eu quis fazer.

### desabilitar plugins

Basta criar um arquivo chamado `lua/plugins/disabled.lua` e colocar nele uma table com o nome do plugin seguido de `enabled = false`.

No meu caso ficou assim:

```lua
return {
  -- disable mini.surround, (confusing keybindings)
  -- I have years of muscle memory using tpope/vim-surround
  { "echasnovski/mini.surround", enabled = false },
}
```

### instalação de plugins simples

Alguns plugins eu uso há anos e não consigo trabalhar sem eles. Para instalar esses plugins que não exigem configurações, eu crio um arquivo `lua/plugins/basic-plugins.lua`, com esse conteúdo:

```lua
return {
  "vim-scripts/ReplaceWithRegister",
  "tpope/vim-surround",
  "tpope/vim-repeat",     -- make vim-surround dot-repeatable
  "tpope/vim-speeddating", -- <C-a>/<C-x> to increase/decrease dates
}
```

**OBSERVAÇÃO**: o plugin `vim-scripts/ReplaceWithRegister` utiliza a combinação `gr` (mnemônico para **G**o **R**eplace), e isso conflita com a configuração default do LSP no Lazyvim, que usa `gr` para **G**oto **R**eference. Veja logo abaixo como desabilitar essa tecla para o LSP.

## Ajuste de keybindings

### dica: customização dos plugins "nativos"

Uma prática bacana que eu aprendi lendo o [LazyVim for Ambitions Developers](https://lazyvim-ambitious-devs.phillips.codes/): quando for customizar um plugin que já é instalado por padrão pelo LazyVim, criar um arquivo prefixado como `extended-`.

Eu gostei pois isso deixa claro na listagem de arquivos quais são configs de plugins "nativos" e quais são plugins que eu instalei por minha conta.

No exemplo a seguir, eu uso `lua/plugins/extended-lsp.lua`.

#### `gr` para "go replace"

Eu tenho anos de memória muscular usando `gr` como "**G**o **R**eplace" (graças ao consagrado plugin "ReplaceWithRegister"). Só que o comportamento padrão do LazyVim é usar `gr` para "**G**oto **R**eferences" do LSP.

Eu desabilito o comportamento default através dessa configuração no arquivo `lua/plugins/extended-lsp.lua`:

```lua
-- overriding LazyVim's default LSP configs
return {
  "neovim/nvim-lspconfig",
  opts = function()
    -- keymaps
    local keys = require("lazyvim.plugins.lsp.keymaps").get()

    -- disable gr, I want to use it for "Go Replace" (ReplaceWithRegister plugin)
    keys[#keys + 1] = { "gr", false }

    -- use gh to "hover documentation"
    keys[#keys + 1] = { "gh", vim.lsp.buf.hover, desc = "Hover" }
```

Observe que aqui 👆 no finalzinho do arquivo eu configurei `gh` para funcionar como "hover". Isso é útil quando queremos ver documentação de um método/função, ou algo do tipo.

### desabilitar H e L para navegar entre buffers

Apesar de reconhecer que usar H e L (maiúsculos) pode parecer "intuitivo" para navegar para o buffer que vem antes/depois, estas teclas já possuem um significado histórico no vim, e que eu já estou muito habituado a usar (inclusive em outras aplicações que emulam vim keybindings, como tmux e Obsidian).

- `H` (mnemônico pra High) vai pro topo da área visível do buffer atual
- `L` (mnemônico pra Low) vai pro final da área visível do buffer atual

Portanto, eu quero desabilitar o comportamento que vem no LazyVim.

No arquivo `lua/config/keymaps.lua` eu coloco isso:

```lua
vim.keymap.del("n", "<S-h>")
vim.keymap.del("n", "<S-l>")
```

### navegar entre buffers como se fossem abas

O conceito de abas (tabs) no vim é um pouco diferente do que estamos habituados a ver em outros aplicativos que usamos no dia a dia (browser, por exemplo). No entanto a UI do LazyVim é feita de forma a fazer com que os buffers abertos apareçam como se fossem as abas que estamos acostumados a ver. E eu gosto de navegar entre esses buffers usando `gt`/`gT`.

Eu adiciono isso no meu `lua/config/keymaps.lua`

```lua
vim.keymap.set("n", "gT", ":bprevious<cr>", { desc = "Prev buffer" })
vim.keymap.set("n", "gt", ":bnext<cr>", { desc = "Next buffer" })
```

### desabilitar `<enter>` para aceitar autosuggestions

Quando eu aperto `<enter>` eu espero inserir uma quebra de linha no texto, e não aceitar seja lá o que o autosuggestions esteja me oferecendo. Para desabilitar esse comportamento irritante, precisamos alterar a configuração de um plugin que já vem no LazyVim: o blink.cmp.

Eu adiciono isso no meu `lua/plugins/extended-blink.lua`:

```lua
return {
  "saghen/blink.cmp",
  opts = {
    keymap = {
      preset = "default",
    },
  },
}
```

## Configurações puramente visuais

As alterações que faço nessa seção são puramente visuais para se adequar as minhas preferências, e não alteram em nada as funcionalidades do editor.

Se você já está curtindo o visual do seu LazyVim, pode ignorar essa parte.

### desabilitar o relógio no lualine

O lualine é o plugin que deixa aquela barrinha bonitinha na parte de baixo da tela. O LazyVim coloca um relógio ali, e eu não curto isso (existem outros lugares da minha tela onde consigo ver as horas).

Para remover o relógio eu coloco isso no meu `lua/plugins/extended-lualine.lua`:

```lua
return {
  "nvim-lualine/lualine.nvim",
  opts = {
    sections = {
      lualine_z = {},
    },
  },
}
```

### habilitar bordas em "subjanelas"

O que estou chamando aqui de "subjanelas" são coisas como a interface do Mason, do próprio Lazy plugin manager, a documentação que aparece no "hover", os diagnósticos do linter, etc. Para não me confundir com o texto real que estou editando, eu gosto de ter esses widgets com bordas.

Veja esse exemplo mostrando um comparativo da interface do Lazy plugin manager com e sem borda:

![LazyVim com borda vs. sem borda](/img/lazyvim-with-border-comparison.png)

#### borda no Lazy.nvim

Para adicionar bordas basta editar o `lua/config/lazy.lua` e adicionar essa config de `ui` lá no final do `setup`, assim:

```lua
-- a lot of configs here...
require("lazy").setup({
  -- more configs here...
  ui = {
    border = "rounded",
  },
})
```

#### borda no Mason

`lua/plugins/extended-mason.lua`

```lua
return {
  "williamboman/mason.nvim",
  opts = {
    ui = {
      border = "rounded",
    },
  },
}
```

#### borda no sugestões de código

O plugin responsável pelo autosuggestions é o `blink.cmp`. Para colocar uma borda naquelas caixinhas de sugestões, basta adicionar [essas configurações](https://cmp.saghen.dev/recipes.html#border) no `lua/plugins/extended-blink.cmp.lua`:

```lua
return {
    "saghen/blink.cmp",
    opts = {
      -- other configs here...
      completion = {
        menu = { border = "single" },
        documentation = { window = { border = "single" } },
      },
      signature = { window = { border = "single" } },
    },
  }
```

#### bordas na documentação do "hover"

Quando estamos lendo um código e queremos ver a documentação de um determinado código usamos `K` (ou `gh`, se você está seguindo minha config). Isso é gerenciado pelo `noice`, e eu também gosto de ter bordas nessa janelinha de documentação. Pra isso basta adicionar isso no `lua/plugins/extended-noice.lua`:

```lua
return {
 "folke/noice.nvim",
 opts = {
   presets = {
     lsp_doc_border = true,
   },
 },
}
```

#### bordas nas caixas de diagnósticos

O plugin responsável por renderizar os diagnósticos do LSP na tela é o `nvim-lspconfig`, e para adicionar bordas nas janelinhas de diagnóstico adicionamos essa config no `lua/plugins/extended-lsp.lua`

```lua
return {
  "neovim/nvim-lspconfig",
  opts = function(_, opts)
    opts.diagnostics = {
      float = {
        border = "rounded",
      },
    }
  end,
}
```

## Casos de uso específicos

### vim-tmux-navigator

**OBSERVAÇÃO**: se você não usa tmux pode pular essa parte.

Há tanto tempo quanto uso o vim, eu também uso o [tmux](https://github.com/tmux/tmux). E para navegar facilmente entre painéis do tmux onde estou rodando o Vim/Neovim, eu uso o [vim-tmux-navigator](https://github.com/christoomey/vim-tmux-navigator).

Para isso eu crio um arquivo `lua/plugins/vim-tmux-navigator.lua` com esse conteúdo:

```lua
return {
  "christoomey/vim-tmux-navigator",
  cmd = {
    "TmuxNavigatorLeft",
    "TmuxNavigatorRight",
    "TmuxNavigatorUp",
    "TmuxNavigatorDown",
  },
  keys = {
    { "<c-h>", ":TmuxNavigatorLeft<cr>" },
    { "<c-j>", ":TmuxNavigatorDown<cr>" },
    { "<c-k>", ":TmuxNavigatorUp<cr>" },
    { "<c-l>", ":TmuxNavigatorRight<cr>" },
  },
}
```

Dessa forma eu consigo navegar "para fora" do Neovim usando uma combinação de `Ctrl` e as tradicionais teclas de movimento `hjkl`.

### Neovim dentro do VSCode

De vez em quando eu uso VSCode (seja por conta de alguma extensão que não tem equivalente no Neovim, ou para não ficar confundindo os colegas quando preciso parear e compartilhar tela).

Felizmente é possível usar o Neovim **dentro** do VSCode, graças à extensão [VSCode Neovim](https://github.com/vscode-neovim/vscode-neovim).

Outra coisa boa é que o LazyVim tem um "LazyExtra" que torna essa integração ainda melhor, pois podemos ativar/desativar comportamentos do Neovim que não queremos ter quando tivermos usando de dentro do VSCode.

Para isso precisamos ativar o `vscode` no `:LazyExtras`.

Conforme [mencionado na documentação](https://www.lazyvim.org/extras/vscode), existe um pequeno conjunto de plugins que estão habilitados quando rodamos o LazyVim por dentro do VSCode. E se quisermos plugins adicionais precisamos habilitar explicitamente usando `vscode = true`.

Portanto o meu `lua/plugins/basic-plugins.lua` acaba ficando assim:

```lua
return {
  { "vim-scripts/ReplaceWithRegister", vscode = true },
  { "tpope/vim-surround", vscode = true },
  { "tpope/vim-repeat", vscode = true }, -- make vim-surround dot-repeatable
  { "tpope/vim-speeddating", vscode = true }, -- <C-a>/<C-x> to increase/decrease dates
}
```

Eu também edito o `lua/config/keymaps.lua` para criar 3 seções:

- keymaps que rodam tanto Neovim do terminal quanto dentro do VSCode
- keymaps exclusivas do VSCode
- keymaps exclusivas do terminal

O arquivo fica tipo assim:

```lua
-- lua/config/keymaps.lua

----------------------------------------------------
--[ keymaps for both terminal and VSCode ]--
----------------------------------------------------

-- disable H and L to navigate between buffers.
-----------------------------------------------
-- reasoning: H and L have a default Vim behavior
-- and should never be overwritten
vim.keymap.del("n", "<S-h>")
vim.keymap.del("n", "<S-l>")

----------------------------------------------------
--[ VSCode-only keymaps go inside this if block, ]--
----------------------------------------------------
if vim.g.vscode then
  -- the "close current buffer" in VSCode is kinda buggy
  vim.keymap.del("n", "<leader>bd")

  -- disable terminal when in VSCode
  vim.keymap.del("n", "<C-/>")

  -- disable lazygit
  vim.keymap.del("n", "<leader>gg")

  return
end

-------------------------------------------
--[ terminal-only keymaps go below here ]--
-------------------------------------------

-- since LazyVim comes with bufferline, making buffers look
-- like tabs, I want to navigate between buffers with gt/gT.
vim.keymap.set("n", "gt", ":BufferLineCycleNext<CR>")
vim.keymap.set("n", "gT", ":BufferLineCyclePrev<CR>")
```

## Conclusão

Pra evitar de ficar um artigo muito longo, vou parando por aqui.

Mais pra frente faço outro artigo configurando o LazyVim com recursos que uso no meu dia a dia de trabalho como desenvolvedor.
