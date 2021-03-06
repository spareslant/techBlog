---
title: "Vim Vagrant Plugin using Vundle"
date: 2020-02-25T17:13:09Z
draft: false
tags: ["vim", "vagrantfile plugin", "Vundle"]
---

## Introduction
We will be installing *VIM* vagrant plugin using *Vundle*. This assumes that you have already configured your *VIM* to use *Vundle*

## A sample `.vimrc` file after installing `Vundle` is below.
`Vundle` installation can be done by following this link https://github.com/VundleVim/Vundle.vim .

```
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
```

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

```
set nocompatible              " be iMproved, required
filetype off                  " required

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
Plugin 'VundleVim/Vundle.vim'
Plugin 'file:///</full/path/to/home/dir>/.vim/vagrantfile'

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
```

## Activate `vagrantfile` plugin
Run following command to install the plugin and activate it.
```bash
vim +PluginInstall +qall
```



