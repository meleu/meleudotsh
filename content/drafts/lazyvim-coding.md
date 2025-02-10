
No [artigo anterior](/lazyvim/) falei da configuração básica que faço logo após uma instalação do [LazyVim](https://lazyvim.org/). Nesse artigo mostro como o configuro para desenvolvimento.

Como ultimamente eu tenho trabalhado com Ruby on Rails, falarei um pouco enviesado para essa stack (além do bash, claro). Mas tenho certeza que o artigo será útil até se você trabalha com outras linguagens.

## Coisas úteis para saber logo de cara

### Folding

Muitas vezes, quando estamos trabalhando num código grande e complexo, com muitos blocos de código e indentação aninhada, é útil usar o recurso de "folding" para diminuir a quantidade de código exibido.

Esses são os comandos relacionados a folding que vem configurado por default no LazyVim:

- `zM` - fecha todos os folds
- `zR` - abre todos os folds
- `za` - toggle fold under cursor
- `zA` - toggle all folds under cursor
- `zi` - toggle folding

Uma coisa que uso muito quando num arquivo desses é:

1. `zM` pra fechar todos os folds
2. vou abrindo um nível `za`, um a um até chegar no bloco do meu interesse.
3. quando chego onde quero aperto `zA` pra abrir o bloco todo.


TODO: adicionar gif?

### Navegando no código

- `<leader>ss` - ⭐ pule rapidamente pra um método que você tem aberto no seu buffer
- `<c-o>`, `<c-i>` and gd to navigate the code
- `<leader>bp` - pin buffers
- `<leader>bP` - delete all non pinned buffers
- Add TODOs in files you want to work on in future but don't need now and delete their buffers, git will track them



### Reiniciando LSP

Se por um acaso você tiver a impressão que o Language Server não está funcionando, tente um `:LspStart` ou `:LspRestart` que isso pode resolver (OBS.: lembro de fazer isso no passado, ultimamente não tenho precisado fazer isso).

### LazyExtras

- Instale o LazyExtra da linguagem que você trabalha. TODO: falar mais aqui.

- `ui.treesitter-context`: mantém o contexto do código no topo. TODO: quero aumentar a quantidade de níveis de contexto.

---

## configs

### `.bats` as shell scripts

This is important to have `shfmt` and `shellcheck` even when working on `bats` files.

Put this at the end of `lua/config/options.lua`

```lua
vim.filetype.add({
  -- consider BATS files as shell scripts
  extension = { bats = "sh" },

  -- consider Dangerfile as ruby code
  filename = { ["Dangerfile"] = "ruby" },
})

```


### disable json and yaml "renderization"

**NOTE**: we can also disable `conceallevel` on demand with `<leader>uc`

`lua/config/autocmds.lua`:

```lua
-- Disable the concealing in some file formats
-- This "conceal" thing omits quotes.
vim.api.nvim_create_autocmd({ "FileType" }, {
  pattern = { "json", "jsonc", "yaml" },
  callback = function()
    vim.wo.conceallevel = 0
  end,
})
```

---

## LazyExtras

#### SQL

![[database queries from inside neovim]]

#### GitHub Copilot

<https://www.lazyvim.org/extras/coding/copilot>

`:LazyExtras` -> `coding.copilot`

I also configured a way to enable/disable copilot in `lua/plugins/copilot.lua`:

```lua
return {
  "zbirenbaum/copilot.lua",
  -- TODO:
  -- - find a way to toggle copilot
  -- - disable for markdown and text files

  keys = {
    { "<leader>cpd", ":Copilot disable<cr>", desc = "GitHub Copilot Disable" },
    { "<leader>cpe", ":Copilot enable<cr>", desc = "GitHub Copilot Enable" },
  },
}
```

---

## external plugins

### treesj

A very satisfying plugin to split blocks with breaklines lines!

```lua
return {
  "Wansmer/treesj",
  keys = {
    {
      "<leader>ct",
      ":TSJToggle<cr>",
      desc = "Toggle Treesitter Split",
    },
  },
  cmd = { "TSJToggle", "TSJSplit", "TSJJoin" },
  opts = { use_default_keymaps = false },
}
```

### Open in GitHub

OBS.: isso já é oferecido pelo LazyVim

Useful to open the current project/file in the browser (or simply copy the URL to the clipboard)

`lua/plugins/openingh.lua`

```lua
return {
  "Almo7aya/openingh.nvim",
  -- tip from here:
  -- https://github.com/Almo7aya/openingh.nvim/issues/24#issuecomment-2212536651
  init = function()
    vim.g.openingh_copy_to_register = true
  end,
  keys = {
    -- open in browser
    {
      "<leader>gBr",
      ":OpenInGHRepo<cr>",
      desc = "Open Repo on Browser",
    },
    {
      "<leader>gBf",
      ":OpenInGHFile<cr>",
      desc = "Open File on GitHub",
    },
    {
      "<leader>gBf",
      ":OpenInGHFileLines<cr>",
      desc = "Open Lines on GitHub",
      mode = "v",
    },

    -- yank to clipboard
    {
      "<leader>gyr",
      ":OpenInGHRepo+<cr>",
      desc = "Yank Repo URL",
    },
    {
      "<leader>gyf",
      ":OpenInGHFile+<cr>",
      desc = "Yank File URL",
    },
    {
      "<leader>gyf",
      ":OpenInGHFileLines+<cr>",
      desc = "Yank Lines URL",
      mode = "v",
    },
  },
}

```

---

## references

- [[LazyVim for Ambitious Developers]]
- [5 Neovim Plugins To Improve Your Productivity (video)](https://www.youtube.com/watch?v=NJDu_53T_4M)
