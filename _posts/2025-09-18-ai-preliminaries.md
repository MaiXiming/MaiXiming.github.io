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

`len(x)`: 张量x的维度/轴的数量。（理解：因为x是一个大列表，所以len返回列表的长度）

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

## 线性代数

### 基本定义

- 0阶：标量scalar：创建`x = torch.tensor(3.0)`
- 1阶：向量vector：由标量作为元素组成的列表。
    - 比如一个样本就可以用一个向量表示，元素代表各个特征。
    - 创建：`x = torch.arange(4)`
    - 向量的默认方向是列向量。

- 2阶：矩阵：m行n列。
    - 创建：`x = torch.arange(12).reshape(3,4); x = torch.ones((3,4))`
    - 转置transpose：`A.T`

- n阶：张量。

- 维度：向量/张量某个轴的维度 = 向量/轴的长度；张量的维度 = 张量具有的轴的数量；
    - 张量（3,4,5）的维度是3；张量第二个轴的维度是4.

### 理解张量组成

数学上，向量是列向量，矩阵是二维向量有行和列，理解为由多个列向量组成的列表。理解X[0,0]很好，但是理解矩阵X[0]不太直观

python中，虽然向量在数学上是列向量，但是表示依然为一个行列表`tensor([1,2,3])`。而矩阵、张量则是由一维一维的行向量堆叠而来。比如三维张量`tensor([[[1,2,3], [2,3,4]], [[3,4,5], [4,5,6]], [[5,6,7], [6,7,8]]])`,shape=(3,2,3)。也就是说，最里面的维度/张量最右侧的维度是一个个列向量，由此组成外面一层维度，...，组成最外侧/最左侧的维度。因此，`x[0][0][0]`从外到内不断索引，和`x[0,0,0]`本质相同；

所以在python中，我们就将张量理解为大列表/大行向量，由一个个行向量/横向列表堆叠而成的数据。大列表=最外侧的列表，有许多元素；每个元素，都是一个次大列表，有许多元素；...；到倒数第二层，依然是一个列表，每个元素都是一个列表/行向量；最后一层，列表中的元素就是标量元素了。

我们通常将最外侧的轴，即x[k]，作为一个个样本。x[k]的内容，则是样本里的特征。比如矩阵(n, k), 大列表的每个元素x[k]/每一行x[k,:]都是一个样本，包括了样本的许多特征;比如图像(batch, x, y)，batch最外层代表样本，内侧x/y则是像素维度。而单独获取内侧的维度x[:,k]，则是获取所有样本的某一个维度的值的集合。

### 张量运算

- Hadamard积: = 元素乘积。

- 降维：降低张量的维度
    - 标量：`x.sum(); A.sum()`
    - 消失某个维度：`A.sum(axis=0); A.mean(axis=0)`.axis=0，指定沿着某个轴来求和，从而让该轴消失。
    - 消失多个维度：`A.sum(axis=[0,1])`
    - **（高维计算加速妙招）保持维度**，以能通过广播机制对矩阵所有元素进行处理：`sum_A = A.sum(axis=1, keepdims=True); A/sum_A`

- 点积 dot product: `torch.dot(x,y)` or `torch.sum(x*y)`向量元素乘积之和，返回标量。
    - 加权平均，夹角余弦，都属于点积。

- 矩阵A-向量x积 matrix-vector product: `torch.mv(A, x)`,返回向量。
    - 将矩阵每行视作向量，与x做内积。
    - y = Ax, 可以视为矩阵A来表示旋转。

- 矩阵A-矩阵B乘法：`torch.mm(A, B)`
    - 矩阵A每行为向量；矩阵b每列为向量；对b的每一列，进行Ax的mv，然后列`cat(, dim=1)`

### 范数
范数表示一个向量有多大，即将向量转为标量，从而可以将不同向量之间进行比较。

深度学习的优化问题，目标函数一般就用范数来表示。即将张量数据样本，表达为一个标量，从而可以比较大小，然后希望目标尽可能小。

- L2范数：`torch.norm(x)`元素平方和的平方根，深度学习最常见，一般``||x||``默认等同于``||x||2``.

- L1范数：`torch.abs(x).sum()`元素绝对值的和。

- Lp范数：元素绝对值的p次方的和，然后再p次方根。

- 矩阵Frobenius范数：`torch.norm(A)`，矩阵->向量->L2范数，表达矩阵的大小。
    - 官方更推荐`torch.linalg.norm(A)`，`torch.norm`被弃用了。
    - 指定p：`torch.linalg.norm(A, ord=k)`
    - 每个batch计算范数：`torch.linalg.norm(A, dim=(1,2))`。

## 微积分

