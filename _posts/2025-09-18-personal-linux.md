---
layout: single
title: "Ubuntu/Linux命令总结"
date: 2025-09-18 12:00:00 +0800
categories: [personal]
comments: true
tags: [ubuntu, linux]
toc: true
---

## 系统命令
### 删除rm
| 命令 | 解释 | 备注 |
| --- | --- | --- |
| `rm file` | 删除文件 |
| `rm -f test.txt` | 强制，不提示 |
| `rmdir folder_empty/` | 只能删除空文件夹，里面有文件时会报错 |  |
| `rm -r folder/` | 递归删除目录及其内容 | 高危命令，仔细检查 |
| `rm -rf folder/` | 甚至不提示 | 最危险 | `rm -rf /` `rm -rf ~/` `rm -rf *`很恐怖 |
| `rm -rv folder/` | 显示删除过程中的文件 |  |
| `rm *.log` | 通配符，所有.log文件 |  |
| `rm tmp*` | tmp开头的所有文件 |  |
| `trash-put file.txt` | trash-cli工具包 | `trash-put trash-list trash-empty` |  

## 工具

### tmux
| 命令 | 解释 | 备注 |
| --- | --- | --- |
| `tmux ls` | 显示可选窗口 |  |
| `tmux new -s tmp` | 新建 |  |
| `tmux a -t tmp` | 打开已有窗口 |  |
| `tmux detach` | 关闭窗口，回到cmd | （ctrl + b, 松开，d） |
| `tmux kill-session -t tmp` | 删除窗口 |  |
| `tmux kill-server` | 删除所有窗口 |  |
