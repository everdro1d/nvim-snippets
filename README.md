# nvim-snippets

A simple snippet engine that allows vscode style snippets to be used with native neovim `vim.snippet`.
Also comes with support for [friendly-snippets](https://github.com/rafamadriz/friendly-snippets).

## Features

- Supports vscode style snippets
- Uses `vim.snippet` under the hood for snippet expansion
- Builtin suppert for:
  - [friendly-snippets](https://github.com/rafamadriz/friendly-snippets)
  - Native neovim completion (requires 0.12+)

## Requirements
- Requires neovim >= 0.12
- (optional) [nvim-cmp](https://github.com/hrsh7th/nvim-cmp) for completion support
- (optional) [friendly-snippets](https://github.com/rafamadriz/friendly-snippets) for pre-built snippets

## Installation

Using [lazy.nvim](https://github.com/folke/lazy.nvim)

```lua
{
    "everdro1d/nvim-snippets",
    dependencies = { "rafamadriz/friendly-snippets" }, -- optionally add friendly-snippets
},

```

Native Completion Example.

```lua
require('snippets').setup({
    create_cmp_source = false,

    create_autocmd = true,
    create_native_completion = true,
    native_completion_kind = 'Snippet',

    friendly_snippets = true,

    -- Use package.json file OR name child directories after filetypes
    -- (original author) https://www.reddit.com/r/neovim/comments/188js80/comment/kbn9f3b/
    search_paths = {
        vim.fn.stdpath('config') .. '/snippets'
    },
})

-- add native snippet completion
vim.opt.completefunc = "v:lua.nvim_snippets_complete"
```

Keybinds Config Example

```lua
-- next snippet part or show completion if not in snippet
vim.keymap.set({ "i", "s" }, "<C-l>", function()
    if vim.snippet.active({ direction = 1 }) then
        vim.schedule(function()
            vim.snippet.jump(1)
        end)
    end
    -- Open completion menu
    vim.api.nvim_feedkeys(vim.api.nvim_replace_termcodes("<C-x><C-u>", true, true, true), "i", false)
end, { silent = true, desc = "jump to next snippet part or open completion" })

-- go to previous snippet part
vim.keymap.set({ "i", "s" }, "<C-h>", function()
    if vim.snippet.active({ direction = -1 }) then
        vim.schedule(function()
            vim.snippet.jump(-1)
        end)
    end
end, { silent = true, desc = "go back a snippet part" })

```

<detail>
<summary>Legacy Config Example</summary>

```lua
{
  "everdro1d/nvim-snippets",
  keys = {
    {
      "<Tab>",
      function()
        if vim.snippet.active({ direction = 1 }) then
          vim.schedule(function()
            vim.snippet.jump(1)
          end)
          return
        end
        return "<Tab>"
      end,
      expr = true,
      silent = true,
      mode = "i",
    },
    {
      "<Tab>",
      function()
        vim.schedule(function()
          vim.snippet.jump(1)
        end)
      end,
      expr = true,
      silent = true,
      mode = "s",
    },
    {
      "<S-Tab>",
      function()
        if vim.snippet.active({ direction = -1 }) then
          vim.schedule(function()
            vim.snippet.jump(-1)
          end)
          return
        end
        return "<S-Tab>"
      end,
      expr = true,
      silent = true,
      mode = { "i", "s" },
    },
  },
}
```

</detail>

<detail>
<summary>With minimal <code>hrsh7th/nvim-cmp</code> and <code>rafamadriz/friendly-snippets</code> setup:</summary>

```lua
{
  "hrsh7th/nvim-cmp",
  dependencies = {
    "hrsh7th/cmp-nvim-lsp",
    "hrsh7th/cmp-buffer",
    "hrsh7th/cmp-path",
    "hrsh7th/cmp-cmdline",
    "rafamadriz/friendly-snippets",
    {
      "everdro1d/nvim-snippets",
      create_cmp_source = true,
      friendly_snippets = true,
    },
  },
  config = function()
    local cmp = require("cmp")
    cmp.setup({
      snippet = {
        expand = function(args)
          vim.snippet.expand(args.body)
        end,
      },
      mapping = cmp.mapping.preset.insert({
        -- Recommended keymap.
        ["<C-b>"] = cmp.mapping.scroll_docs(-4),
        ["<C-f>"] = cmp.mapping.scroll_docs(4),
        ["<C-Space>"] = cmp.mapping.complete(),
        ["<C-e>"] = cmp.mapping.abort(),
        ["<CR>"] = cmp.mapping.confirm({ select = true }),
      }),
      sources = cmp.config.sources({
        { name = "nvim_lsp"},
        { name = "snippets" }
      }, {
        { name = "buffer" }
      })
    })
  end
},
```

</detail>

## Configuration Options

| Option                  | Type        | Default                                     | Description           |
--------------------------|-------------|---------------------------------------------|------------------------
create_autocmd            | `boolean?`  | `false`                                     | Optionally load all snippets when opening a file. Only needed if not using [nvim-cmp](https://github.com/hrsh7th/nvim-cmp).
create_cmp_source         | `boolean?`  | `true`                                      | Optionally create a [nvim-cmp](https://github.com/hrsh7th/nvim-cmp) source. Source name will be `snippets`.
create_native_completion  | `boolean?`  | `false`                                     | Optionally create a native completion function for snippets. Function name will be `nvim_snippets_complete`.
native_completion_kind    | `string?`   | `'Snippet'`                                 | The completion kind to use for the native completion function.
friendly_snippets         | `boolean?`  | `false`                                     | Set to true if using [friendly-snippets](https://github.com/rafamadriz/friendly-snippets).
allowed_filetypes         | `string[]?` | `nil`                                       | Passed as `FileType` autocommand pattern (`*` if `nil`) to restrict the set of filetypes. Sometimes it can be more convenient than finding what filetype should be separately ignored (think about noice, mini.notify, etc.).
ignored_filetypes         | `string[]?` | `nil`                                       | Filetypes to ignore when loading snippets.
extended_filetypes        | `table?`    | `nil`                                       | Filetypes to load snippets for in addition to the default ones. `ex: {typescript = {'javascript'}}`
global_snippets           | `string[]?` | `{'all'}`                                   | Snippets to load for all filetypes.
search_paths              | `string[]`  | `{vim.fn.stdpath('config') .. '/snippets'}` | Paths to search for snippets.

## Example Snippet

```json
{
  "Say hello to the world": {
    "prefix": ["hw", "hello"],
    "body": "Hello, ${1:world}!$0"
  }
}
```
