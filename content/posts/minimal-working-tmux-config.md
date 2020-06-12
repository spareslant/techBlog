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
set -g history-limit 99999
# setw -g status-utf8 on
set -g status-bg colour16
setw -g display-time 5000
setw -g window-status-format "#I.#P #W"
setw -g window-status-current-format "#I.#P#F #W"
setw -g window-status-current-style fg=colour7,bold,bg=colour22
setw -g window-status-last-style fg=colour7,bold,bg=colour23
setw -g window-status-style fg=colour11,bold,bg=colour52
setw -g window-status-separator "<->"

# setw -g pane-border-style fg=red,bg=black
setw -g pane-border-status top
setw -g pane-border-format "#[bold]#I.#P#F"

set -g allow-rename off
set -g mode-keys vi
set-option -g mouse on
```

## Launch tmux
```bash
tmux -u
```
**Note:** `-u` is specified for terminals where locale is not set to `UTF-8`


