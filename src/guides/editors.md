# Editor Support

>!N **AI-generated:** This page was written by AI (Claude) from the source code and tests of this fork of
>!N Emojicode, and has not been fully reviewed by a human. If something here disagrees with the compiler, the
>!N compiler is right.

Emojicode comes with a language server, `emojicode-lsp`, which uses the
compiler to tell you about errors while you type and to help you find your way
around your code. The `editors` directory of the
[compiler repository](https://github.com/DylanJones/emojicode/tree/master/editors)
contains support for Visual Studio Code and Neovim, which both use the language
server, and a tree-sitter grammar.

>!H The language server and the editor support are part of this fork of
>!H Emojicode. See [Building from Source](install.html#building-from-source)
>!H for how to get them.

## The Language Server

`emojicode-lsp` speaks the Language Server Protocol over its standard input and
output. You don’t run it yourself: your editor starts it. It is built together
with the compiler, and the installer copies it next to `emojicodec`, e.g. to
`/usr/local/bin`, so that editors find it on your `PATH`. In a build
directory, it is `LanguageServer/emojicode-lsp`.

The language server provides:

- **Errors and warnings as you type.** Whenever you stop typing for a quarter of
  a second, the server checks your code with the compiler’s parser and
  semantic analysis, using the unsaved contents of all files you have open, and
  shows the errors and warnings the compiler reports, including their notes.
- **Hover.** Hovering over an expression or variable shows its type. Hovering
  over a method, initializer or type shows its declaration and documentation.
- **Go to definition** of types, methods, initializers and variables, also into
  the interfaces of imported packages like the `s` package.
- **Semantic highlighting.** The server classifies every token, using the
  analysis to tell types, methods and variables apart.
- **Outline.** It lists the types of a file with their instance variables,
  initializers and methods.
- **Completion.** Completion only offers what can be written where the cursor
  is: at the start of a member of a type, for example, ❗️, 🆕 and the
  attributes, in a declaration types, and in code variables, types, methods
  and keywords. Type a word that describes what you want and pick it from the
  list: `grapes` offers 🍇, `append` offers the methods whose name or
  documentation contains it, `class` or `if` insert the whole construct, and
  variables complete by their name as usual.

### How Files Are Checked

The compiler always compiles one package, starting from its main file. The
language server therefore checks a file as part of the package whose main file
[includes](../reference/basics.html#including-other-source-code-files) it with
📜, directly or through other included files. As an include’s path is
relative to the including file, the server looks for including files near the
file, the nearest first: in the file’s directory, then in the directory above
and its subdirectories, and then in the directory above that and its
subdirectories up to two levels deep.

The file it arrives at is checked as a program, unless it and the files it
includes have no 🏁 block and export at least one type with 🌍. Then it is
checked as a package named like the file without its extension, e.g.
`catsimulator.emojic` as the package `catsimulator`. Interface files (`🏛` and
`.emojii` files), which you may open by going to a definition in an imported
package, are checked as the package of their directory.

### Package Search Paths

The language server looks for imported packages in these directories, in this
order:

1. The directories your editor passes in its `packageSearchPaths` setting (see
   below), which are like the compiler’s `-S` option.
2. The `packages` directory next to the package’s main file.
3. The directories in the environment variable `EMOJICODE_PACKAGES_PATH`, if
   set.
4. The default package search path, `/usr/local/EmojicodePackages/`, unless it
   was changed when the compiler was built.

Note that unlike the compiler, which looks in `./packages` relative to the
directory you run it in, the language server looks for `packages` next to the
main file. A language server that is run from a build directory also finds the
packages built in that directory. See
[Package Search Paths](../reference/compiler.html#package-search-paths) for how
the compiler searches.

## Visual Studio Code

The extension in `editors/vscode` registers the Emojicode language for `.emojic`,
`.🍇` and `.emojii` files and for interface files named `🏛`. It works on its
own and provides:

- Syntax highlighting, bracket matching, folding and comment toggling.
- Closing of what you open, like the editor does for braces in other languages:
  typing 🍇 in code adds 🍉, 🤜 adds 🤛, and 🐚 or 🍿 add 🍆. Typing 🔤, 📗 or
  📘 adds the one that ends the string or documentation comment. Nothing is
  added in strings and comments, and VS Code’s `editor.autoClosing…` settings
  apply.
- No warnings about U+FE0F, ➕ and ➖: VS Code usually highlights U+FE0F as an
  invisible character and ➕ and ➖ as characters that can be confused with `+`
  and `-`, but they are ordinary in Emojicode.

If it finds `emojicode-lsp`, the extension starts it and you get errors,
hover, go to definition (F12), semantic highlighting, the outline and
completion as described above. Otherwise it shows a warning and only
highlights your code.

### Installing the Extension

The extension is built from its source code in the repository, which requires
Node.js with npm, and needs VS Code 1.90 or newer. In `editors/vscode`, run:

```bash
npm install
npm run compile
npx vsce package
code --install-extension emojicode-0.1.0.vsix
```

Then make sure that `emojicode-lsp` is on your `PATH`, which it is after
installing Emojicode, or set `emojicode.server.path` to it.

To try the extension without installing it, open `editors/vscode` in VS Code
and press F5.

### Settings

| Setting | Default | Description |
| ------- | ------- | ----------- |
| `emojicode.server.path` | `emojicode-lsp` | The language server executable. A name without a path is looked up on the `PATH`. |
| `emojicode.packageSearchPaths` | `[]` | More directories to search for packages, like the compiler’s `-S` option. |
| `emojicode.trace.server` | `off` | Logs the messages between VS Code and the language server to the Emojicode output channel: `off`, `messages` or `verbose`. |
| `emojicode.learnerDocs.enabled` | `false` | Turns on learner docs (see below). |
| `emojicode.learnerDocs.baseUrl` | `https://www.emojicode.org/` | The documentation website that learner docs link to. |

Changing `emojicode.server.path` or `emojicode.packageSearchPaths` restarts the
language server.

### Learner Docs

If you’re learning Emojicode, learner docs link your code to this
documentation. Turn them on with the command **Emojicode: Toggle Learner
Docs**, by clicking *Learner docs* in the status bar or with the setting
`emojicode.learnerDocs.enabled`.

Hovering over code then shows a link to the section of the documentation that
explains it. The link depends on where the emoji is: 🔂 links to
[For In](../reference/controlflow.html#-for-in), ❗️ to
[Methods](../reference/classes-valuetypes.html#methods) where it declares a
method and to [Calling Methods](../reference/classes-valuetypes.html#calling-methods)
where it calls one, and a 😀 inside 🔤…🔤 to
[String Literals](../reference/literals.html#-string-literals), as it is just
text there. With the language server, methods and types of packages, like 😀
or 🔢, link to their page in the package documentation. The command
**Emojicode: Open Documentation for Token at Cursor** opens the link for the
token at the cursor directly.

To use a local copy of this documentation, for instance one that you’re
working on and serving with its `./serve` script, set
`emojicode.learnerDocs.baseUrl` to `http://localhost:8080/`.

## Neovim

`editors/nvim` adds filetype detection for Emojicode files, comments
(`💭`), jumping between 🍇 and 🍉 with `%`, indentation that follows 🍇 🍉 as
well as 🍿 and 🐚 with 🍆, and a configuration for the language server. It requires Neovim 0.11 or
newer.

### Installing the Plugin

Add the directory to your runtimepath, e.g. with lazy.nvim:

```lua
{ dir = '/path/to/emojicode/editors/nvim' }
```

or directly in your `init.lua`:

```lua
vim.opt.runtimepath:prepend('/path/to/emojicode/editors/nvim')
```

Then enable the language server:

```lua
vim.lsp.enable('emojicode')
```

With the language server, Neovim’s LSP support shows errors as you type,
highlights your code, shows types and documentation on hover (`K`), goes to
definitions (e.g. with `CTRL-]`) and lists the symbols of a file (`gO`).

For completion as you type with Neovim’s built-in completion, enable it when
the server attaches:

```lua
vim.api.nvim_create_autocmd('LspAttach', {
  callback = function(args)
    local client = vim.lsp.get_client_by_id(args.data.client_id)
    if client and client.name == 'emojicode' then
      vim.lsp.completion.enable(true, client.id, args.buf, { autotrigger = true })
    end
  end,
})
```

`emojicode-lsp` must be on your `PATH`. If it isn’t, or to search more
directories for packages, override the configuration:

```lua
vim.lsp.config('emojicode', {
  cmd = { '/path/to/build/LanguageServer/emojicode-lsp' },
  init_options = { packageSearchPaths = { '/path/to/packages' } },
})
```

### Tree-sitter

Without tree-sitter, highlighting comes from the language server alone, so it
appears once the server has checked the file. For highlighting as soon as a
file opens, folding by syntax and text objects, build the tree-sitter parser
from `editors/tree-sitter-emojicode` into the plugin’s directory. You need the
[tree-sitter CLI](https://github.com/tree-sitter/tree-sitter/tree/master/cli)
and a C compiler:

```bash
cd editors/tree-sitter-emojicode
tree-sitter generate
tree-sitter build -o ../nvim/parser/emojicode.so
```

The parser isn’t included in the repository and has to be generated inside it,
because `grammar.js` reads the emoji and keywords from the compiler’s
[formal grammar](../reference/syntax.html#the-formal-grammar).

The plugin starts tree-sitter highlighting when the parser is there. For
folding by syntax, add:

```lua
vim.api.nvim_create_autocmd('FileType', {
  pattern = 'emojicode',
  callback = function()
    vim.wo.foldmethod = 'expr'
    vim.wo.foldexpr = 'v:lua.vim.treesitter.foldexpr()'
    vim.wo.foldlevel = 99
  end,
})
```

With nvim-treesitter-textobjects, the text objects `@function`, `@class`,
`@conditional`, `@loop`, `@parameter`, `@call`, `@comment` and `@block` are
available.

## Other Editors

Any editor with support for the Language Server Protocol can use
`emojicode-lsp`: start it for Emojicode files, without arguments or with
`--stdio`. To pass more
package search paths, send them as `packageSearchPaths`, a list of
directories, in the `initializationOptions` of the `initialize` request.

The tree-sitter grammar in `editors/tree-sitter-emojicode` comes with queries
for highlighting, folding, indentation, text objects and local scopes, which
editors that use tree-sitter can build on. It is ported from the compiler’s
formal grammar and is more permissive than the compiler, but parses everything
the compiler accepts.
