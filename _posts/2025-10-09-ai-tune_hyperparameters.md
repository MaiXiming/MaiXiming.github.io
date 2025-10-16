---
layout: single
title: "深度学习总结-基调超参实践"
date: 2025-09-25 15:00:00 +0800
categories: [ai]
tags: [deep_learning, pytorch, models]
toc: true
---


### 线性回归（略，直接可以得解析解）

### softmax回归
使用d2l中的Fashion-MNIST数据集：
- lr=0.1, epochs=10: loss曲线平滑，train loss 0.447, train acc 0.849, test acc 0.834
- lr=0.1, epochs=20: loss曲线平滑，train loss 0.421, train acc 0.857, test acc 0.837
- lr=0.1, epochs=50: loss曲线平滑，train loss 0.394, train acc 0.865, test acc 0.839

- lr=0.5, epochs=10: loss曲线初始极高，有下降趋势，但是过程抖动，上下上下，train loss 0.748, train acc 0.822, test acc 0.739
- lr=5, epochs=10: loss曲线nan（权重也是nan，说明溢出+inf-inf了），train loss nan, train acc 0.100, test acc 0.100

- lr=0.05, epochs=10: loss曲线平滑，train loss 0.478, train acc 0.841, test acc 0.828
- lr=0.01, epochs=10: loss曲线平滑，但是初始很高（因为lr太小了，1epoch后loss依然很高）train loss 0.606, train acc 0.807, test acc 0.797

结论：
- lr代表下降的速度，过大会震荡，导致碗状曲线冲出、loss增大的情况；lr太大太大甚至会产生inf/nan的问题；lr过小虽平滑，但是慢，三步等于原来一步，相同时间后loss仍然较高。
- epoch代表搜索的时间范围，epoch越大，给的时间越多，在lr合适（曲线平滑）的情况下，train_loss下降越多，但是test_acc效果不能保证（因为可能某个epoch后已经饱和，收益不大了）

### MLP
使用d2l中的Fashion-MNIST数据集 concise：
- lr=0.1, epoch=10, hidden=256: loss曲线平滑，但是还没收敛，train loss 0.381, train acc 0.866, test acc 0.854
- lr=1/0.8, epoch=10, hidden=256: loss曲线陡峭，效果也不好。说明lr太大了。
- lr=0.5, epoch=10,hidden=256: loss曲线平滑，但是还没收敛，train loss 0.304, train acc 0.886, test acc 0.856
- lr=0.5, epoch=100,hidden=256: loss曲线平滑，基本收敛了（train_acc快100%），但是出现loss_spike（1/2次），平稳时train loss 0.091, train acc 0.967, test acc 0.888。

- lr=0.5, epoch=100,hidden=16: loss曲线陡峭，train loss 0.710, train acc 0.737, test acc 0.658
- lr=0.3, epoch=100,hidden=16: loss曲线平滑，但没收敛，train loss 0.266, train acc 0.901, test acc 0.862
- lr=0.3, epoch=500,hidden=16: loss曲线平滑，收敛了，但是train_acc没到1。train loss 0.210, train acc 0.922, test acc 0.831

- lr=0.1, epoch=10, hidden=512: loss曲线平滑，但是还没收敛，train loss 0.379, train acc 0.866, test acc 0.848（loss低了，但是test acc也低了）
- lr=0.1, epoch=10, hidden=784: loss曲线平滑，但是还没收敛，train loss 0.376, train acc 0.866, test acc 0.841（loss更低了，但是test acc也更低了。是overfitting，但是怎么能保证不是lr/epoch没调好？）

- lr=0.1, epoch=50, hidden=784: loss曲线平滑，但是还没收敛，train loss 0.224, train acc 0.920, test acc 0.888（说明模型还没有overfitting吧，因为loss低了，test acc变高了。所以改了网络参数，不能认为test acc下降了就是overfitting了。每个模型的lr/epoch都不一样，要调到比较优的才可以？）
- lr=0.1, epoch=100, hidden=784: loss曲线平滑，但是还是没收敛，train loss 0.142, train acc 0.951, test acc 0.891
- lr=0.1, epoch=150, hidden=784: loss曲线平滑，但是还是没收敛，train loss 0.091, train acc 0.971, test acc 0.898
- lr=0.1, epoch=1000, hidden=784: loss曲线平滑，且在epoch=400左右收敛了，train loss 0.001, train acc 1.000, test acc 0.898（这是真的overfitting了。用时33min）

