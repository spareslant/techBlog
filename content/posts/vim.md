---
title: "Vim"
date: 2019-09-01T20:24:26+01:00
draft: true
tags: ["vim"]
---

## motion
* `ctrl+e` => move screen down with moving cursor. `ctrl+y` => reverse
* `*` => search whole word under the cursor in file. `#` => search backwards
* `H M L`  => (50% screen scroll) High, Medium, Low cusror postion in current viewport of screen
* `ctrl+d` => 25% screen scroll along with cursor. `ctrl+u` => reverse
* `ctrl+f` => full screen scroll along with cursir. `ctrl+b` => reverse
* `g_` => move cursor to the end of line till first non-blank character.
* `gE` => move cursor to the end of previous word. special chars are not treated special.
* `50gg` => moves cursor to line 50 in normal mode.
* `:50` => moves cursor to line 50 in command mode
* `zz` => center the screen

## change
* in vim type `:h word-motions`
* `cit` => change text in-between tags
* `cf<char>` => search for a <char> in forward direction and deletes the whole text including char and place in insert mode. 
* `cF<char>` => same as above in backward direction.
* `ct<char>` => same as `cf<char>` but do not include the char.
* `cT<char>` => do not include char in backward.
* `~` => change the case of letter in normal mode
* `g-` => undo
* `g+` => redo
* `u` => undo
* `ctrl+r` => redo
* `e!<enter>` => undo all changes


## Delete in Insert mode.
* `ctrl+h` => deletes one char in backward direction
* `ctrl+w` => deletes a word in backward direction.
* `ctrl+u` => deletes till the start of line in backward direction.

## Delete leading white space
* `^` => move the cursor to the first char of line. then press `d0` to remove the white space

## Help
* :help i_^N  => insert-mode ctrl-n help
* :help c_^N  => command-mode ctrl-n help
* :help :g => command-mode g help
* :help g => normal-mode g help

## repeat actions from register in insert mode.
* in NORMAL mode, press`q`<a char> e.g. qw to start recording actions in `w` register. When finish come to normal mode and press `q` again to register all the actions in `w` register. In order to repeat the actions in saved in `w` register, following are the two methods.
   * In order to repeat all the actions in exactly same manner, go to normal mode and type `@w`
   * In order to paste all the chars in register, go to insert mode and type `ctrl+rw`

## Paste from registers
* `4yy` => copied text in normal mode, and to paste this text in Insert mode use `ctrl+r"`
* `:3,9y` => copies text in command-mode, and to paste this text in Insert mode use `ctrl+r"`

## Completion
* `ctrl+x ctrl+]` => Tag completion (ctags)
* `ctrl+x ctrl+F` => Filename completions
* `ctrl+x ctrl+o` => omni completions => `:pc` to close preview pane

## buffers
* `:ls` => list current buffers
* `:b <buffer number>` => jump to that buffer (file)

## Filters
* `:1,$ ! perl -wpl -e 's/Content/CONTENT/g'`

## system clipboard in normal mode
* `"*yy` => copies text in `*` register. now you can use system ctrl+v to paste outside vim.
* `"*dd` => similar to above

## system clipboard in command mode
* `:4,8y+` => copies the range of lines in `+` buffer which represents system clipboard
* `+` and `*` both represents system clipboard

## command mode without arrows
* In normal mode `q:` opens a vim window with the command history. You can work in it like any other vim buffer. Enter executes a command. 
* `q/` and `q?` do the same for searches.
* Also, while you are typing a command, you can press `Ctrl-f` to open the command-line window and continue editing the command there.

## Move lines.
* `:.m-3` => moves current line 2 lines above.
* `:5,7m 21` => moves line 5-7 after line 21

## command-mode commands
* `:reg` => list registers and its contents
* `:jumps` => list all jumps. (`ctrl+]` and `ctrl+o`)
* `changes` => list all changes
* `:@:` => Repeat last command run in command-mode

## marks
* `ma` => created a mark named `a` at a given line
* <`a> => move back to the marked position.

## tabs in insert mode
* `crtl+t` => insert indent in insert mode
* `crtl+d` => delete indent in insert mode

## line search repeat
* `fx` => searches for char `x` in line
* `;` => repeat the above search and `,` searches backwards
* `tx` => same as `fx` but cursor stops before `x`
* `Tx` => reverse of `tx`

## delete and yank from current postion till a match (flexible match)
* `y/pattern` => copied from current cursor postion till the pattern match
* `d/pattern` => delete from current cursor position till the pattern macth

## Join range of lines 
* `3J` => join three lines
* `4,5j` => join lines from 4 to 5

## Vim Dir explorer
* open vim and run `:Lexplore`, then `:vertical resize 40`
* optional settings: let g:netrw_browse_split = <number>

## Paste a command output in the file
* `:read ! ls -l` => will paste the output of ls -l command into current cursor position

## vscode change change escape key binding
* in vscode => settings => vim: Handle Keys => Add following
```
"vim.insertModeKeyBindingNonRecursive" : [
   {
      "before": ["§"],
      "after": ["<ESC>"]
   }
],
```
