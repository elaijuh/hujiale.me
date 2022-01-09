---

date: 2017-10-13T17:20:40+08:00
draft: false
title: "Install Youcompleteme with pathogen in neovim"
slug: ""
categories: ["tech"]
tags: ["vim", "neovim"]

---

Notes for installing YCM with pathogen, Mac environment.

Prerequisites:

- brew
- cmake
- python3/python2 (here i use python3)


```sh

# 1. Install neovim
brew install neovim

# 2. Install neovim python provider
# https://neovim.io/doc/user/provider.html#provider-python
pip3 install --upgrade neovim

# 3. Install YouCompleteMe
cd ~/.vim/bundle
git clone https://github.com/Valloric/YouCompleteMe.git && cd $_
git submodule update --init --recursive
# here i use system clang rather than downloading, and i only need c family, golang, javascript, rust completer
python3 install.py  --system-libclang --clang-completer --go-completer --js-completer --rust-completer

```






