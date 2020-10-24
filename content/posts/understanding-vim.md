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
* `zz` => center the window with cursor on text
* `zt` => top of the window with cursor on text
* `zb` => bottom  of the window with cursor on text
* To move paragraphs => { and } or `{ and `}
* To move to start of end of highlisted text (in visual mode): `< and `>
* To move to a marked postion: `<marked char> (to mark: m<aChar>).
* To move between the changes done in file: `g;` and `g,`
* To move between the jumps done in file: `ctrl+o` and `ctrl+i`
* To move the cursor to its last postion when file was closed: `"

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
* :help ^W => help for windows,tab, switching between tabs and windows
* :help key-codes => list of key-codes for special keys like <ESC>, <Up> etc

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
* `:edit /tmp/yahoo | r! ls -l `

## Filters
* `:1,$ ! perl -wpl -e 's/Content/CONTENT/g'`

## launch vim with terminal
* run command => `vim +term`. This opens two horizontally stacked windows with terminal opened in upper window.
* `ctrl+w w` => swith to other unwanted window
* `ctrl+w q` => close this unwanted window

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
* To change width => `ctrl+w 5<` or `ctrl+w 6>`
* To change height => `ctrl+w 5+` or `ctrl+w 6-`

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
* `ctrl+w q`  => closes window
* `<num>ctrl+w _` => make current window height to max
* `<num>ctrl+w |` => make currrent window width to max
* `<num>ctrl+w =` => make all windows of equal size.
* `<num>ctrl+w [+-]` => window resize vertically
* `<num>ctrl+w [<>]` => window resize horizontally.
* `z<number><Enter> => resizes current window vertically as per <number>`
* Go to explorer winow using `ctrl+w w` and press `i` to change view. Keep on pressing i to see different viees.
* `set splitbelow` to open new window below the current
* `set splitrigt` to open new window on the right of the current window.
* change file(buffer) in current window => `b! <buffer number>`
* `:only` => close all other buffers and open only the current one

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

## Highlight all the trailing whitespaces
* /\s\+$<Enter>

## ex-command index
* http://vimdoc.sourceforge.net/htmldoc/vimindex.html#ex-cmd-index

## search and editing
* http://vimdoc.sourceforge.net/htmldoc/usr_10.html#10.4

* Remove trailing white spaces for lines
* `:59,60s/\s\+$//gc`

* create folds
* `zf{motion}` => `zf}` => creates fold of a paragraph
* `8zf` => creates fold of 8 lines
* `:3,52fold` => creates fold from lines 3 till 52
* `za` => toggles open and close folds
* `zE` => eleminiate all folds
* `zO` => open all folds

* spell suggesstion
* `z=`

## Search and perform an operation on all searches
* `/yahoo`  => search for something first
* `qqq` => clear q register. 1st q --> starts recording, 2nd q --> name of the reg ro record into, 3rd q ---> stop recording. Now `q` register is empty. we could have used `qaq` as well, in this case register is `a`
* `qq` => start recording actions in q register. Now type `eai_zzz<esc>n`. This is going to append _zzz to the end word containing the string `yahoo`. type q again to stop recording. <esc> can be entered by pressing ctrl-v + <esc>
* now press @q to run the macro on next search. Keep in repeating @q in order to apply macro on next search. After pressing @q first time, you can repeat @q by using @@.
* NOTE: you do not need to press `n` to move to next search. However you can press n to skip search. 

## Search and replace a whole word in Normal mode
* use `*` or `g*` to take current cursor word in search register. `*` => matches only whole word. `g*` matches word in any pattern.
* pressing `*` or `g*` moves you to the next match. To go back to previous location press `#` or `g#`.
* once back in original position. use `cw` to change the word. press <ESC> and press `n` to search for next match and then press `.` (dot). keep on pressing `n` followed by `.`(dot) to replace all the occurrences.

## Search and replace any pattern in visual/normal mode
* search the word first using `/wordtosearch`
* press `n` and go to that word or `N` to go backward.
* press `v` to go in visual mode and select pattern using motion keys like `l,h,w,b` etc.
* now press `c` to replace the word and press <ESC>
* press `n` to go to next word and press `.` (dot) and replace the word

