## Set up your environment

To productively get going with Deno you should set up your environment. This
means setting up shell autocomplete, environmental variables and your editor or
IDE of choice.

### Environmental variables

There are several env vars that control how Deno behaves:

`DENO_DIR` defaults to `$HOME/.cache/deno` but can be set to any path to control
where generated and cached source code is written and read to.

`NO_COLOR` will turn off color output if set. See https://no-color.org/. User
code can test if `NO_COLOR` was set without having `--allow-env` by using the
boolean constant `Deno.noColor`.

### Shell autocomplete

You can generate completion script for your shell using the
`deno completions <shell>` command. The command outputs to stdout so you should
redirect it to an appropriate file.

The supported shells are:

- zsh
- bash
- fish
- powershell
- elvish

Example (bash):

```shell
deno completions bash > /usr/local/etc/bash_completion.d/deno.bash
source /usr/local/etc/bash_completion.d/deno.bash
```
Example [(windows ubuntu wsl)](https://github.com/denoland/deno/issues/5777#issuecomment-726268305)
```shell
mkdir -p ~/.completions
deno completions bash >~/.completions/deno.bash
echo "[[ -d ~/.completions && -n $(ls -A ~/.completions) ]] && . ~/.completions/*" >>~/.bashrc
source ~/.completions/*
```
Example (zsh without framework):

```shell
mkdir ~/.zsh # create a folder to save your completions. it can be anywhere
deno completions zsh > ~/.zsh/_deno
```

then add this to your `.zshrc`

```shell
fpath=(~/.zsh $fpath)
autoload -Uz compinit
compinit -u
```

and restart your terminal. note that if completions are still not loading, you
may need to run `rm ~/.zcompdump/` to remove previously generated completions
and then `compinit` to generate them again.

Example (zsh + oh-my-zsh) [recommended for zsh users] :

```shell
mkdir ~/.oh-my-zsh/custom/plugins/deno
deno completions zsh > ~/.oh-my-zsh/custom/plugins/deno/_deno
```

After this add deno plugin under plugins tag in `~/.zshrc` file. for tools like
`antigen` path will be `~/.antigen/bundles/robbyrussell/oh-my-zsh/plugins` and
command will be `antigen bundle deno` and so on.

Example (Powershell):

```shell
deno completions powershell > $profile
.$profile
```

This will be create a Powershell profile at
`$HOME\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1` by default,
and it will be run whenever you launch the PowerShell.

### Editors and IDEs

Because Deno requires the use of file extensions for module imports and allows
http imports, and most editors and language servers do not natively support this
at the moment, many editors will throw errors about being unable to find files
or imports having unnecessary file extensions.

The community has developed extensions for some editors to solve these issues:

#### VS Code

The beta version of [vscode_deno](https://github.com/denoland/vscode_deno) is
published on the
[Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=denoland.vscode-deno).
Please report any issues.

#### JetBrains IDEs

Support for JetBrains IDEs is available through
[the Deno plugin](https://plugins.jetbrains.com/plugin/14382-deno).

For more information on how to set-up your JetBrains IDE for Deno, read
[this comment](https://youtrack.jetbrains.com/issue/WEB-41607#focus=streamItem-27-4160152.0-0)
on YouTrack.

#### Vim and NeoVim

Vim works fairly well for Deno/TypeScript if you install
[CoC](https://github.com/neoclide/coc.nvim) (intellisense engine and language
server protocol).

After CoC is installed, from inside Vim, run`:CocInstall coc-tsserver` and
`:CocInstall coc-deno`. To get autocompletion working for Deno type definitions
run `:CocCommand deno.types`. Optionally restart the CoC server `:CocRestart`.
From now on, things like `gd` (go to definition) and `gr` (goto/find references)
should work.

#### Emacs

Emacs works pretty well for a TypeScript project targeted to Deno by using a
combination of [tide](https://github.com/ananthakumaran/tide) which is the
canonical way of using TypeScript within Emacs and
[typescript-deno-plugin](https://github.com/justjavac/typescript-deno-plugin)
which is what is used by the
[official VSCode extension for Deno](https://github.com/denoland/vscode_deno).

To use it, first make sure that `tide` is setup for your instance of Emacs.
Next, as instructed on the
[typescript-deno-plugin](https://github.com/justjavac/typescript-deno-plugin)
page, first `npm install --save-dev typescript-deno-plugin typescript` in your
project (`npm init -y` as necessary), then add the following block to your
`tsconfig.json` and you are off to the races!

```json
{
  "compilerOptions": {
    "plugins": [
      {
        "name": "typescript-deno-plugin",
        "enable": true, // default is `true`
        "importmap": "import_map.json"
      }
    ]
  }
}
```

#### LSP clients

Deno has builtin support for the
[Language server protocol](https://langserver.org) as of version 1.6.0 or later.

If your editor supports the LSP, you can use Deno as a language server for
TypeScript and JavaScript.

The editor can start the server with `deno lsp`.

##### Example for Kakoune

After installing the [`kak-lsp`](https://github.com/kak-lsp/kak-lsp) LSP client
you can add the Deno language server by adding the following to your
`kak-lsp.toml`

```toml
[language.deno]
filetypes = ["typescript", "javascript"]
roots = [".git"]
command = "deno"
args = ["lsp"]
```

##### Example for Vim/Neovim

After installing the [`vim-lsp`](https://github.com/prabirshrestha/vim-lsp) LSP
client you can add the Deno language server by adding the following to your
`vimrc`/`init.vim`:

```vim
if executable("deno")
  augroup LspTypeScript
    autocmd!
    autocmd User lsp_setup call lsp#register_server({
    \ "name": "deno lsp",
    \ "cmd": {server_info -> ["deno", "lsp"]},
    \ "root_uri": {server_info->lsp#utils#path_to_uri(lsp#utils#find_nearest_parent_file_directory(lsp#utils#get_buffer_path(), "tsconfig.json"))},
    \ "allowlist": ["typescript", "typescript.tsx"],
    \ "initialization_options": {
    \     "enable": v:true,
    \     "lint": v:true,
    \     "unstable": v:true,
    \   },
    \ })
  augroup END
endif
```

##### Example for Sublime Text

- Install the [Sublime LSP package](https://packagecontrol.io/packages/LSP)
- Install the
  [TypeScript package](https://packagecontrol.io/packages/TypeScript) to get
  syntax highlighting
- Add the following `.sublime-project` file to your project folder

```json
{
  "settings": {
    "LSP": {
      "deno": {
        "command": [
          "deno",
          "lsp"
        ],
        "initializationOptions": {
          // "config": "", // Sets the path for the config file in your project
          "enable": true,
          // "importMap": "", // Sets the path for the import-map in your project
          "lint": true,
          "unstable": false
        },
        "enabled": true,
        "languages": [
          {
            "languageId": "javascript",
            "scopes": ["source.js"],
            "syntaxes": [
              "Packages/Babel/JavaScript (Babel).sublime-syntax",
              "Packages/JavaScript/JavaScript.sublime-syntax"
            ]
          },
          {
            "languageId": "javascriptreact",
            "scopes": ["source.jsx"],
            "syntaxes": [
              "Packages/Babel/JavaScript (Babel).sublime-syntax",
              "Packages/JavaScript/JavaScript.sublime-syntax"
            ]
          },
          {
            "languageId": "typescript",
            "scopes": ["source.ts"],
            "syntaxes": [
              "Packages/TypeScript-TmLanguage/TypeScript.tmLanguage",
              "Packages/TypeScript Syntax/TypeScript.tmLanguage"
            ]
          },
          {
            "languageId": "typescriptreact",
            "scopes": ["source.tsx"],
            "syntaxes": [
              "Packages/TypeScript-TmLanguage/TypeScriptReact.tmLanguage",
              "Packages/TypeScript Syntax/TypeScriptReact.tmLanguage"
            ]
          }
        ]
      }
    }
  }
}
```

If you don't see your favorite IDE on this list, maybe you can develop an
extension. Our [community Discord group](https://discord.gg/deno) can give you
some pointers on where to get started.
