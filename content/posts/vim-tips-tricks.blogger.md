---
title: "Vim Tips and Tricks"
date: 2019-09-01T20:24:26+01:00
draft: true
tags: ["vim", "fzf"]
---
<!--- Below style are also defined in static/css/my.css file.
They are repeatedly defined here so that pandoc can generate
the final HTML with all necessary css styles.
Note: draft: true above. This prevents publishing it to GitHUB.
--->
<style>
/* To highlight text in Green in pre tag */
.hl {color: #008A00;}
/* To highlight text in Bold Green in pre tag */
.hlb {color: #008A00; font-weight: bold;}
/* To highlight text in Bold Red in pre tag */
.hlbr {color:#e90001; font-weight: bold;}
/* <code> tag does not work in blogger. Use following class with span tag */
.code {
    color:#7e168d; 
    background: #f0f0f0; 
    padding: 0.1em 0.4em;
    font-family: SFMono-Regular, Consolas, "Liberation Mono", Menlo, Courier, monospace;
}
</style>

## motion

* `CTRL-e` => move screen down with moving cursor. `CTRL-y` => reverse
* `*` => search whole word under the cursor in file. `#` => search backwards
* `H M L`  => (50% screen scroll) High, Medium, Low cusror postion in current viewport of screen
* `CTRL-d` => 25% screen scroll along with cursor. `CTRL-u` => reverse
* `CTRL-f` => full screen scroll along with cursor. `CTRL-b` => reverse
* `g_` => move cursor to the end of line till first non-blank character.
* `gE` => move cursor to the end of previous word. special chars are not treated special.
* `50gg` => moves cursor to line 50 in `NORMAL` mode.
* `:50` => moves cursor to line 50 in `COMMAND` mode
* `zz` => center the window with cursor on text
* `zt` => top of the window with cursor on text
* `zb` => bottom  of the window with cursor on text
* To move among paragraphs => `{` and `}` or  ``  `{  `` and ``  `}  ``
* To move to start of end of highlisted text (in `VISUAL` mode): ``  `<  `` and ``  `>  ``
* To move to a marked postion: ``  `<marked char> `` (to mark: `` m<aChar>  ``).
* To move between the changes done in file: `g;` and `g,`
* To move between the jumps done in file: `CTRL-o` and `CTRL-i`
* To move the cursor to its last postion when file was closed: `` `"  ``


## change

* in vim type `:h word-motions`
* `cit` => change text in-between tags
* `cf<char>` => search for a <char> in forward direction and deletes the whole text including char and place in insert mode. 
* `cF<char>` => same as above in backward direction.
* `ct<char>` => same as `cf<char>` but do not include the char.
* `cT<char>` => do not include char in backward.
* `~` => change the case of letter in `NORMAL` mode
* `g-` => undo
* `g+` => redo
* `u` => undo
* `CTRL-r` => redo
* `e!<enter>` => undo all changes


## save and quit

* `ZZ` => Save and quit
* `ZQ` => Quit without saving


## Delete in Insert mode.

* `CTRL-h` => deletes one char in backward direction
* `CTRL-w` => deletes a word in backward direction.
* `CTRL-u` => deletes till the start of line in backward direction.


## Delete leading white space

* `^` => move the cursor to the first char of line. then press `d0` to remove the white space


## Help

* `:help i_^N`  => insert-mode `CTRL-n` help
* `:help c_^N`  => `COMMAND`-mode `CTRL-n` help
* `:help :g` => `COMMAND`-mode g help
* `:help g` => `NORMAL`-mode g help
* `:help map-modes` => all modes
* `:help :,` => `COMMAND` mode , help
* `:help ^W` => help for windows,tab, switching between tabs and windows
* `:help key-codes` => list of key-codes for special keys like `<ESC>`, `<Up>` etc
* `:help autocmd-events` => list of all events


## repeat actions from register in insert mode.

* in `NORMAL` mode, press`` q<achar> `` e.g. `qw` to start recording actions in `w` register. When finish come to `NORMAL` mode and press `q` again to register all the actions in `w` register. In order to repeat the actions in saved in `w` register, following are the two methods.
   * In order to repeat all the actions in exactly same manner, go to `NORMAL` mode and type `@w`
   * In order to paste all the chars in register, go to insert mode and type `CTRL-rw`


## Paste from registers

* `4yy` => copied text in `NORMAL` mode, and to paste this text in Insert mode use `CTRL-r"`
* `:3,9y` => copies text in `COMMAND`-mode, and to paste this text in Insert mode use `CTRL-r"`

## Completion

* `CTRL-x CTRL-]` => Tag completion (ctags)
* `CTRL-x CTRL-F` => Filename completions
* `CTRL-x CTRL-o` => omni completions => `:pc` to close preview pane
* `CTRL-x ctlr+p repeatedly` => To complete the lines from existing text


## buffers

* `:ls` => list current buffers
* `:b <buffer number>` => jump to that buffer (file)
* `:edit /tmp/yummy | r! ls -l `
* `:ba` => open all buffers in current window


## Filters

* `:1,$ ! perl -wpl -e 's/Content/CONTENT/g'`


## launch vim with terminal

* run command => `vim +term`. This opens two horizontally stacked windows with terminal opened in upper window.
* `CTRL-w w` => swith to other unwanted window
* `CTRL-w q` => close this unwanted window


## system clipboard in `NORMAL` mode

* `"*yy` => copies text in `*` register. now you can use system `CTRL-v` to paste outside vim.
* `"*dd` => similar to above


## system clipboard in `COMMAND` mode

* `:4,8y+` => copies the range of lines in `+` buffer which represents system clipboard
* `+` and `*` both represents system clipboard

## `COMMAND` mode without arrows

* In `NORMAL` mode `q:` opens a vim window with the command history. You can work in it like any other vim buffer. Enter executes a command. 
* `q/` and `q?` do the same for searches.
* Also, while you are typing a command, you can press `CTRL-f` to open the command-line window and continue editing the command there.


## Move lines (cut lines)

* `:.m-3` => moves current line 2 lines above.
* `:5,7m 21` => moves line 5-7 after line 21


## reduce multiple blank lines to one blank lime

* `:38,$s!\n\+!^M^M!g`  => `^M` can be entered by `` <CTRL-v><CTRL-ENTER> ``
* `:38,$s/\n\+/\r\r/g` => same as above


## Copy line

* `:21t-1` => copies line at line-no:21 and paste it in current position.


## `COMMAND`-mode commands

* `:reg` => list registers and its contents
* `:jumps` => list all jumps. (`CTRL-]` and `CTRL-o`)
* `changes` => list all changes
* `:@:` => Repeat last command run in `COMMAND`-mode


## marks

* `ma` => created a mark named `a` at a given line 
* ``  `a  `` => move back to the marked position.

## tabs in insert mode

* `crtl+t` => insert indent in insert mode
* `crtl+d` => delete indent in insert mode


## indent a code

* `=i{` => indent a code inside {} pair excluding '{' and '}' characters.
* `=a{` => indent a code inside {} pair including '{' and '}' characters.
* `:38,$g/oo/>` => indent all those lines in line range from 38 till end, that matches a pattern `oo`
* `:38,$g/oo/>>>>` => indent 4 times all those lines in line range from 38 till end, that matches a pattern `oo`
* `vjj=` => re-indent in `VISUAL` style
* `5,10 norm ==` => re-indent using ex-`COMMAND` mode.
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
* optional settings: `let g:netrw_browse_split = <number>`


## Paste a command output in the file

* `:read ! ls -l` => will paste the output of `ls -l` command into current cursor position


## vscode change escape key binding

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

* `CTRL-w =` => resizes all windows equally
* `CTRL-w p` => move back to previous window
* `CTRL-w <HJKL>` => move windows
* https://vimhelp.org/windows.txt.html#window-resize
* To change width => `CTRL-w 5<` or `CTRL-w 6>`
* To change height => `CTRL-w 5+` or `CTRL-w 6-`
* `CTRL-w s` => horizontal split
* `CTRL-w v` => vertical split
* `CTRL-w T` => move current window to new tab


## VIM IDE

* start vim
* Open dir browser on left =>  `:Lexplore`
* resize dir browser => `:vertical resize 30` or `CTRL-w [-+]`
* open new file in new split window when `ENTER` is pressed on a file in dir window => `:let g:netrw_browse_split = 2`
* switch to new window => `CTRL-w w`
* create a new file in vertical split => `:vs afile.txt`
* open terminal => `:term`
* Bring terminal down => `CTRL-w J`
   * Enter terminal `NORMAL` mode => `CTRL-w N`
   * to adjust height of terminal => `CTRL-w :resize 40`
   * `CTRL-w |` => maximizes width of window
   * `CTRL-w =` => restores all window sizes to equal proportions.
* `CTRL-w q`  => closes window
* `<num>CTRL-w` => make current window height to max
* `<num>CTRL-w |` => make currrent window width to max
* `<num>CTRL-w =` => make all windows of equal size.
* `<num>CTRL-w [+-]` => window resize vertically
* `<num>CTRL-w [<>]` => window resize horizontally.
* `z<number><ENTER> => resizes current window vertically as per <number>`
* Go to explorer winow using `CTRL-w w` and press `i` to change view. Keep on pressing `i` to see different viees.
* `set splitbelow` to open new window below the current
* `set splitright` to open new window on the right of the current window.
* change file(buffer) in current window => `b! <buffer number>`
* `:only` => close all other buffers and open only the current one

https://vimhelp.org/pi_netrw.txt.html

https://vimhelp.org/windows.txt.html#window-resize

https://vimhelp.org/terminal.txt.html


## set filetype

* `:set ft=sh`  => sets filetype to a shell script


## To delete all lines from a file matching a pattern

* `:g/profile/d`
* `:g!/profile/d` => reverse of above command.


## get a range of lines (6-8) from another file

* `:read ! perl -wnl -e 'if ($. >= 6 &&  $. <= 8) {print}' second.rs`


## repeat a word multiple times

* `<ESC>20iyummy<ESC>`


## egrep and egrep -v

* `:g/found/v/notfound/{cmd}`


## operation on first occurence of match

* `:g/yummy/.norm nwi_zzz`  appends `zzz` at the `end of every word` containing yummy (just first occurence)
* `:g/yummy/.norm 10nwi_zzz` => repeat nwi_zzz 10 times on a line. 


## operation on all occurence of match

* `:%s/yummy/&_zzz/g` appends `zzz` at the `end of every matching word` containing yummy


## operations on all the lines matching a pattern

* `:g/yummy/t21`  => copy all lines matching yummy to line 21 onwards
* `:g/yummy/m21`  => move all lines matching yummy to line 21 onwards


## operations on a range of lines matching a pattern

* `:186,209g/^set statusline/.norm 0iyummy`
* `:12,17g/omitempty/m8` => In line range 12,17 move all lines having omitempty to line 8
* `38,46g/oo/ s/some/SOME/g` => in the line range 38,46 find all lines with `oo` pattern and replace some with SOME in those found lines.
* `:38,$  w /tmp/yummy` => copy a range of lines to a file (creates the file if doesn't exis)
* `:38,$g/oo/  . w! >> /tmp/yummy` => copy lines to file /tmp/yummy in line range from 38 till end of file which has matched pattern of `oo`
* `/pattern1/,/pattern2/ {ex-cmd}` => operation on the range of lines.
* `:38,$g/./s/^/#/g` => from line 38 till end, comment all lines but do not comment empty line.
* `:10,16g/omit/ | s/json/JSON/` 


## insert repeated chars

* `30i*<ESC>` => inserts 30 * chars
* `30i<SPACE><ESC>` => inserts 30 spaces
* `:normal! 50iabc <CTRL-v><ESC>` => inserts `abc ` 50 times. After `abc<SPACE>` press `CTRL-v` and then `escacpe` to insert escape. 

## Highlight all the trailing whitespaces

* `/\s\+$<ENTER>`


## ex-`COMMAND` index

* http://vimdoc.sourceforge.net/htmldoc/vimindex.html#ex-cmd-index


## search and editing

* http://vimdoc.sourceforge.net/htmldoc/usr_10.html#10.4
* Remove trailing white spaces for lines
* `:59,60s/\s\+$//gc`


## create folds
* `zf{motion}` => `zf}` => creates fold of a paragraph
* `8zf` => creates fold of 8 lines
* `:3,52fold` => creates fold from lines 3 till 52
* `za` => toggles open and close folds
* `zE` => eleminiate all folds
* `zO` => open all folds

## spell suggesstion

* `z=`


## Search and perform an operation on all searches

* `/yummy`  => search for something first
* `qqq` => clear q register. 1st `q` --> starts recording, 2nd `q` --> name of the reg ro record into, 3rd `q` ---> stop recording. Now `q` register is empty. we could have used `qaq` as well, in this case register is `a`
* `qq` => start recording actions in `q` register. Now type `eai_zzz<ESC>n`. This is going to append `_zzz` to the end word containing the string `yummy`. type `q` again to stop recording. `<ESC>` can be entered by pressing `CTRL-v`  `<ESC>` or `CTRL-v` `CTRL-[`
* now press `@q` to run the macro on next search. Keep in repeating @q in order to apply macro on next search. After pressing `@q` first time, you can repeat `@q` by using `@@`.
* NOTE: you do not need to press `n` to move to next search. However you can press n to skip search. 


## Search and replace a whole word in `NORMAL` mode

* use `*` or `g*` to take current cursor word in search register. `*` => matches only whole word. `g*` matches word in any pattern.
* pressing `*` or `g*` moves you to the next match. To go back to previous location press `#` or `g#`.
* once back in original position. use `cw` to change the word. press <ESC> and press `n` to search for next match and then press `.` (dot). keep on pressing `n` followed by `.`(dot) to replace all the occurrences.


## Search and replace any pattern in `VISUAL`/`NORMAL` mode

* search the word first using `/wordtosearch`
* press `n` and go to that word or `N` to go backward. [This step is not required]
* press `v` to go in `VISUAL` mode and select pattern using motion keys like `l,h,w,b` etc.
* now press `c` to replace the word and press `<ESC>`
* press `n` to go to next word and press `.` (dot) and replace the word


## Search and replace a pattern (starting from existing position) in `VISUAL` mode - 2nd method

* press `v` to go in `VISUAL` mode, and start selecting pattern using motion keys like `l,h,w,b` etc. e.g `vEE` (to select two words that may have non-word chars)
* press `c` now to cut the content in `"` register and start in insert mode
* edit the text and when finished press `<ESC>` to go in `NORMAL` mode.
* press `/` to go in search mode and press `CTRL-r "` to paste the cut contents and press <ENTER>. This will move the cursor to the next searched pattern.
* now press `.` to replace the pattern
* press `n` to move to next search and press `.` to replace it again.


## Search and replace in `VISUAL` mode

* select the block of text with `CTRL-v`
* press `<ESC>`
* `:%s/\%Vpattern/replacedPatterm/g`
* Above will replace only in visually selected area.
* after pressing `<ESC>`, do this to search in visually selected area `/\%Vpattern`


## Search and replace in `VISUAL` mode - method-2

* select the block of text with `CTRL-v`
* press `c` (delete and insert)
* type the text you want
* press `<ESC>`. It will replace the text you typed on the very first selected line


## Search and replace in `VISUAL` mode - method-3

* select the block of text with `CTRL-v`
* press `r`
* type the character you want to fill `VISUAL` block with.
* press `<ESC>`. It will replace the entire `VISUAL` block the the single character only.


## put quotes around a word using macro

* `qqq` => clear q register
* go to a word by pressing `w`. Now press `qq` to start recording macro. Now type `ebi"<ESC>ea"<ESC>`. This will insert quotes around a word. now press `q` to stop recording macro
* now go to any word which you want to surround with quotes and press `@q`. To perform the same action on another word you can keep on typing `@@`.


## put quotes around a word using `VISUAL` selection

* `qqq` => clear q register
* go to a word by pressing w. press `v` to start character `VISUAL` mode.
* keep on pressing `w` to keep on selecting words. or `l` to select character-wise. you can use arrows keys as well.
* once you have selected the text. press `qq` to start recording in `q` register.
* press `<ESC>`
* press ``  `>   ``   to go the end of selection. press `a` to go in insert mode. insert double quote i.e `"`
* press `<ESC>`
* press ``  `<   `` to go to start of selection. press `i` to go in insert mode. insert double quote i.e `"`
* press `<ESC>`
* press ``  `>   `` to go back to the end of selection. press `ll` to move cursor to the end of selection. 
* press `q`again to stop recording.
* now go to any other position in file. visually select the text. press `<ESC>` and then `@q` to apply quotes to selected text.

## split all lines of an english paragraph in its own lines separately

* `:s/\./\.<CTRL-v><CTRL-ENTER>/gc` 

## split a line in its individual words on its own separate lines

* `s/\(\w\+\)/\1<CTRL-v><CTRL-ENTER>/gc`


## VIM shell compile/run edit cycle

* open file => `vim some.py`
* run the python script => `:e! /tmp/output-1 | r! python3 #` => This command will print output/error into a new buffer (file) called `/tmp/yummy` and swicthes to that buffer.
* open python buffer in separate window => `vertical sb #` => without vertical keyword it will split horizontally.
* to list buffers => `:ls`
* to switch buffer in current active window => `b! <number>`
* run python script again with output in different buffer => `e! /tmp/output-2 | r! python3 #` => This makes `/tmp/outout-2` buffer to replace some.py window
* Bring back some.py in current window: `b! #`
* To hide a buffer => `:hide`
* To delete a buffer => `:ls` and then `:bd! <number>`
* Go to other window => `CTRL-w w`
* open `/tmp/output-2` buffer horizontally in this window => `:sb 3`
* switch to `some.py` window and run python script again with output going to `/tmp/output-1` => `:e! /tmp/output-1 | r! python3 #` => This replaces current some.py window with lastest output
* bring back some.py => `b! #`

* following can also be used and should be much simpler
* `:vsplit  /tmp/some-new | r! python3 -m json.tool #` => opens `/tmp/some-new` in new vertical window. You can keep on repeating above command. and it will open a new window every time. You can close the previous window by switching to it using `CTRL-w w` and then `:hide`
* you can list all buffers with `:ls`
* you can open a buffer in current window using `:b! <number>`
* you can split window on either right or left using `:set splitright!` or `:set splitright`

## Register operations:


* in `NORMAL`-mode: `"ayy` => copies current line in register `a`
* in `NORMAL`-mode: `"ap`  => paste the contents of register `a` below the current line
* in insert-mode: `CTRL-r a` => paste the contents of register in current position.
* `"` is the default register.
* `:` is register which stores commands entered in ex-`COMMAND` mode.
* in `NORMAL`-mode: `@a` => will execute the contents of register.
* in `NORMAL`-mode: `@:` => will execute the contents of register `:`, hence commands entered in `COMMAND`-mode will be executed.(last ex-`COMMAND`)


## repeat register operations

* start recording a macro in `NORMAL` mode.
  * `qaq` => clears register `a`
  * `qa`  => starts recording key-strokes in register `a`
  * `d/:` => deletes till `:` found in line of text.
  * `q`   => stops recording.
* go to ex-`COMMAND` mode:
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
* now press `d/:<ENTER>` => This will delete the content till `:` found in the line of text. As we have specified `"B` in previous step, these contents will go in `b` register and append mode.
* press `q` to stop recording actions in register `a`
* go to ex-`COMMAND`-mode and run following
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
* press `"bp` or `CTRL-r b`. I moved to the last line and now contents look like this.

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

* In order to put above one line content in different lines, run following command in ex-`COMMAND` mode. Make sure your cursor is on that line.
* `:. ! perl -wpl -e 's/(".+?")/$1\n/g' `
* or run this command => `:s/""/"<CTRL-v><CTRL-ENTER>"/g`
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
* `CTRL-v ` and select the whole block and press `y` to copy or `d` to delete. I pressed `y`.
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


## FZF plugin flow

* To List all files in current directory
* `:Files`  `:Files!` (Full screen, use shit-arrows to scroll in preview windows)
* To list git modified files
* `:GFiles?` `:GFiles!?` (Full screen mode.)
* Git status
* `:Gstatus`


## Search and replace interactively in multiple files using quickfix.

* make sure `fzf.vim` and `ripgrep` is installed.
* Search for pattern and select all matched file in `quickfix` list.
* `:Rg pattern`  => now select all files and press `<ENTER>.` A `quickfix` window will open listing all the files
* now run following command to replace the pattern
* `:cfdo %s/pattern/NEW_PATTERN/gc | update`
* you can view `quickfix` list manually as well via `clist` or `copen` command.
* use `cnext` => to move to next match in quickfix window
* in `NORMAL` mode use `@:` to repeat last executing ex-`COMMAND`, ie. in this case :cnext
* After `@:`, use `@@` to keep repeating ex-`COMMAND` in `NORMAL` mode.
* If you want to remove entries from `quickfix` window then swith to quickfix window by pressing `CTRL-w w` and then `:set modifiable`. Now you can run `NORMAL` vim commands here, like `dd` or `:g/<pattern>/d`


## Search and replace interactively in multiple files using standard unix tools and quickfix

* `vim -q <(egrep -n -R yummy *) -c ':copen'` => This command will search for pattern `yummy` in all the files recursively in current directory and populate the `quickfix` list.
* now run following command to replace the pattern
* `:cfdo %s/pattern/NEW_PATTERN/gc | update`


## Search and replace interactively in multiple files by populating quickfix list directly

* `vim -c ':set errorformat+=%f | :cexpr system("ls") | copen'` => This populated `quickfix` list with the filenames in current directory. 
* Note: We are not using `vim -q` to populate `quickfix` and also we changed the `errorformat` so that plain file list can be navigated in `quickfix` list.
* now run following command to replace the pattern
* `:cfdo %s/pattern/NEW_PATTERN/gc | update`
 
## Search and replace interactively in multiple files by using args

* `vim * -c ':argdo %s/YAHOO/yummy/gc | update'` => This command will open all files in current directory and tries to replace text YAHOO with yummy interactively.

## replace a bunch of text with a copied text.

* Copy some text in vim first either using `VISUAL` mode or via `line1,line2y`
* `:75,79d | .normal k"0p` => This command will delete the contents between 75 and 79 lines and insert the text copied in previous step. Content copied in previous step may be bigger and larger than the deleted content and it fits nicely

## Save a VIM session

* `:mksession saved_session.vim`
* `vim -S header-files-work.vim` to restore the session. or use `:source saved_session.vim`


## Moving between VIM tabs

* `gt`
* `g<TAB>`


## Edit a pattern in a text file - example-1

Convert following line
```
           ["from", "person", "hobbies", "2", "0", "outdoor"]
```
into
```
inputPath = "/from/person/hobbies/2/0/outdoor"
```

* `:s/"//g | s/, /\//g | s/\[/"\//g | s/\]/"/g | .normal IinputPath = CTRLvCTRL[==CTRLvCTRL[` 
**Note**: `CTRLvCTRL[` => places <ESC> and  == is for indentation

## Edit a pattern in a text file - example-2
Convert following line
```
            , "Travel") 
```

into

```
inputPathValue = "Travel"
```

* `:s/, // | s /)//g | normal IinputPathValue = ^[<<....]`

**Note:** `<<` is for left indentation and `...` are repeating the last command


## Edit a pattern in a text file - example-3

Convert following line

```
             ["from", "mars", "plan", "locality"], "City")
```

into 

```
  inputPath = "/from/mars/plan/locality"
 	inputPathValue = "City"
```

* `.normal f]a^M^[k:s/"//g | s/, /\//g | s/\[/"\//g | s/\]/"/g | .normal IinputPath = ^[==^[j:s/, // | s /)//g | normal IinputPathValue = ^[<<....^[
`

## Using VIM as less

* cat `<somefile>` | vim -R -


## Minimal vim configuration

```
set nocompatible
set sw=2
set ts=2
set softtabstop=2
set hlsearch
set incsearch
set autoindent
set expandtab
set backspace=indent,eol,start
syntax on
filetype on
filetype plugin indent on
```

## invoke vi editor in bash command-line editing

* `export VISUAL=/usr/bin/vim` or `export EDITOR=vim`
* In `emacs` editing mode in zsh/bash press `CTRL-x CTRL-e` command line to invoke temp vim editor. 
* In `vi` editing mode in zsh/bash press v (two times) and the line will be opened in a temp file
* In `vi` editing mode in order to delete a word backword use `daW`and then use `.` command to repeat the deletion

## special keys can be used in vimrc in `COMMAND` mode in following format

* `<space>`
* `<c-d>`
* `<esc>`
* `<leader>`
* `<localleader>`
* `<cr>`
* `<nop>`
* `<buffer>`
* `<left>`
* `<cword>` => the word under the cursor
* `<cWORD>`

* `normal!` doesn't recognize "special characters" like `<cr>`. There are a number of ways around this, but the easiest to use and read is execute
* variables starting with an `@` are registers. `@@` is the "unnamed" register `:` the one that Vim places text into when you yank or delete without specify a particular register.


## create vertical line in a block of text using macro

We will place commented vertical linein the following block of rust code.

```rust
fn references_example_1() {
    println!("======== references_example_1 ========");
    let a = "yummy;
    let mut b = &a;                                                                                                                                                                                                 
    println!("a = {}, b = {}", a, b);
    // a = "hotmail";
    b = &"oracle";
    println!("a = {}, b = {}", a, b);

}
```

* place the cursor on line 3 (this where we want to start commenting)
* We shall store the action in a register named `z`.
* Go in `NORMAL` mode.
* press `qzq` to empty the register `z`.
* press `qz` to start recording in register `z`.
* Now press the following sequence of keys to place the first vertical line comment.
* `$50a<SPACE><ESC>45|d$a// |<ESC>j` => this will place // | at the current line at coloumn number 45 and move the cursor to next line.
* 50 is the `width of the max spanned row` + 2 in the paragraph.
* press `q` to stop recording.
* now press `@z` to place `// |` in current line. It will move the cursor to next line.
* now keep on pressing `@@` to keep entering `// |` in subsequent lines.
* final text will look like below.

```rust
fn references_example_1() {
    println!("======== references_example_1 ========");
    let a = "yummy;                         // |
    let mut b = &a;                         // |
                                            // |
    println!("a = {}, b = {}", a, b);       // |
    // a = "hotmail";                       // |
    b = &"oracle";                          // |
    println!("a = {}, b = {}", a, b);       // |

}
```

* Explanation of `$50a<SPACE><ESC>45|d$a// |<ESC>j`
   * `$` => moves the end of current line.
   * `50a<SPACE><ESC>` => inserts 50 spaces from the end of current line.
   * `45|` => moves the cursor to column 45. Note 45 is counted right from the left edge of the screen not from the end of line.
   * `d$` => from 45 column onwards delete any extra spaces introduced by `50a<SPACE><ESC>` command.
   * `a` => go in insert mode and start appending
   * `// |<ESC>` => the actual text to be inserted at column 45.
   * `j` => move the cursor to the next line.

### Edit the macro

* If we want to the place the `// |` at line `40` we can edit the macro with following steps.
* In the above step, we had created the macro in `z` register.
* Somewhere in empty place in vim, run the following command in `NORMAL` mode.
* `"zp` => paste the contents of `z` register in current cursor position. It will look like this => `$50a ^[<80><fd>a45|d$a// |^[j`
* Edit the `45` number to your desired number.
* press `v` to go in `VISUAL` mode and keep pressing `l` till the end (upto `j`).
* press `"zy` => to copy the highlighted text in `z` register.
* Now you can use `@z` to insert `// |` at column `45` in the lines.
* you can delete the line where you pasted the content from macro.

### Place another vertical line on the left of existing line.

* Above we inserted `// |` at column `45`. Now we want to insert `|| /` at `39` at the left this vertical wall
* We will be recording sequence in `q` macro.
* press `qqq` to empty the `q` register.
* press `qq` again to start recording in `q` register.
* Now go to line `3` again and press following key sequences.
* `39|R// |<ESC>j` => this will place `// |` at the cloumn `39` and moves cursor to next line.
* press `q` to stop recording.
* Now press `@q` to insert `// |` in current line and move the cursor to next (automatically).
* Now keep on pressing `@@` to keep on inserting `// |` in subsequent lines. 
* final result will look like below

```rust
fn references_example_1() {
    println!("======== references_example_1 ========");
    let a = "yummy;                   // |  // |
    let mut b = &a;                   // |  // |
                                      // |  // |
    println!("a = {}, b = {}", a, b); // |  // |
    // a = "hotmail";                 // |  // |
    b = &"oracle";                    // |  // |
    println!("a = {}, b = {}", a, b); // |  // |

}

```

## More vertical lines using ex-`COMMAND`s on right side

* We want to create vertical lines of different texts
* `:execute "normal! $40a \<ESC>39|d$a//\<ESC>20a \<ESC>j"` => inserts `//` at 39 coloumn and inserts 20 blank spaces after `//` as well. Inserting extra blank space is required in order to insert text on right side.
* Now in `NORMAL` mode you can type @@ to repeat above command, and it will keep on inserting `//` in subsequent lines. However next ex-`COMMAND` will overwrite `@` register.
* Second option: is to take above ex-`COMMAND` in register and execute register in `NORMAL` mode. We shall take this ex-`COMMAND` in `r` register.
   * write above ex-`COMMAND` like this => `:execute "normal! $40a \<ESC>39|d$a//\<ESC>20a \<ESC>j"<ctl+v><CTRL-ENTER>`. `CTRL-v` and `CTRL-ENTER` are literal key press. This will look like below.
   * `:execute "normal! $40a \<ESC>39|d$a//\<ESC>20a \<ESC>j"^M`
   * In `NORMAL` mode, press `v` and select the above ex-`COMMAND` text (using l repeatedly) till `^` point only.
   * Now press `"ry` to copy this text in `r` register.
   * Now go to line where you want to insert `//` and press `@r` (in `NORMAL`) mode.
   * Now keep on pressing `@@` to keep on inserting `// |` in subsequent lines. 
* Third option: is to take above ex-`COMMAND` in register and execute register in `COMMAND` mode. We shall take this ex-`COMMAND` in `r` register.
   * write above ex-`COMMAND` like this => `:execute "normal! $40a \<ESC>39|d$a//\<ESC>20a \<ESC>j"`
   * In `NORMAL` mode, press `v` and select the above ex-`COMMAND` text (using l repeatedly) till " point only.
   * Now press `"ry` to copy this text in `r` register.
   * Now go to line where you want to insert `//` and go to `COMMAND` mode and press `:@r` and press `ENTER` key in the end.
   * Go to `COMMAND` mode and press `:@:` to repeat the last ex-`COMMAND`.
   * Keep on repeating `@@` in `NORMAL` mode to keep on inserting `// |` in subsequent lines.

* We will be storing following 4 command in their own respective registers.
* `:execute "normal! $40a \<ESC>39|d$a//\<ESC>20a \<ESC>j"^M`  => store it in register `r`. `^M` is actually `<CTRL-v><CTRL-ENTER>` key sequence 
* `:execute "normal! 43|\<ESC>R|a\<ESC>j"^M` => store it in register `a`. `^M` is actually `<CTRL-v><CTRL-ENTER>` key sequence
* `:execute "normal! 47|\<ESC>R|b\<ESC>j"^M` => store it in register `b`. `^M` is actually `<CTRL-v><CTRL-ENTER>` key sequence
* `:execute "normal! 51|\<ESC>R|c\<ESC>j"^M` => store it in register `c`. `^M` is actually `<CTRL-v><CTRL-ENTER>` key sequence

* We have following text

```rust
fn references_example_2() {
    println!("======== references_example_2 ========");
    let mut a = "yummy";
    let b = &mut a;
    println!("a = {}, b = {}", a, b);
    println!("b = {}", b);
    *b = "oracle";
    println!("b = {}", b);

    let c = &a;

    println!("a = {}", a);

    println!("b = {}", b);


    println!("c = {}", c);

}
```

Above text will be converted into

```rust
fn references_example_2() {
    println!("======== references_example_2 ========");
    let mut a = "yummy";
    let b = &mut a;                   //      |b
                                      //      |b
    println!("a = {}, b = {}", a, b); //  |a  |b
                                      //  |a  |b
    println!("b = {}", b);            //  |a  |b
    *b = "oracle";                    //  |a  |b
    println!("b = {}", b);            //  |a  |b
                                      //  |a  |b
    let c = &a;                       //  |a  |b  |c
                                      //  |a  |b  |c
    println!("a = {}", a);            //  |a  |b  |c
                                      //      |b  |c
    println!("b = {}", b);            //      |b  |c
                                      //          |c
                                      //          |c
    println!("c = {}", c);            //          |c

}
```

## Draw table in vim

* In INSERT mode type `|`
* go in  `NORMAL` mode and type `yy20p` => this draws the vertical line.
* Keep in `NORMAL` mode and type `gg` => brings you to the top of file
* press `CTRL-v` to go in BLOCK `VISUAL` mode
* press `G` (shift-g) to select the vertical line till the end.
* now copy this vertical line in register a by pressing `"ay` => this also moves the cursor to the top of file.
* Now create a macro in `q` register. In `NORMAL` mode, press `qqq` to first clear the `q` register. then press `qq` again to start recording.
* Now press `15a<SPACE><ESC>` to insert 15 spaces in the first line. Note: you have to literally press `<SPACE>` and `<ESC>` key sequences here
* Now press `"ap` => to paste the vertical line at 17th coloumn.
* Now press `q` to stop recording.
* Now press `@q` to insert vertical 33rd coloumn
* Now press `@@` to insert vertical 49th coloumn
* Now press `@@` to insert vertical 65th coloumn
* Now press `@@` to insert vertical 81st coloumn and so on...
result will be like this:

```
|               |               |               |               |               |
|               |               |               |               |               |
|               |               |               |               |               |
|               |               |               |               |               |
|               |               |               |               |               |
|               |               |               |               |               |
|               |               |               |               |               |
|               |               |               |               |               |
|               |               |               |               |               |
|               |               |               |               |               |
|               |               |               |               |               |
|               |               |               |               |               |
|               |               |               |               |               |
|               |               |               |               |               |
|               |               |               |               |               |
|               |               |               |               |               |
|               |               |               |               |               |
|               |               |               |               |               |
|               |               |               |               |               |
|               |               |               |               |               |
|               |               |               |               |               |
```
