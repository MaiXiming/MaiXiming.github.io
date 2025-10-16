---
layout: single
title: "conda"
date: 2025-10-17 10:00:00 +0800
categories: [practice]
tags: [环境]
toc: true
---

### conda简介
配环境用的工具，为每个项目配一个环境，因为每个项目的包的版本可能不一样，防止冲突。

### conda channels

conda安装包需要去一个网站url上搜索。

conda默认：
- default channels: [repo.anaconda.com/...] 官方提供的库，但是有墙，而且开源的包很少。
- conda-forge: [conda.anaconda.com/....] 官方认证的库，包非常的全，但是依然有墙的问题

因为访问速度的原因，我们希望更换url/库，来conda install，所以就有清华源/阿里源等等的url。他们的内容和官方库基本一样的，作用就是代替官方的repo./conda./，让我们在conda install时搜索国内的库，加速下载。

具体原理 / `~/.condarc`文件：

`~/.condarc`文件包含了conda channels的所有设置。需要对channels进行设置以后才会出现，否则是没有这一文件的。

```
channels: # 搜索时的优先级
- default
- conda-forge

channel_alias: xxx.com/ # 很少用，代表-c 别名时添加的前缀

default channels:
- tuna.tsinghua.xxx.com/xxxxxx # 直接替换为清华源，那么默认就去清华源搜索了
# 默认是 - repo.anaconda.com/...

custom channels:
- conda-forge: tuna.tsinghua.com/xxxx # 默认 conda.anaconda.com/xxx 这里也可以替换为清华源
- pytorch: tuna.tsinghua.com/xxxx # 默认 conda.anaconda.com/xxx 这里也可以替换为清华源
```

逻辑：
- 当conda install时，会按照channels栏目的优先级去对应的库中寻找。
- 比如default，那么就会去default channels中给出的库寻找。官方设置是去repo.anaconda.com/xxx寻找，但是我们希望加速，所以直接替换成tsinghua，那么default和官方库就完全没关系了。
- default没有，那么就会去conda-forge中寻找。同理，我们替换为tsinghua，加速了，已经不会去conda.anaconda.com/xxx寻找了。
- custom channels代表的是别名的键值对。即`conda install -c [别名] package_name`时可以使用别名而不用使用url（也不可以使用url，必须是别名）。一般conda-forge/pytorch/binoxxx的网址都差不多，因为每个库都包含了这些东西。

### conda 常见命令
