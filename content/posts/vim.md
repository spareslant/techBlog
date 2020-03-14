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

## change
* in vim type `:h word-motions`
* `cit` => change text in-between tags
* `cf<char>` => search for a <char> in forward direction and deletes the whole text including char and place in insert mode. 
* `cF<char>` => same as above in backward direction.
* `ct<char>` => same as `cf<char>` but do not include the char.
* `cT<char>` => do not include char in backward.