## Search and replace in visual mode
* select the block of text with `ctrl+v`
* press <ESC>
* `:%s/\%Vpattern/replacedPatterm/g`
* Above will replace only in visually selected area.
* after pressing <ESC>, do this to search in visually selected area `/\%Vpattern`

## put quotes around a word using macro
* `qqq` => clear q register
* go to a word by pressing w. Now press `qq` to start recording macro. Now type `ebi"<esc>ea"<esc>`. This will insert quotes around a word. now press q to stop recording macro
* now go to any word which you want to surround with quotes and press @q. To perform the same action on another word you can keep on typing @@.

## put quotes around a word using visual selection
* `qqq` => clear q register
* go to a word by pressing w. press `v` to start character visual mode.
* keep on pressing `w` to keep on selecting words. or `l` to select character-wise. you can use arrows keys as well.
* once you have selected the text. press `qq` to start recording in `q` register.
* press <esc>
* press `> to go the end of selection. press `a` to go in insert mode. insert double quote i.e "
* press <esc>
* press `< to go to start of selection. press `i` to go in insert mode. insert double quote i.e "
* press <esc>
* press `> to go back to the end of selection. press `ll` to move cursor to the end of selection. 
* press `q`again to stop recording.

* now go to any other position in file. visually select the text. press <esc> and then `@q` to apply quotes to selected text.

## split all lines of an english paragraph in its own lines separately
* `:s/\./\.<ctrl+v><ctrl+ENTER>/gc` 

## split a line in its individual words on its own separate lines
* `s/\(\w\+\)/\1<ctrl+v><ctrl+ENTER>/gc`

## VIM shell compile/run edit cycle
* open file => `vim some.py`
* run the python script => `:e! /tmp/output-1 | r! python3 #` => This command will print output/error into a new buffer (file) called /tmp/yahoo and swicthes to that buffer.
* open python buffer in separate window => `vertical sb #` => without vertical keyword it will split horizontally.
* to list buffers => `:ls`
* to switch buffer in current active window => `b! <number>`
* run python script again with output in different buffer => `e! /tmp/output-2 | r! python3 #` => This makes /tmp/outout-2 buffer to replace some.py window
* Bring back some.py in current window: `b! #`
* To hide a buffer => `:hide`
* To delete a buffer => `:ls` and then `:bd! <number>`
* Go to other window => `ctrl+w w`
* open /tmp/output-2 buffer horizontally in this window => `:sb 3`
* switch to some.py window and run python script again with output going to /tmp/output-1 => `:e! /tmp/output-1 | r! python3 #` => This replaces current some.py window with lastest output
* bring back some.py => `b! #`

* following can also be used and should be much simpler
* `:vsplit  /tmp/some-new | r! python3 -m json.tool #` => opens /tmp/some-new in new vertical window. You can keep on repeating above command. and it will open a new window every time. You can close the previous window by switching to it using `ctrl+w w` and then `:hide`
* you can list all buffers with `:ls`
* you can open a buffer in current window using `:b! <number>`
* you can split window on either right or left using `:set splitright!` or `:set splitright`

## Register operations:
* in normal-mode: `"ayy` => copies current line in register `a`
* in normal-mode: `"ap`  => paste the contents of register `a` below the current line
* in insert-mode: `ctrl+r a` => paste the contents of register in current position.
* `"` is the default register.
* `:` is register which stores commands entered in ex-command mode.
* in normal-mode: `@a` => will execute the contents of register.
* in normal-mode: `@:` => will execute the contents of register `:`, hence commands entered in command-mode will be executed.(last ex-command)

## repeat register operations
* start recording a macro in normal mode.
  * `qaq` => clears register `a`
  * `qa`  => starts recording key-strokes in register `a`
  * `d/:` => deletes till `:` found in line of text.
  * `q`   => stops recording.
* go to ex-command mode:
  * `:.,+5 normal @a` => delete text till `:` found in all lines starting from current and till next 5 lines.

