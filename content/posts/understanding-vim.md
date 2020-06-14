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
* `ctrl+f` => full screen scroll along with cursor. `ctrl+b` => reverse
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
* :help map-modes => all modes
* :help :, => command mode , help

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

## reduce multiple blank lines to one blank lime
* `:38,$s!\n\+!^M^M!g`  => ^M can be entered by <ctrl-v><ctrl-enter>
* `:38,$s/\n\+/\r\r/g` => same as above

## Copy line
* `:21t-1` => copies line at line-no:21 and paste it in current position.

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

## indent a code
* `=i{` => indent a code inside {} pair excluding '{' and '}' characters.
* `=a{` => indent a code inside {} pair including '{' and '}' characters.
* `:38,$g/oo/>` => indent all those lines in line range from 38 till end, that matches a pattern "oo"
* `:38,$g/oo/>>>>` => indent 4 times all those lines in line range from 38 till end, that matches a pattern "oo" 
* `vjj=` => re-indent in visual style
* `5,10 norm ==` => re-indent using ex-command mode.
* `:s/\s\+//g | norm ==` => remove white-space between words and reindent it

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
* `:4,5j` => join lines from 4 to 5

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
      "before": ["j", "k"],
      "after": ["<ESC>"]
   }
],
```
## Window operations
* `ctrl+w =` => resizes all windows equally
* `ctrl+w p` => move back to previous window
* `ctrl+w <HJKL>` => move windows
* https://vimhelp.org/windows.txt.html#window-resize

## VIM IDE
* start vim
* Open dir browser on left =>  `:Lexplore`
* resize dir browser => `:vertical resize 30` or `ctrl+w [-+]`
* open new file in new split window when `Enter` is pressed on a file in dir window => `:let g:netrw_browse_split = 2`
* switch to new window => `ctrl+w w`
* create a new file in vertical split => `:vs afile.txt`
* open terminal => `:term`
* Bring terminal down => `ctrl+w J`
   * Enter terminal Normal mode => `ctrl+w N`
   * to adjust height of terminal => `ctrl+w :resize 40`
   * `ctrl+w |` => maximizes width of window
   * `ctrl+w =` => restores all window sizes to equal proportions.

https://vimhelp.org/pi_netrw.txt.html
https://vimhelp.org/windows.txt.html#window-resize
https://vimhelp.org/terminal.txt.html

## set filetype
* :set ft=sh  => sets filetype to a shell script

## To delete all lines from a file matching a pattern
* `:g/profile/d`
* `:g!/profile/d` => reverse of above command.

## get a range of lines (6-8) from another file
* `:read ! perl -wnl -e 'if ($. >= 6 &&  $. <= 8) {print}' second.rs`

## repeat a word multiple times
* `<esc>20iyahoo<esc>`

## egrep and egrep -v
* `:g/found/v/notfound/{cmd}`

## operation on first occurence of match
* `:g/yahoo/.norm nwi_zzz` => append _zzz at the `end of every word` containing yahoo (just first occurence)

* `:g/yahoo/.norm 10nwi_zzz` => repeat nwi_zzz 10 times on a line. 

## operation on all occurence of match
* `:%s/yahoo/&_zzz/g` => append _zzz at the `end of every matching word` containing yahoo

## operations on all the lines matching a pattern
* `:g/yahoo/t21`  => copy all lines matching yahoo to line 21 onwards
* `:g/yahoo/m21`  => move all lines matching yahoo to line 21 onwards

## operations on a range of lines matching a pattern
* `:186,209g/^set statusline/.norm 0iyahoo`
* `:12,17g/omitempty/m8` => In line range 12,17 move all lines having omitempty to line 8
* `38,46g/oo/ s/some/SOME/g` => in the line range 38,46 find all lines with "oo" pattern and replace some with SOME in those found lines.
* `:38,$  w /tmp/yahoo` => copy a range of lines to a file (creates the file if doesn't exis)
* `:38,$g/oo/  . w! >> /tmp/yahoo` => copy lines to file /tmp/yahoo in line range from 38 till end of file which has matched pattern of "oo"
* `/pattern1/,/pattern2/ {ex-cmd}` => operation on the range of lines.
* `:38,$g/./s/^/#/g` => from line 38 till end, comment all lines but do not comment empty line.
* `:10,16g/omit/ | s/json/JSON/` 

## insert repeated chars
* `30i*<esc>` => inserts 30 * chars
* `30i<space><esc>` => inserts 30 spaces
* `:normal! 50iabc <ctrl-v><esc>` => inserts `abc ` 50 times. After abc<space> press ctrl-v and then escacpe to insert escape. 

## ex-command index
* http://vimdoc.sourceforge.net/htmldoc/vimindex.html#ex-cmd-index

## search and editing
* http://vimdoc.sourceforge.net/htmldoc/usr_10.html#10.4

## Search and perform an operation on all searches
* `/yahoo`  => search for something first
* `qqq` => clear q register. 1st q --> starts recording, 2nd q --> name of the reg ro record into, 3rd q ---> stop recording. Now `q` register is empty. we could have used `qaq` as well, in this case register is `a`
* `qq` => start recording actions in q register. Now type `eai_zzz<esc>n`. This is going to append _zzz to the end word containing the string `yahoo`. type q again to stop recording. <esc> can be entered by pressing ctrl-v + <esc>
* now press @q to run the macro on next search. Keep in repeating @q in order to apply macro on next search. After pressing @q first time, you can repeat @q by using @@.
* NOTE: you do not need to press `n` to move to next search. However you can press n to skip search. 

## put quotes around a word using macro
* `qqq` => clear q register
* go to a word by pressing w. Now press `qq` to start recording macro. Now type `ebi"<esc>ea"<esc>`. This will insert quotes around a word. now press q to stop recording macro
* now go to any word which you want to surround with quotes and press @q. To perform the same action on another word you can keep on typing @@.