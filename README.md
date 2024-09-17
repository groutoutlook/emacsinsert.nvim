# EmacsInsert.nvim

Readline motions and deletions in Neovim.

## Supported Readline commands

| Readline command             | RL shortcut | EmacsInsert.nvim function / Vim command |
| ---------------------------- | ----------- | --------------------------------------- |
| `kill-line`                  | `C-k`       | `kill_line`                             |
| `backward-kill-line`         | `C-u`       | `backward_kill_line`                    |
| `kill-word`                  | `M-d`       | `kill_word`                             |
| `backward-kill-word`         | `M-BS`      | `backward_kill_word`                    |
| `unix-word-rubout`           | `C-w`       | `unix_word_rubout`                      |
| `delete-char`                | `C-d`       | `<Delete>`                              |
| `backward-delete-char`       | `C-h`       | `<BS>`                                  |
| `beginning-of-line`          | `C-a`       | `beginning_of_line`                     |
| `back-to-indentation`        | `M-m`       | `back_to_indentation`                   |
| `end-of-line`                | `C-e`       | `end_of_line`                           |
| `forward-word`               | `M-f`       | `forward_word`                          |
| `backward-word`              | `M-b`       | `backward_word`                         |
| `forward-char`               | `C-f`       | `<Right>`                               |
| `backward-char`              | `C-b`       | `<Left>`                                |
| `next-line`                  | `C-n`       | `<Down>`                                |
| `previous-line`              | `C-p`       | `<Up>`                                  |
| `transpose-chars`            | `C-t`       | [Ask if desired][issues]                |
| `transpose-words`            | `M-t`       | [Ask if desired][issues]                |
| `quoted-insert`              | `C-v`       | `<C-v>`                                 |
| `yank` (called `put` in Vim) | `C-y`       | [Ask if desired][issues]                |
| `yank-pop`                   | `M-y`       | [Ask if desired][issues]                |
| `undo`                       | `C-_`       | [Ask if desired][issues]                |
| `upcase-word`                | `M-u`       | [Ask if desired][issues]                |
| `downcase-word`              | `M-l`       | [Ask if desired][issues]                |
| `capitalize-word`            | `M-c`       | [Ask if desired][issues]                |

## WIP

| Emacs command        | Emacs shortcut | EmacsInsert.nvim function |
| -------------------- | -------------- | ------------------------- |
| `backward-paragraph` | `C-Up`         | `backward_paragraph`      |
| `forward-paragraph`  | `C-Down`       | `forward_paragraph`       |

References: GNU docs for [moving](https://www.gnu.org/software/bash/manual/html_node/Commands-For-Moving.html), [text](https://www.gnu.org/software/bash/manual/html_node/Commands-For-Text.html), and [killing](https://www.gnu.org/software/bash/manual/html_node/Commands-For-Killing.html); and [Readline on Wikipedia](https://en.wikipedia.org/wiki/GNU_Readline).

## Supported Emacs commands

| Emacs command                        | Emacs shortcut | EmacsInsert.nvim function |
| ------------------------------------ | -------------- | ------------------------- |
| [`back-to-indentation`][emacsindent] | `M-m`          | `back_to_indentation`     |

[emacsindent]: https://www.gnu.org/software/emacs/manual/html_node/emacs/Indentation-Commands.html

## Do-what-I-mean commands

The following functions can be bound like `vim.keymap.set('!', '<C-a>', require 'readline'.dwim_beginning_of_line)`. Inspired by [mwim.el](https://github.com/alezost/mwim.el).

- `dwim_beginning_of_line`
  - Cycle the cursor through the following locations:
    - (Used only if the current line is a single-line comment with no other code, for example a line that begins with `//` in C or JavaScript, with `#` in Python, or with `--` in Lua.) The first character after the comment leader.
    - The first non-whitespace character on the line.
    - Column 0.
  - For example, calling this function repeatedly would move the cursor (`|`) like this:
    ```
        # Hello, wo|rld!
        # |Hello, world!
        |# Hello, world!
    |   # Hello, world!
        # |Hello, world!
    ```
- `dwim_backward_kill_line`
  - Similar to `dwim_beginning_of_line`, but kills instead of moving the cursor, and does not cycle. If the cursor is to the right of the comment leader on an EOL-comment line, kills the comment text left of the cursor, then the comment leader, then the whitespace left of the cursor, then rolls to the previous line.

## Configuring word characters

Readline motions that operate a-word-at-a-time (`forward-word`, etc.) advance the cursor until arriving at a "word character", then advance until arriving at the first non-word character. Deletion-by-word works similarly. The default set of word characters in Readline consists of [letters and digits](https://www.gnu.org/software/bash/manual/html_node/Commands-For-Moving.html).

What should count as a "word character" can be context-dependent: for example, in C, one may wish to take the word characters to be the alphanumeric characters (`abc…zABC…Z0123…9`) and the underscore (`_`) – those characters that can be used in C variable/symbol names.

This plugin allows you to configure what counts as a word character in two ways:

- You can set the buffer variable `readline_word_chars` to a string consisting of the word characters for the current buffer.
  - `readline.alphanum` is provided for convenience, and consists of the characters `abc…zABC…Z0123…9`.
  - For example, `vim.b.readline_word_chars = readline.alphanum .. '_-'` will cause alphanumerics, hyphens, and underscores to be treated as word characters in the current buffer.
- You can set `readline.word_chars.YOUR_FILETYPE = YOUR_STRING`.
  - For example, `readline.word_chars.bash = readline.alphanum .. '_-'` will cause alphanumerics, hyphens, and underscores to be treated as word characters when `filetype=bash`.
  - The value of `vim.b.readline_word_chars` takes precedence in a buffer (if it's set).

If neither of the above is set, we fall back to `readline.default_word_chars`, which you can set to whatever you like, but which by default is equal to `'abc…zABC…Z0123…9'`.

## Sample configs

### Maximal

This config creates Neovim mappings for all of the default Readline text-editing shortcuts, minus ones EmacsInsert.nvim doesn't (yet) have support for, for example `transpose-chars` and `capitalize-word`. You can copy-paste this block into your `init.lua` and delete the mappings you don't want.

```lua
local readline = require 'readline'
vim.keymap.set('!', '<C-k>', readline.kill_line)
vim.keymap.set('!', '<C-u>', readline.backward_kill_line)
vim.keymap.set('!', '<M-d>', readline.kill_word)
vim.keymap.set('!', '<M-BS>', readline.backward_kill_word)
vim.keymap.set('!', '<C-w>', readline.unix_word_rubout)
vim.keymap.set('!', '<C-d>', '<Delete>')  -- delete-char
vim.keymap.set('!', '<C-h>', '<BS>')      -- backward-delete-char
vim.keymap.set('!', '<C-a>', readline.beginning_of_line)
vim.keymap.set('!', '<C-e>', readline.end_of_line)
vim.keymap.set('!', '<M-f>', readline.forward_word)
vim.keymap.set('!', '<M-b>', readline.backward_word)
vim.keymap.set('!', '<C-f>', '<Right>') -- forward-char
vim.keymap.set('!', '<C-b>', '<Left>')  -- backward-char
vim.keymap.set('!', '<C-n>', '<Down>')  -- next-line
vim.keymap.set('!', '<C-p>', '<Up>')    -- previous-line
```

## Similar plugins

- [vim-rsi](https://github.com/tpope/vim-rsi)
- [readline.vim](https://github.com/ryvnf/readline.vim)
