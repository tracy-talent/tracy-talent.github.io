---
title: 各类概率分布函数tutorial
date: 2019-08-08 19:00:03
tags:
    - Math
categories:
    - Math
---

## 泊松分布、指数分布与伽马分布简介及其关系

泊松分布: $$Poisson(N(t)=n)=\frac{(\lambda t)^{n}e^{- \lambda t}}{n!},\ (\lambda t > 0, n \ge 0)$$

指数分布: $$f(x)=\lambda e^{-\lambda t},\ (\lambda > 0,t > 0)$$

伽马分布: 先看伽马函数，伽马函数又称第二类欧拉积分，函数式为$$\Gamma(x)=\int_{0}^{\infty}{t^{x-1}e^{-t}dt}(x > 0)$$，对伽马函数做一个变形即可得到
$$
\int_{0}^{\infty}{\frac{x^{\alpha-1}e^{-x}}{\Gamma(\alpha)}}dx=1 \tag{1.1}
$$
式(1.1)去除积分即为伽马分布的概率密度函数
$$
Gamma(x|\alpha)=\frac{x^{\alpha-1}e^{-x}}{\Gamma(\alpha)},(x>0,\alpha>0)
$$
令$$\beta t=x$$，代入积分公式(1.1)中则可得到新的概率密度函数
$$
Gamma(t|\alpha)=\frac{\beta^{\alpha}t^{\alpha-1}e^{-\beta t}}{\Gamma(\alpha)},(\beta t>0,\alpha>0)
$$
其中$$\alpha$$为shape parameter，主要决定了分布曲线的形状，$$\beta$$为scale parameter，主要决定曲线有多陡，$$\beta$$越大则曲线越陡。

伽马分布是为解决阶乘数列的插值而产生的，伽马函数使得任意正实数都有阶乘，伽马函数有几条特性如下:

1. $$\Gamma(x+1)=x\Gamma(x)$$
2. $$\Gamma(n)=(n-1)!$$

其中，$$\Gamma(\frac{1}{2})=\sqrt{\pi},\frac{1}{2}!=\Gamma(\frac{3}{2})=\frac{1}{2}\Gamma(\frac{1}{2})=\frac{\sqrt{\pi}}{2}$$

<font color='red' size=4>物理解释:</font>

泊松分布: 单位时间t内独立事件发生次数n的概率分布

指数分布: 独立事件的时间间隔的概率分布

伽马分布: $$\alpha$$个独立事件的时间间隔的概率分布

<font color="green" size=4>参考链接:</font>

* [伽马分布，指数分布，泊松分布的关系](https://www.jianshu.com/p/6ee90ba47b4a)

  这篇文章先讲述泊松分布，然后由泊松分布引出指数分布，说明了泊松分布与指数分布的关系，最后说明了伽马分布与指数分布的关系.

* [LDA-math-神奇的Gamma函数(3)]()

  这篇文章由二项分布和Beta分布的关系式，借用二项分布的极限是泊松分布的性质转换得到泊松分布与伽马分布的关系式，以及泊松分布的概率累积函数与伽马分布的概率累积函数互补的关系。



## Beta分布简介

Beta分布: 首先来看Beta函数，Beta函数又称贝塔函数或者第一类欧拉积分，它的函数式如下
$$
B(\alpha, \beta)=\int_{0}^{1}x^{\alpha-1}(1-x)^{\beta-1}dx=\frac{\Gamma(\alpha)\Gamma(\beta)}{\Gamma(\alpha+\beta)},(x \in [0,1],\alpha \gt 0, \beta \gt 0) \tag{2.1}
$$
对Beta函数作一个变形即可得到
$$
\int_{0}^{1}{\frac{x^{\alpha-1}(1-x)^{\beta-1}}{B(\alpha,\beta)}dx}=1,(\alpha>0,\beta>0)
$$
去除式(2.1)的积分即可得到Beta分布的概率密度函数
$$
X\sim Be(\alpha,\beta):Beta(x|\alpha,\beta)=\frac{1}{B(\alpha,\beta)}x^{\alpha-1}(1-x)^{\beta-1}=\frac{\Gamma(\alpha+\beta)}{\Gamma(\alpha)\Gamma(\beta)}x^{\alpha-1}(1-x)^{\beta-1},(\alpha>0,\beta>0)
$$
<font color="green" size=4>参考链接:</font>

* [LDA-math-认识Beta/Dirichlet分布(1)](http://www.52nlp.cn/lda-math-%E8%AE%A4%E8%AF%86betadirichlet%E5%88%86%E5%B8%831)

  这篇文章由掷骰子游戏从物理上解释了Beta分布的由来

* [LDA-math-认识Beta/Dirichlet分布(2)](http://www.52nlp.cn/lda-math-%E8%AE%A4%E8%AF%86betadirichlet%E5%88%86%E5%B8%832)

  这篇文章在上一篇文章的基础上通过更新掷骰子的游戏规则引出了Beta分布与二项分布的共轭分布关系，共轭的意思就是，数据符合二项分布的时候，参数的先验分布和后验分布都能保持Beta 分布的形式。我们知道贝叶斯的参数估计过程是
  $$
  先验分布\ + \ 数据的知识\ =\ 后验分布
  $$
  对应到beta分布和二项分布的共轭
  $$
  Beta(p|\alpha, \beta)\ +\ Binomial(m1|m1+m2,p)\ =\ Beta(p|\alpha+m1,\beta+m2)
  $$


## Dirichlet分布简介 

Multinomial分布: 中文名多项式分布
$$
Mult(\vec{n}|\vec{p},N)=\binom{N}{\vec{n}}\prod_{k=1}^{K}{p_{k}^{n_{k}}}
$$
Dirichlet分布: 中文名狄利克雷分布
$$
Dir(\vec{p}|\vec{\alpha})=\frac{\Gamma(\sum_{k=1}^{K}{\alpha_k})}{\sum_{k=1}^{K}\Gamma(\alpha_{k})}{\prod_{k=1}^{K}{p_{k}^{\alpha_{k}-1}}}
$$
<font color="green" size=4>参考链接:</font>

* [LDA-math-认识Beta/Dirichlet分布(3)](http://www.52nlp.cn/lda-math-%E8%AE%A4%E8%AF%86betadirichlet%E5%88%86%E5%B8%833)

  这篇文章由掷骰子的游戏引出Dirichlet分布，并且通过改变游戏规则引出Dirichlet分布与Multinomial分布的关系，但是在推导共轭关系时定义有些许错误要注意

* [LDA基础知识---Dirichlet 分布](https://v.youku.com/v_show/id_XMzI4NzI4Mjg5Mg==.html?spm=a2hzp.8244740.0.0)

  这是优酷上讲解LDA的Dirichlet分布的视频









