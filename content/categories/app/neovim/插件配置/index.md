+++
title= "插件配置"
description= "neovim的一些插件配置"
date= 2022-09-11T02:22:44+08:00
author= "chao"
draft= false
image= "" 
math= true
categories= [
    "app"
]

tags=  [
    "vim "
]

+++

# neovim插件配置



## 前提: proxy网络代理

### 1.WSL1

~~~
nano ~/.bashrc

###############PROXY####################
export WIN11_IP=127.0.0.1
export all_proxy="socks5://${WIN11_IP}:7890"
~~~



### 2.wsl2

~~~
nano ~/.bashrc

###############PROXY####################
export WIN11_IP=$(cat /etc/resolv.conf |grep "nameserver" |cut -f 2 -d " ")
export all_proxy="socks5://${WIN11_IP}:7890"
~~~

### 3. VM

~~~
nano ~/.bashrc

###############PROXY####################
export WIN11_IP=windowsIP
export all_proxy="socks5://${WIN11_IP}:7890"
~~~

**测试**

~~~
curl www.google.com
~~~

## 插件



### 1.packer.nvim



~~~
##下载插件
git clone --depth 1 https://github.com/wbthomason/packer.nvim\
 ~/.local/share/nvim/site/pack/packer/start/require("plugins").nvim
~~~
修改~/.config/nvim/init.lua 
~~~
nvim ~/.config/nvim/init.lua
添加
require("plugins")
~~~

修改~/.config/nvim/lua/plugins.lua

~~~
nvim ~/.config/nvim/lua/plugins.lua
~~~
添加如下内容
~~~
-- This file can be loaded by calling `lua require('plugins')` from your init.vim

-- Only required if you have packer configured as `opt`
vim.cmd [[packadd packer.nvim]]

return require('packer').startup(function(use)
  -- packer.nvim插件包管理
  use 'wbthomason/packer.nvim'

end)
~~~

nvim打开任意文件，在command模式下输入(更新插件)

~~~
:PackerSync
~~~

### 2.air-line

~~~
  use {
        "vim-airline/vim-airline",
        requires = {
          "vim-airline/vim-airline-themes",
          --综合图标支持such vim-airline lightline, vim-startify
          "ryanoasis/vim-devicons"
        }
  }
~~~





### 3.主题

#### 3.1 gruvbox

#### [gruvbox地址](https://github.com/ellisonleao/gruvbox.nvim)

~~~
    -- gruvbox theme
    use {
        "ellisonleao/gruvbox.nvim",
        requires = {"rktjmp/lush.nvim"}
    }
~~~

### 4.nvim-tree

~~~
   use {
        'kyazdani42/nvim-tree.lua',
        requires = 'kyazdani42/nvim-web-devicons'
    }
~~~

### 5.bufferline

~~~
-- bufferline
use {'akinsho/bufferline.nvim', requires = 'kyazdani42/nvim-web-devicons'}
~~~

