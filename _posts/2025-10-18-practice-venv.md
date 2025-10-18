---
layout: single
title: "venv & pip"
date: 2025-10-17 10:00:00 +0800
categories: [practice]
tags: [环境]
toc: true
---

### venv 简介
python自带的虚拟环境创建。

### venv常用命令
```
python3.8 -m venv .myenv [/path/to/env_name] # 默认在当前目录下创建. "."是约定俗成，区别于一般文件夹。
python3.8 -m venv --system-site-packages .venv # 继承系统的所有包

source .venv/bin/activate # 命令行会显示环境名称

pip install pkgname[==3.2.12] # pip ==, conda =
pip install requests pandas # 多个包
pip install pkgname -i https://pypi.tuna.tsinghua.edu.cn/simple # 设置查找包的源 -i等价于--index-url
pip list

pip freeze > requirements.txt # 导出依赖
pip install -r requirements.txt

deactivate

rm -rf .venv
```

### pip安装
1. wheel安装

python第三方库（包）会将二进制文件打包为`.whl`格式的文件，供pip安装。`pip install numpy`本质上就是去源找`numpy.whl`文件，进行安装。所以等价安装方式是下载`numpy.whl`，然后`pip install numpy.whl`，效果一样的。

2. 源码安装

python第三方库（包）的github仓库一般是一个工程项目，包含了源码，所以包的第二种方式就是源码安装。

`pip install -v -e ".[notebooks]"`：定位到工程目录下，会有`setup.py`，运行命令，就实现了从源码开始安装。
- -v表示详细输出，-e表示可编辑模式，即对本包的源码改动，立刻对安装的包的功能进行修改，方便调试。.代表当前目录下。[notebooks]代表setup.py中含有notebooks栏，里面包含一些其他的包，也一起下载安装掉。双引号表示是字符串，用反义符号表示一样的。

### venv vs conda
问大家似乎说也没啥区别，所以都可以。主要看自己网络环境下，哪个有可用的源。