[不太记得可回看详情，下面只是关键点的总结，不如看一遍清晰](https://zh.d2l.ai/chapter_preliminaries/calculus.html)
- 微分对深度学习来说很重要
- 导数是一元函数`y=f(x)`，x的瞬时变化率
- 偏导数是多元函数`f(x1, x2, ..., xn)`，f对xi的瞬时变化率`D_xi = partial(f) / partial(xi)`
- 梯度 = 偏导数组成的向量 = `[D_x1, D_x2, ..., D_xn]`
- 链式法则用于处理复合函数composite，因为深度学习中一层一层堆叠就是一层复合一层
- 深度学习拟合的函数就是`[y1=f1(x1,x2,...,xn), y2=f2(x1,x2,...,xn), ..., ym=fm(x1,x2,...,xn)]`
![多元函数的链式法则](/assets/img/ai/gradient_composite.png)

疑问：
- 梯度小节中，定义的函数f(x)是标量，但是规则中的f(x)是矢量。当然矢量函数的求导是矩阵没问题，但是这还符合梯度的定义吗？还是说矢量函数的求导又有一个新名字（以前学过，忘了）
![矢量求导](/assets/img/ai/gradient_vector_gradient.png)

## 自动微分 autograd
- 深度学习框架可以自动计算导数，通过构建计算图 computational graph的方式，来追踪计算，从而可以实现反向传播梯度（具体细节没细究）
- 当y是张量而非标量时，梯度就是高阶张量。但是不需要细究，因为深度学习的损失函数是标量，意味着y肯定是标量。
- 自动微分的好处是，即使是用控制流 (if/while/for/任意函数),依然可以追踪变量。
- 如果连续执行两次y.backward()，不会报错，而是将梯度信息积累到x.grad，所以两次的x.grad值不一样
- **y.backward()要求y必须是标量**，求出的是x的梯度x.grad。但是这**可以巧妙地用来一下求出一个函数的导数函数**，例如练习5.`y = sin(x); y.sum().backward(); x.grad` x.grad就是不同x处的导数值，因为可以将向量x看作x1-xn，sum()后求梯度就是求x1-xn各自的偏导数，对应的就是不同x取值处的导数了。

```
## 案例
x = torch.arange(4.0)
x.requires_grad_(True) # == x = torch.arange(4.0, requires_grad_=True)
x.grad # default None
y = 2 * torch.dot(x, x)
y.backward()
x.grad # tensor([0, 4, 8, 12])

## 新函数的梯度计算
x.grad.zero_() # 默认积累梯度，所以计算新的梯度前需要清除之前的值
y = x.sum()
y.backward()
x.grad

## 分离计算
x.grad.zero_()
y = x * x
u = y.detach() # 作为常数处理，梯度不会向后经u传到x
z = u * x
z.sum().backward()
x.grad == u # True

```

## 概率 probability
[不太记得可回看详情，下面只是关键点的总结，不如看一遍清晰](https://zh.d2l.ai/chapter_preliminaries/probability.html)（连续随机变量可见[链接](https://d2l.ai/chapter_appendix-mathematics-for-deep-learning/random-variables.html)）
- 机器学习就是根据概率做出预测
- 概率本身是一个很直观的概念，并不难体会，因为概率度量了**(不)确定性水平**
- 但是因为世界的分布很复杂，过程也很复杂，所以将概率公理化数学化，然后就可以用公理和推理来计算复杂的过程（比如HIV检测这种反直觉的事情）
- 分布理解为对事件的概率分配。
- `matrix.cumsum(dim=0)`可以用来计算数据集矩阵随着时间/行的推移的总和；`matrix/cumsum(dim=0)/matrix.cumsum(dim=0).sum(dim=1, keppdims=True)`来计算随着时间推移，不同列的特征占全体的比例/概率。
- 离散随机变量很好理解；连续随机变量从密度来理解，任何一点的重量/概率都是0，但是其密度反映了一个小区间下的重量/概率。
- 独立性更多理解为一个可以利用的条件，比如“因为A和B独立，所以`P(A|B)=P(A)`, `P(AB)=P(A)*P(B)`”.**若A和B独立，`P(A|B, C) = P(A|C)`，B可以直接去掉**



## help torch

`print(dir(torch.distributions))`: 查看某个模块的所有属性（类和函数）
- __双下划线开头/结尾的可以忽略，python的特殊对象。
- _单下划线开头的函数是内部函数。

`help(torch.ones)`: 查看函数/类具体的说明

（Jupyter中）`list?`显示帮助文档；`list??`显示实现函数的python代码

## d2l好用的工具

plot
- x为向量，y可以是列表，列表元素为f1(x), f2(x), etc.
- 完整：`plot(X, Y=None, xlabel=None, ylabel=None, legend=None, xlim=None, ylim=None, xscale='linear', yscale='linear', fmts=('-', 'm--', 'g-.', 'r:'), figsize=(3.5, 2.5), axes=None)`