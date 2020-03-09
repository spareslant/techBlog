---
title: "Vim Vagrant Plugin using Vundle"
date: 2020-02-25T17:13:09Z
draft: false
tags: ["vim", "vagrantfile plugin", "Vundle"]
---
<!--- Below style are also defined in static/css/my.css file.
They are repeatedly defined here so that pandoc can generate
the final HTML with all necessary css styles.
--->
<style>
.hl {color: #f155f1;}
.hlb {color: #f155f1; font-weight: bold;}
.hlbr {color:#e90001; font-weight: bold;}
/* <code> tag does not work in blogger. Use following class with span tag */
.code {color:#f20101; background: #f0f0f0; padding: 0.2em;    
</style>

## Introduction
We will be installing *VIM* vagrant plugin using *Vundle*. This assumes that you have already configured your *VIM* to use *Vundle*

## A sample `.vimrc` file after installing `Vundle` is below.
`Vundle` installation can be done by following this link https://github.com/VundleVim/Vundle.vim .

<pre>
set nocompatible              " be iMproved, required
filetype off                  " required

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
Plugin 'VundleVim/Vundle.vim'

" All of your Plugins must be added before the following line
call vundle#end()            " required

set tabstop=4
set expandtab
set softtabstop=4
set shiftwidth=4
filetype indent on
syntax on
" color desert
" color lettuce
</pre>
## Preparation of `Vagrantfile` plugin for `Vundle`
Create following structure of directories and files.
```bash
mkdir -p $HOME/.vim/vagrantfile/plugin
touch $HOME/.vim/vagrantfile/plugin/vagrantfile.vim
curl https://raw.githubusercontent.com/hashicorp/vagrant/master/contrib/vim/vagrantfile.vim -o $HOME/.vim/vagrantfile/plugin/vagrantfile.vim
cd $HOME/.vim/vagrantfile 
git init .
``` 
Note: It is important to have above structure and initialized it as a git repo.

## Make changes to `$HOME/.vimrc` file
Add following line to `$HOME/.vimrc` file.
```bash
Plugin 'file:///</full/path/to/home/dir>/.vim/vagrantfile'
```
`.vimrc` file will look like below:

<pre>
set nocompatible              " be iMproved, required
filetype off                  " required

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
Plugin 'VundleVim/Vundle.vim'
<span class="hlb">Plugin 'file:///</full/path/to/home/dir>/.vim/vagrantfile'</span>

" All of your Plugins must be added before the following line
call vundle#end()            " required

set tabstop=4
set expandtab
set softtabstop=4
set shiftwidth=4
filetype indent on
syntax on
" color desert
" color lettuce
</pre>

## Activate `vagrantfile` plugin
Run following command to install the plugin and activate it.
```bash
vim +PluginInstall +qall
```



