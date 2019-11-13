---
title: Backpropagation
date: 2019-11-13 19:46:55
tags:
	- backpropagation
categories:
	- Machine Learning
---

后向传播(backpropagation)算法在深度学习中扮演了非常重要的角色，它能够从损失函数开始链式地对网络层中的权重进行梯度计算与更新。可以先参考[维基百科: Backpropagation](https://en.wikipedia.org/wiki/Backpropagation)，文章很好地还原了深度神经网络(DNN)每一层网络从后往前的链式梯度计算关系。然后再结合下面这几张图理解每一层网络中每个单元的梯度具体计算与推导过程。


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

