---
title: "Minimal Working Tmux Config"
date: 2020-03-15T20:33:04Z
draft: true
tags: ["terminal", "tmux"]
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


