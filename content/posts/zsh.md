---
title: "Zsh"
date: 2020-05-04T21:35:35+01:00
draft: true
---

## ZSH customization

### .zshrc
```
export ZSH="/Users/gpal/.oh-my-zsh"
ZSH_THEME="bira"
plugins=(git aws direnv git-prompt jenv pyenv rbenv rust ssh-agent terraform tmux vagrant-prompt vi-mode virtualenv osx per-directory-history z zsh-autosuggestions zsh-synta
x-highlighting gcloud)
source $ZSH/oh-my-zsh.sh
bindkey -v
```

