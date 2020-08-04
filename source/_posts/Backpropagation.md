---
title: Backpropagation
date: 2019-11-13 19:46:55
tags:
	- backpropagation
categories:
	- Machine Learning
---

后向传播(backpropagation)算法在深度学习中扮演了非常重要的角色，它能够从损失函数开始链式地对网络层中的权重进行梯度计算与更新。可以先参考[维基百科: Backpropagation](https://en.wikipedia.org/wiki/Backpropagation)，文章很好地还原了深度神经网络(DNN)每一层网络从后往前的链式梯度计算关系。然后再结合下面这几张图理解每一层网络中每个单元的梯度具体计算与推导过程，假设前向计算公式$wx + b$.


<div align="center">
    <img src="/images/backpropagation/bp1.png">
</div>

<div align="center">
    <img src="/images/backpropagation/bp2.png">
</div>

<div align="center">
    <img src="/images/backpropagation/bp3.png">
</div>

<div align="center">
    <img src="/images/backpropagation/bp4.png">
</div>

<div align="center">
    <img src="/images/backpropagation/bp5.png">
</div>
举个例子

<img src="/images/backpropagation/bp6.svg" align="center">

上图是一个两层的全连接神经网络，其中$net_{i}$是输入，$net_{k}$是输出，输出在softmax之后计算交叉熵损失。下面给出详细计算隐藏层$net_{j}$对应的权重$w_{ij}$梯度的过程，其中$N$是batch size，$y$是真实标签，$o$是softmax激活后的输出
$$
E(w)=\frac{1}{N}\sum_{i=1}^{N}E_{p}(w)=\frac{1}{N}\sum_{i=1}^{N}\sum_{d=1}^{D}{-y_{id}\log{o_{id}}}\\
\begin{aligned}
\frac{\partial{E_{p}(w)}}{\partial{w_{ij}}}&=\frac{\partial{E_{p}(w)}}{\partial{net_{j}}} \cdot \frac{\partial{net_{j}}}{\partial{w_{ij}}}\\
&=x_{i}\sum_{k \in nextlayer(j)}\frac{\partial{E_{p}(w)}}{\partial{net_k}} \cdot \frac{\partial{net_k}}{net_{j}}\\
&=x_{i}\sum_{k \in nextlayer(j)}\frac{\partial{E_{p}(w)}}{\partial{net_{k}}} \cdot w_{jk}
\end{aligned}
$$
又因为
$$
\begin{aligned}
\frac{\partial{E_{p}(w)}}{\partial{net_{k}}} &= \frac{\partial{E_{p}(w)}}{\partial{o_k}} \cdot \frac{\partial{o_k}}{\partial{net_k}} \\
&=\frac{1}{-o_k} \cdot o_k \cdot (1-o_k) \\
&=o_k - 1
\end{aligned}
$$
所以
$$
\frac{\partial{E_p(w)}}{\partial{w_{ij}}}=x_i\sum_{k \in nextlayer(j)}{((o_{k}-1) \cdot w_{jk})}
$$
除此之外，还有一个问题，交叉熵损失函数计算值只与标签1对应的输出相关，那么标签0对应输出的相关权重就无需计算梯度并进行更新了么？跑了一下上图两层全连接神经网络的demo程序，发现无论输出对应标签是0还是1，其相关权重梯度都不为0。通过对权重梯度的核算发现，对于标签0对应输出，虽然在交叉熵损失函数计算结果中没有得到体现，但是在后向传播梯度计算过程中，会采用$-(1-y_0)\log{(1-O_0)}$作为0标签相关权重梯度计算的损失，这点类似于逻辑回归损失函数，其中$y_0$为0，$O_0$为对应softmax输出。demo程序如下

```python
import torch
import torch.nn as nn

class mymodel(nn.Module):
    def __init__(self):
        super(mymodel, self).__init__()
        self.l1 = nn.Linear(10, 20, bias=False)
        nn.init.constant_(self.l1.weight, 0.1)
        self.l2 = nn.Linear(20, 2, bias=False)
        nn.init.constant_(self.l2.weight, 0.1)

    def forward(self, x):
        t = self.l1(x)
        print(t)
        y_out = self.l2(t)
        print(y_out)
        return y_out

x = torch.rand(1, 10)
y = torch.tensor([0])
m = mymodel()
optimizer = torch.optim.SGD(m.parameters(), lr=0.1)
loss_func = nn.CrossEntropyLoss()
print(x)
print(m.l1.weight)
print(m.l2.weight)
y_out = m(x)
loss = loss_func(y_out, y)
print(loss)
optimizer.zero_grad()
loss.backward()
optimizer.step()
for name, param in m.named_parameters():
    print(name, param.grad, param)

```

