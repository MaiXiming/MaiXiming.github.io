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
pip install pkgname -i https://pypi.tuna.tsinghua.edu.cn/simple # 设置查找包的源
pip list

pip freeze > requirements.txt # 导出依赖
pip install -r requirements.txt

deactivate

rm -rf .venv
```


### venv vs conda
问大家似乎说也没啥区别，所以都可以。主要看自己网络环境下，哪个有可用的源。