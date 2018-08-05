---
title: mac系统中vim-taglist的配置
date: 2018-08-06 00:20:37
tags: 
 -- mac
 -- vim
 -- taglist
---

`taglist`, `nerdtree`, 语法检查等是 `vim` 最常用的插件。今天在mac系统下配置的 `taglist.vim` 插件的时候遇到一点坑，特此记录备忘。
<!--more-->

### 问题描述

按照 [`taglist 官网`](http://vim-taglist.sourceforge.net/installation.html) 的描述下载和安装插件后，插件并没有生效。
`which ctags` 命令，确认了依赖的 `ctags` 已经安装。没有头绪。

google 了一下，原来 `mac osx` 系统自带的 `ctags` 并不是 `taglist` 插件依赖的那个。
问题确认了，那么安装依赖的软件就好了。

### 解决方案

1. 重装`ctags`

	这里可以去下载源码编译；更简单的方法当然是 `brew install ctags`;

	我这一步可以看到安装的路径：`/usr/local/Cellar/ctags/5.8_1/bin/ctags`

2. 调整命令
	
	更新`taglist.vim`插件的配置，采用新安装的ctags命令来运行插件;
	在`.vimrc`（或位于其他地方的配置文件）中添加：

	```
	let g:Tlist_Ctags_Cmd='$ctags_executable'

	# 比如，我的 `brew install ctags` 安装在 /usr/local/Cellar/ctags/5.8_1/bin/ctags

	let g:Tlist_Ctags_Cmd='/usr/local/Cellar/ctags/5.8_1/bin/ctags'
	```

	之后在 `vim` 中尝试下 `:TlistToggle`，发现函数目录就出来了。

	最后就是添加快捷键(这里配置为`F2`)：

	```
	 noremap <F2> :TlistToggle<CR>
	```