## How to cut the content in json:
Following is the json
```
"devDependencies": {
    "@types/diff": "4.0.2",
    "@types/diff-match-patch": "1.0.32",
    "@types/glob": "7.1.3",
    "@types/lodash": "4.14.157",
    "@types/mocha": "8.0.0",
    "@types/node": "12.12.31",
    "@types/sinon": "9.0.4",
    "@types/vscode": "1.37.0",
    "clean-webpack-plugin": "3.0.0",
}
```
we want to cut the column before `:` and paste is somewhere else.
* we will be using two registers here (namely `a` and `b`). One (`a`) will record the actions and other (`b`) will record the cut content.
* `qaq` and `qbq` => clear the contents of register `a` and `b`
* place the cursor on line 2.
* `qa` => start recording the action.
* make movement adjustment first before cutting/yanking the content. i.e press `^` to bring cursor where the line starts.
* now press `"B` to keep the record of cut contents. Please note that we are using `B` (capital B) here. Capital named register allows to append into existing register `b`. If we use `"b`, then on next cut, contents of `b` will be lost. Please also note that contents will eventually go into `b` register if you use either `"B` abd `"b`. Using `B` just indicates that we do not want the contents in register `b` to be overwritten.
* now press `d/:<enter>` => This will delete the content till `:` found in the line of text. As we have specified `"B` in previous step, these contents will go in `b` register and append mode.
* press `q` to stop recording actions in register `a`
* go to ex-command-mode and run following
* `:.+1,+5 normal @a` => This will apply the actions from current line and next 5 lines and append the deleted content in register `b`.
* `:reg` => to check the content of all registers

* contents will now look like this now.
```
"devDependencies": {
    : "4.0.2",
    : "1.0.32",
    : "7.1.3",
    : "4.14.157",
    : "8.0.0",
    : "12.12.31",
    "@types/sinon": "9.0.4",
    "@types/vscode": "1.37.0",
    "clean-webpack-plugin": "3.0.0",
}
```

* Now go to the line where you want to paste the cut content.
* press `"bp` or `ctrl+r b`. I moved to the last line and now contents look like this.
```
"devDependencies": {
    : "4.0.2",
    : "1.0.32",
    : "7.1.3",
    : "4.14.157",
    : "8.0.0",
    : "12.12.31",
    "@types/sinon": "9.0.4",
    "@types/vscode": "1.37.0",
    "clean-webpack-plugin": "3.0.0",
}
"@types/diff""@types/diff-match-patch""@types/glob""@types/lodash""@types/mocha""@types/node"
``` 
* In order to put above one line content in different lines, run following command in ex-command mode. Make sure your cursor is on that line.
* `:. ! perl -wpl -e 's/(".+?")/$1\n/g' `
* or run this command => `:s/""/"<ctrl+v><ctrl+ENTER>"/g`
* now contents will look like this.
```
"devDependencies": {
    : "4.0.2",
    : "1.0.32",
    : "7.1.3",
    : "4.14.157",
    : "8.0.0",
    : "12.12.31",
    "@types/sinon": "9.0.4",
    "@types/vscode": "1.37.0",
    "clean-webpack-plugin": "3.0.0",
}
"@types/diff"
"@types/diff-match-patch"
"@types/glob"
"@types/lodash"
"@types/mocha"
"@types/node"
```
* Now if you wish you can put back this block back to its original position.
* `:set virtualedit=all`
* `ctrl-v ` and select the whole block and press `y` to copy or `d` to delete. I pressed `y`.
* Place the cursor at line 2 before `:` and press `p`. Contents will look like this now.
```
"devDependencies": {
    "@types/diff"            : "4.0.2",
    "@types/diff-match-patch": "1.0.32",
    "@types/glob"            : "7.1.3",
    "@types/lodash"          : "4.14.157",
    "@types/mocha"           : "8.0.0",
    "@types/node"            : "12.12.31",
    "@types/sinon": "9.0.4",
    "@types/vscode": "1.37.0",
    "clean-webpack-plugin": "3.0.0",
}
"@types/diff"
"@types/diff-match-patch"
"@types/glob"
"@types/lodash"
"@types/mocha"
"@types/node"
```
## FZF flow
* To List all files in current directory
* `Files`  `Files!` (Full screen, use shit-arrows to scroll in preview windows)
* To list git modified files
* `GFiles?` `GFiles!?` (Full screen mode.)
* Git status
* `Gstatus` 