---
title: 搭建hexo博客，Netlify持续集成
date: 2019-01-04 23:49:16
tags:
---

[TOC]

基于macOs搭建

\# 建立远程仓库

\1. 建立以个人github域名为名的软件仓库，例如`abc.github.com`，则建立`abc.github.io`的软件仓库。

\2. 在软件仓库中，setting设置个人域名为`abc.github.io`。

\# 本地准备

安装`git`，`homebrew`，`node.js`，`hexo`。

\1. 安装homebrew

    macos建议使用`homebrew`进行软件安装，对应命令为：

\```shell

/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" 

 \```

\2. 安装git

    使用git版本管理工具，与GitHub进行通信

\```shell

brew install git

\```

\## 安装hexo

\## 创建本地文件

\# Hello World