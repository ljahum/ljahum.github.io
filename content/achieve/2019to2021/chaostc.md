---
title: Chaos encrypt system
date: 2021-07-31 00:07:00+08:00
author: ljahum 
description: "widely used in medical treatment"
tags: 
- crypto
- math

categories: 
- posts

hiddenFromHomePage: false
math:
  enable: true
---
<!--more-->

采用的原因：早期的图像加密方法主要基于现有的数据加密技术，如DES、AES等。然而，由于图像本身的固有特性，这些方法的效率和抗攻击能力都较弱。近年来，基于混沌的图像加密方法越来越受到重视。与传统的非混沌图像加密相比，基于混沌的图像加密具有密钥空间大、速度快、易于实现等优点

图像文件加密与普通文件加密的不同之处在于，图像相邻像素之间通常存在着比较大的相关性，对图像加密不仅要使图像变得不可识别，还要尽可能地减小相邻像素之间的相关性。

## 虫口模型 Logistic 混沌映射

> 如果一个系统的演变过程对初始的状态十分敏感，就把这个系统称为是混沌系统。
>
> 在本文中，我们主要探讨一维*Logistic*映射的一些特性

一维*Logistic*映射从数学形式上来看是一个非常简单的混沌映射，早在*20*世纪*50*年代，有好几位生态学家就利用过这个简单的差分方程，来描述种群的变化。此系统具有极其复杂的动力学行为，在保密通信领域的应用十分广泛，其数学表达公式如下：


$X_{n+1}=X_n \times \mu \times (1-X_n)$

$\mu \in[0,4]\;X\in[0,1]$

其中$\mu$被称为*Logistic*参数。研究表明，当$X\in[0,1]$时，Logistic 映射工作处于**混沌状态**，

也就是说，有初始条件$X_n$在*Logistic*映射作用下产生的序列是**非周期的、不收敛的**，而在此范围之外，生成的序列必将收敛于某一个特定的值

![](https://raw.githubusercontent.com/ljahum/images/main/img/20210731124042.png)

可以看出，在μ的取值符合3.5699456<μ<=4的条件，特别是比较靠近4时，迭代生成的值是出于一种伪随机分布的状态，而在其他取值时，在经过一定次数的迭代之后，生成的值将收敛到一个特定的数值，这对于我们来说是不可接受的。

下图中描述了X0值一定时，对于不同的μ的取值，迭代可能得到的值：


![](https://raw.githubusercontent.com/ljahum/images/main/img/20210731124201.png)

图中的点即表明了所有可能的X取值范围。从图中我们可以看出，在μ越接近4的地方，X取值范围越是接近平均分布在整个0到1的区域，因此我们需要选取的Logistic控制参数应该越接近4越好。当3.5699456...<*μ*≤43.5699456...<μ≤4时，映射进入混沌(chaos)区域。Logistic映射分岔图像如图1所示。现在这类模型是人们最常见的，更是广为使用的。

在μ的值确定之后，我们再来看看初始值X0对整个系统的影响。刚才也说过了，混沌系统在初始值发生很小变化时，得到的结构就会大相径庭，在Logistic混沌映射中也是如此。

## **一种像素灰度值替代设计图像加密**

设图像$(i,j)$处的灰度值为 $I(i,j)$,$I'(i,j)$表示替换后的值

本文中，像素值的替代变换是在空域中进行的，一般，设计了两种思路用于实现混沌序列与像素值的替换操作。

$I′(i,j)=((r1(i,j)⊕I(i,j)⊕r2(i,j)+L−r3(i,j)))\;mod\;L)\;mod\;256$

$L$表示图像的颜色深度

$r1,r2,r3$表示的是混沌序列值，替换变换的**密钥**由$r1,r2,r3$对应的混沌系统提供，变换可多次进行，如此加密效果更好。设重复次数为 n ，与混沌模型的初值和参数共同作为这一部分的密钥，增大了密钥的空间

若图像很大时，通过上式能够看出r1,r2,r3模版矩阵需要随之增大，如此就大大减小了加密效率。为此，我们可以通过分块处理的方式对图像进行加密，加密效率明显提高。图2是原始图像和加密后的图像:

![](https://raw.githubusercontent.com/ljahum/images/main/img/20210731130116.png)

##  混沌理论Chaos theory

Chaos theory is a branch of mathematics focusing on the study of chaos — dynamical systems whose apparently random states of disorder and irregularities are actually governed by underlying patterns and deterministic laws that are highly sensitive to initial conditions

> 大意是混沌系统是数学的一个分支
> 
> 这里利用一些自然中对初始条件及其敏感的模型来输出加密序列？