经验总结：
- lr：lr太高会导致损失nan或者震荡；lr太低收敛极慢，效率低。所以要找到一个loss平滑但是lr最大的点。
- epoch：epoch影响模型的学习范围，所以大/小会导致模型本身的过拟合/欠拟合。小epoch下观察不同模型的效果是不对的，因为loss曲线可能都没有收敛，模型还在学习的过程中，不代表最终状态。要关注训练动态（loss曲线），而不是一个数值。所以最佳做法是max epoch + early stopping，来评估当前模型的最终效果。（小epoch，改变模型容量，给出结论是不可靠的。比如epoch增多可能valid_acc更好了，这时候其实还没到模型容量变大的拐点。因为epoch小就不知道当前模型的最优搜索结果，那么当前网络结构的实际效果就不知道。比如epoch=10和epoch=100可以相差5%。模型容量造成的overfitting/underfitting应该建立在loss收敛后，再给出结论。）
- 收敛定义：train_loss/valid_loss在连续5-10个epoch内变化<0.01,或者valid_acc不再改善（饱和/出现极值）。所以max+early stopping才能找到当前模型的最终效果。
- 模型：不同模型（比如hidden不同、架构不同），lr都不一样，一般大模型用小lr。所以模型一旦改变，lr也应该改变。
- batch size: 小了代表没有看全体，有噪声，loss曲线不平滑；大看了全体，但是读取多，所以慢。一般来说，引入噪声没什么不好，所以根据GPU显存能力确定最大的bs即可。注意bs和lr是联动的，不同bs下的lr设置都不一样，所以先确定bs，再确定lr。
- 趋势 vs 快照：一组超参数==一个train_acc/valid_acc，这很好。但是这组超参数组合背后是一个训练过程，训练过程是否平滑决定了训练是否成功，只有平滑了，acc才可信。所以不要只看结果快照，每个结果都要有loss曲线。看趋势，不要只看快照。
- 试超参数：我们的目标是找到最佳，所以一定要找到极值的现象。比如lr=1震荡，lr=0.5平滑，lr=0.1平滑且慢，那么lr选0.5.不能只试了1/0.1后就选0.1.hidden_num同理。不要只探索出单调性，要探索出极值性。
- SGD momentum/Adam等优化器：loss曲线平滑很重要，结论才可信。这些优化器的提出就是能够找到更平滑曲线，从而找出更好的局部最优解，不用担心比如后期是否lr太大等优化过程的问题。换句话说，让我们专注于超参数搜索，而不是纠结于优化过程的bug。也就是说，他们的提出就是让我们能够不操心“结果跑出来了，但是会不会loss曲线不好，优化过程有问题，导致结论不能用”的担忧。比如Adam只要调lr，loss平滑了就不用管了，优化过程肯定顺利，结果就是可信。所以Adam/SGD在不同任务中都一样用。


结论：
1. epoch大小会导致模型本身因学习过程长短的过拟合/欠拟合。如果要评估一个模型下结论，一定是max epoch + early stopping，短epoch下的结论不可信，尤其是关于“模型容量”。（max epoch+early stopping下，不同模型可能能用同一个lr，只要取min(理论最优)，无非慢一点，但是都能stop到收敛）。early stopping的patience步长也比较关键，过小容易停于次优点。
2. loss lanscape损失地貌（想象一座山），模型改变、数据集改变都会改变，那么超参数组合会变，优化过程也就会改变，train_acc/valid_acc也都会改变。loss-landscapes库（pytorch可视化工具）。
3. bs/lr/epoch: 
    - 根据显存容量调最大bs；bs-lr scaling，大bs用大lr。
    - 设置小epoch 10/15，并且部分数据（训练快），找到平滑且max lr；上全数据（lr可能会变小），小epoch，测试选择的lr，若震荡则变小。
    - 大epoch + early stopping，要观察到valid_loss 饱和/出现极值，那么当前超参数组合下的结果就出来了。
4. 在模型足够复杂时（hidden=784/256，但是16模型则太小，不满足），lr合适不过大，那么随着epoch增加，train_loss/train_acc接近于0/1，完美。test_acc/valid_acc会先上升，然后下降/饱和（本例），或者double descent (loss先降后升，经典bias-variance tradeoff U型，但是接下来又再下降。似乎也常见，并不是有bug)



