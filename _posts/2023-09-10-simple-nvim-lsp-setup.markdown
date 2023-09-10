---
layout: post
title:  "Fast and simple neovim lsp setup"
date:   2023-09-10 11:20:00 +0300
categories: neovim nvim lsp language-server config dotfiles completion
---
Most new `nvim` users are usually buffled with LSP-setup, ending up installing a bunch of
bloated plugins, sometimes overapping in functionality, and then using only one-third of the features.

But to have nice LSP support with auto-completion you actually just need to read `:help
lsp-quickstart` to grasp the idea behind the built-in `lsp` in `neovim`. To save your time, I've made
this fast read on how to configure LSP in `nvim` with minimal overkill.

### Setting up LSP
To start using LSP, all you need to do is install the language-server itself, create `ftplugin`
sub-directory in `nvim` config directory, create a file called '<file_type>.lua':

```bash
$ mkdir $XDG_CONFIG_HOME/nvim/ftplugin
$ nvim $XDG_CONFIG_HOME/nvim/ftplugin/<file_type>.lua
```

with following content:

```lua
vim.lsp.start({
  name = 'lsp_name',
  cmd = {'language_server_cmd'},
  root_dir = vim.fs.dirname(vim.fs.find({'file_that_indicates_root_dir', 'another_one'}, { upward = true })[1]),
})
```

You can choose `lsp name` arbitrary, I'm just using common name for the `language-server` I'm
going to start.

If your `language-server` command line needs some arguments to be passed, just append them to the
dictionary (preten array in this case) `cmd` like this:

```
cmd = {'language_server_cmd', '-some', 'atgument', '-goes', 'here'}
```

The `root_dir` key is used to tell nvim where is root directory situated. You can use different
functions for that, expecially from `vim.fs` module, the main one being used the most is `find`.
See `:help vim.fs` for other available.

Here are some examples of configs I'm using in my setup:

`~/.config/nvim/ftplugin/c.lua`:
```lua
vim.lsp.start({
  name = 'clangd',
  cmd = {'clangd'},
  root_dir = vim.fs.dirname(vim.fs.find({'Makefile', 'main.c'}, { upward = true })[1]),
})
```

`~/.config/nvim/ftplugin/rust.lua`:
```lua
vim.lsp.start({
  name = 'rust-analyzer',
  cmd = {'rust-analyzer'},
  root_dir = vim.fs.dirname(vim.fs.find({'Cargo.toml', 'main.rs'}, { upward = true })[1]),
})
```

`~/.config/nvim/ftplugin/elixir.lua`:
```lua
vim.lsp.start({
  name = 'elixir-ls',
  cmd = {'language_server.sh'},
  root_dir = vim.fs.dirname(vim.fs.find({ 'mix.exs' }, { upward = true })[1]),
})
```

`~/.config/nvim/ftplugin/erlang.lua`:
```lua
vim.lsp.start({
  name = 'erlang_ls',
  cmd = {'erlang_ls'},
  root_dir = vim.fs.dirname(vim.fs.find({'rebar.config', 'src/'}, { upward = true })[1]),
})
```

You got the idea:).

After you configure lsp that way, and open file with corresponding `file_type` you already can use
lsp-completion on `onmi-menu` (i.e. `C-X C-O` by default).

If you want to enable auto-completion, read the next section

### Auto-completion
I'm using `lazy.nvim` plugin manager to setup plugins, after `packer.nvim` was discontinued. You
need to add:

```lua
  { 'hrsh7th/cmp-nvim-lsp' },
  { 'hrsh7th/nvim-cmp' },
```

to your plugins list, restart `nvim` to setup them, and you have yourself a nice lsp
auto-completion ready to go.
