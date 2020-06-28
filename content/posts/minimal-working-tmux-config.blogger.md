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

# status line background colour will be the colour of terminal in background
set -g status-bg default
setw -g display-time 5000

####################################################################################################
#      settings for TMUX widnows status decorations                                                #
####################################################################################################
# status format for both last-active-window and all the windows before last-active
# commas are escaped using # char in below statement. i.e #, => escapes comma. Below 5 lines are actually one line broke up into 5
setw -g window-status-format "#{\
?window_last_flag,\
#[bold #, bg=colour226 #, fg=colour232] #I.#P #W ,\
#[bold #, bg=colour124 #, fg=colour232] #I.#P #W \
}"

# status format for current active window
setw -g window-status-current-format "#[bold #, bg=colour77 #, fg=colour232] #I.#P#F #W "
setw -g window-status-separator "   "

####################################################################################################
#      settings for TMUX pane status decorations                                                   #
####################################################################################################
setw -g pane-border-status top

# we do not need bg colour here as we want the bg colour to be the colour of the terminal, therefore bg is set to default. We may even omit bg here.
# set the colouring of border for current active pane
setw -g pane-active-border-style bg=default,fg=colour81

# we cannot have space after the comma here.
# set the colouring of border for non-current pane
setw -g pane-border-style bg=default,fg=colour180

# commas are escaped using # char in below statement. i.e #, => escapes , . Below 5 lines are actually one line broke up into 5
# two different colour schemes for current and all non-current panes TEXT
setw -g pane-border-format "#{\
?pane_active,\
#[bold #, bg=colour81 #, fg=colour16]  ~>> #I.#P   #T <<~  ,\
#[bg=colour180 #, fg=colour16] #I.#P #T \
}"

####################################################################################################
#      Other settings                                                                              #
####################################################################################################
set -g allow-rename off
set -g mode-keys vi
set-option -g mouse on
```

## Launch tmux
```bash
tmux -u
```
**Note:** `-u` is specified for terminals where locale is not set to `UTF-8`
**Note** To change the title of pane run this command `ctrl-b :select-pane -T MyTitle`