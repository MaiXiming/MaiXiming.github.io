---
layout: single
title: "深度学习总结-基础模型代码"
date: 2025-09-25 15:00:00 +0800
categories: [ai]
tags: [deep_learning, pytorch, models]
toc: true
---

### 线性回归
```

```

### softmax回归
```
net = nn.Sequential(nn.Flatten(), nn.Linear(784, 10))
def init_weights(m):
    if type(m) == nn.Linear:
        nn.init.normal_(m.weight, std=0.01)
net.apply(init_weights)

loss = nn.CrossEntropyLoss(reduction='none') 

trainer = torch.optim.SGD(net.parameters(), lr=0.1)
```

softmax回归 = 单层神经网络+softmax归一化。但是模型本身没有softmax层，而是将未规范化的输出直接传递给了CE loss，这一点要注意。所以要看模型预测概率，需要手动加上softmax层。

### MLP
```
net = nn.Sequential(
    nn.Flatten(),
    nn.Linear(784, 256),
    nn.ReLU(),
    nn.Linear(256, 10),
    )
def init_weights(m):
    if type(m) == nn.Linear:
        nn.init.normal_(m.weight, std=0.01)
net.apply(init_weights)

```