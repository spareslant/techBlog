---
title: "Minimal Working Tmux Config"
date: 2020-03-15T20:33:04Z
draft: False
tags: ["terminal", "tmux"]
---

## Minimal working tmux config 
Populate `$HOME/.tmux.conf` with following contents
```
set -g default-terminal "screen-256color"
set -g history-limit 5000
#setw -g status-utf8 on
set -g status-bg colour15
setw -g display-time 5000
setw -g window-status-format "<#I.#P #W>"
setw -g window-status-current-format "|#I.#P#F #W|"
setw -g window-status-last-style fg=red
setw -g window-status-current-style fg=brightcyan,bg=black
setw -g pane-border-style fg=red,bg=black

set -g allow-rename off
set -g mode-keys vi
```

## Launch tmux
```bash
tmux -u
```
**Note:** `-u` is specified for terminals where locale is not set to `UTF-8`


