---
layout: single
title: "深度学习总结-预备知识"
date: 2025-09-18 15:00:00 +0800
categories: [ai]
tags: [deep_learning, summary]
toc: true
---

## 数组操作 ndarray
张量 = 数组，带多个维度。每个维度对应一个轴。一个轴=向量；两个轴=矩阵。

pytorch使用tensor定义张量，和numpy的ndarray很像，但是支持GPU计算，并且支持自动微分。

### 创建张量

`x = torch.arange(12)`: 创建tensor([0-11])的tensor.默认是整数。

`x = torch.arange(12, dtype=torch.float32)`

`x = torch.zeros((2,3,4))`: 全0

`x = torch.ones((2,3,4))`: 全1

`x = torch.randn((3, 4))`: 正态分布随机采样

`x = torch.tensor([[2,1,4,3], [1,2,3,4,],[4,3,2,1]])`: 通过python list创建



### 张量属性

`x.shape`: 返回形状`torch.Size([12])`

`x.numel()`: 返回张量元素总数`12`

`type(x)`: 返回torch.Tensor或者numpy.ndarray

### 元素索引

索引 = 获取张量中的元素。

切片 = 通过连续范围的索引，返回张量的某些元素。是索引的一种。注意，并不会拷贝内存，而是返回视图view，即改变`y=x[:2]`，x也会跟着改变。

`X[-1]; X[1:3]; X[1,2]=9`: 索引/修改元素

`X[0:2, :] = 12`: 索引的元素都赋值为相同元素值



### 张量操作

`X = x.reshape(3, 4)`: 改变张量形状
`Z = torch.cat((X, Y), dim=0)`: 连结。dim是要改变的轴/维度，-1代表最后一个维度。

### 运算

`x + y ; x - y ; x * y ; x / y ; x ** y; torch.exp(x)`: element-wise

`X == Y; X > Y; X < Y`: element-wise，每个位置返回True / False，最终是一个tensor，元素都是True/False

`X.sum()`: 元素求和，返回1*1张量`tensor(66.)`

### 广播机制
形状不相同的两个张量，通过复制元素来拓宽张量，拓展后二者有相同的形状。然后按照元素进行运算。

一般来说，需要不一致的维度，有一个张量该维度长度是1.

```
a = torch.arange(3).reshape((3, 1))
b = torch.arange(3).reshape((1, 2))
a + b # torch.Size([3, 2])
```

### 张量内存分配
`Y = X + Y` 会为Y分配一个新的内存，可以用`id(Y)`获取元素内存地址。

这样不好，因为内存很贵。我们希望原地执行更新。

`Y[:] = X + Y; Y += X`: 原地更新Y，不指定新的内存地址

### 对象转换
torch2numpy: `A = X.numpy()`

numpy2torch: `B = torch.tensor(A)`

to python标量：`a.item(); float(a); int(a)`

## pandas: 待学习




## help torch

`print(dir(torch.distributions))`: 查看某个模块的所有属性（类和函数）
- __双下划线开头/结尾的可以忽略，python的特殊对象。
- _单下划线开头的函数是内部函数。

`help(torch.ones)`: 查看函数/类具体的说明

（Jupyter中）`list?`显示帮助文档；`list??`显示实现函数的python代码

