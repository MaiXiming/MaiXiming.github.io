---
layout: single
title: "深度学习总结-训练框架"
date: 2025-09-25 15:00:00 +0800
categories: [ai]
tags: [deep_learning, pytorch, training framework]
toc: true
---

(框架如pytorch提供了很简洁的api，但是用着用着就会忘记内部到底如何实现的。以线性回归为例，总结内部实现的大致原理，方便以后回忆)

## 总结
- data.TensorDataset + data.DataLoader 内部是for + yield来实现的。使用方法是`for X, y in data_iter:`
- 模型定义nn.Sequential()本质上就是一个计算函数+模型参数组合而成的对象，里面包含参数w/b，同时也包含计算函数（通过net(X)，析构函数？）。
- 损失函数：就是定义一个函数
- 模型初始化：对`net[i].weight.data`进行操作，使用`.func_()`
- 优化算法：本质上就是一个函数，对传入参数params进行更新。使用torch.no_grad()的方式
- `with torch.no_grad()`与`tensor.data -= lr*tensor.grad`本质上一样，对tensor的数值操作而不追踪计算图。推荐no_grad()因为更清晰
- 
```
import numpy as np
import torch
import random
import d2l

## 生成数据集
def synthetic_data(w, b, num_examples):
    # y = Xw+b+noise
    X = torch.normal(0, 1, (num_examples, len(w)))
    y = torch.mamul(X, w) + b
    y += torch.normal(0, 0.01, y.shape)
    return X, y.reshape((-1, 1))

true_w = torch.tensor([2, -3.4])
true_b = 4.2
features, labels = synthetic_data(true_w, true_b, 1000)


## 读取数据集 每次抽取一个小批量样本
# 从零实现
def data_iter(batch_size, features, labels):
    num_examples = len(features)
    indices = list(range(num_examples))
    random.shuffle(indices)
    for i in range(0, num_examples, batch_size):
        batch_indices = torch.tensor(indices[i, min(batch_size + i, num_examples)])
        yield features[batch_indices], labels[batch_indices]


# 框架
from torch.utils import data

def load_array(data_arrays, batch_size, is_train=True):
    dataset = data.TensorDataset(*data_arrays) # 解析
    return data.DataLoader(dataset, batch_size, shuffle=is_train)
    # data.DataLoader返回也是iteration对象，内部也是yield应该

batch_size = 10
data_iter = load_array((features, labels), batch_size) 

# 用法：直接在for循环中调用,从零实现调用函数，框架则直接调用iteration对象 for X, y in data_iter(); 或者next(iter(data_iter)), iter迭代器，next从迭代器中获取第一项


## 模型定义
# 从零实现
def linreg(X, w, b):
    return torch.matmul(X, w) + b

# 框架
from torch import nn
net = nn.Sequential(nn.Linear(2, 1)) # 就一层，net[0].weight/bias


## 初始化模型参数
# 从零实现
w = torch.normal(0, 0.01, size=(2, 1), requires_grad=True)
b = torch.zeros(1, requires_grad=True)

# 框架
net[0].weight.data.normal_(0, 0.01)
net[0].bias.data.fill_(0)

## 损失函数定义
# 从零实现
def squared_loss(y_hat, y):
    y = y.reshape(y_hat.shape)
    return (y - y_hat) ** 2 / 2

# 框架
loss = nn.MSELoss()


## 优化算法
# 从零实现
def sgd(params, lr, batch_size):
    with torch.no_grad():
        for param in params:
            param -= lr * param.grad / batch_size
            param.grad.zero_() # 每个for清零一次，框架放在前面了，没在step()中，都可以

# 框架
trainer = torch.optim.SGD(net.parameters(), lr=0.03)
# 简洁：使用tensor.data来更新，所以不需要no_grad()了


## 训练
# 从零实现
lr = 0.03
num_epochs = 3
net = linreg
loss = squared_loss

for epoch in range(num_epochs):
    for X, y in data_iter(batch_size, features, labels):
        y_hat = net(X, w, b)
        l = loss(y_hat, y)
        l.sum().backward()
        sgd([w, b], lr, batch_size) # grad.zero_()了
    with torch.no_grad():
        train_loss = loss(net(features, w, b), labels)
        print(f"epoch {epoch+1}, loss {float(train_loss.mean()):f}")

# 框架
for epoch in range(num_epochs):
    for X, y in data_iter:
        l = loss(net(X), y)
        trainer.zero_grad()
        l.backward()
        trainer.step()
    l = loss(net(features), labels)
    print(f'epoch {epoch+1}, loss {l:f}') # tensor不用转换吗？


```