---
title: "Zsh"
date: 2020-05-04T21:35:35+01:00
draft: true
---

## ZSH customization

### .zshrc
Install `oh-my-zsh` first.
```
export ZSH="/Users/gpal/.oh-my-zsh"
ZSH_THEME="bira"
plugins=(git aws direnv git-prompt jenv pyenv rbenv rust ssh-agent terraform tmux vagrant-prompt vi-mode virtualenv osx per-directory-history z zsh-autosuggestions zsh-synta
x-highlighting gcloud)
source $ZSH/oh-my-zsh.sh
bindkey -v
```

### iterm
* Go to `Preferences`->`Profiles=Create a new profile`
* Select newly created profile and go to `keys`->`HotKey Window`->`select checkbox`->`configure HotKey window`. Record HostKey as <option>-<space>
* set/check `colors`->`colors-preset=select Pastel` or set the color-preset that suits you.
Ref: https://www.freecodecamp.org/news/jazz-up-your-zsh-terminal-in-seven-steps-a-visual-guide-e81a8fd59a38/
* `Text->Font=Monaco->Size=14`
* `Window`->`Style=Maximized`->`space=All Spaces`

