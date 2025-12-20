---
layout: single
title: "torch 待填坑"
date: 2025-12-20 10:00:00 +0800
categories: [practice]
tags: [工具]
toc: true
---

### torch简介
虽然说现在的工作内容都是改改代码之类的，但是有时候要测试的时候还问AI助手也是很浪费时间，还是要总结一下基础的用法

#### 基础使用
`torch.zeros/ones((shape), device='cuda:0', dtype=torch.float32)`

`tensor.shape`: 返回torch.shape，但是可以.shape[0] .shape[1]来获取维度int

`torch.where(mask, tensor1, tensor2)`: 根据mask来选择t1/t2，高效if-else

`tensor[:] = `原地更新：