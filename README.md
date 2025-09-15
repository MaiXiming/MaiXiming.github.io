# Jekyll 博客骨架（Minimal Mistakes，中文友好）

这是一个可直接上传到 `YOUR_GITHUB_USERNAME.github.io` 仓库的最小可用骨架：
- 主题：Minimal Mistakes（remote_theme）
- 栏目：AI论文（/categories/ai/）、BCI（/categories/bci/）
- 页面：关于我、联系我
- 示例文章：各 1 篇

## 使用方式
1. 在 GitHub 创建名为 `YOUR_GITHUB_USERNAME.github.io` 的公开仓库。
2. 将本仓库所有文件上传（或用 `git` 推送）。
3. 仓库 Settings → Pages → 选择 **Deploy from a branch**，分支选 `main`、目录选 `/ (root)`。
4. 等待 1-3 分钟，访问 `https://YOUR_GITHUB_USERNAME.github.io` 查看站点。
5. 修改 `_config.yml` 中的 `YOUR_GITHUB_USERNAME`、作者信息、站点标题等内容。

## 写新文章
在 `_posts` 新建 `YYYY-MM-DD-标题.md`，Front Matter 示例：
```yaml
---
layout: single
title: "你的标题"
date: 2025-09-15 20:00:00 +0800
categories: [ai]   # 或 [bci]
tags: [tag1, tag2]
toc: true
---
```
然后用 Markdown 写正文，提交即可发布。

## 分类导航
- `/categories/ai/` 显示 AI论文 分类文章
- `/categories/bci/` 显示 BCI 分类文章
- `/categories/` 显示全部分类索引

## 可选：启用评论（giscus）
见 `_config.yml` 注释位置，按教程填入 `repo_id`/`category_id` 等。

## 自定义域名（可选）
根目录创建 `CNAME` 文件（内容为你的域名，如 `blog.example.com`），并在域名 DNS 添加 CNAME 记录到 `YOUR_GITHUB_USERNAME.github.io`。
```
CNAME 内容示例：
blog.example.com
```
