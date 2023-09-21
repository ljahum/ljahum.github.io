---
title: 从零开始的深入浅出密码学day 1
date: 2020-06-04 23:50:59
tags: 
- crypto
categories:
- posts

math:
  enable: true

---

> 今天深入了解了一下 RSA 背后的一些数学原理

# 1.4.1 模运算 

## .2 余数的不唯一性
例：

- 12 $\equiv$ 3 mod 9
- 12 $\equiv$ 21 mod 9
- 12 $\equiv$ -6 mod 9

12、3、21、-6都属于一个整数集 {...-27,-15,-6,3,12,21....} 这个整数集构成一个所谓的等价类 

对于模数9来说还拥有另外 8 个等价类

{....-9 ,0 ,9....}

{....-8 ,1 ,10....}

.
.
.

{....-10 ,-1 ,8....}

对于模运算来说,等价类所以成员的**行为**相同

# 1.4.2 环

## 定义

假设整数环 Z<sub>m</sub> 有以下两部分构成：

1. 集合 Z<sub>m</sub> = {0 ,1 ,2 .... m-1}
2. 两种操作符 +、* 使a 、b $\in$ Z<sub>m</sub>,有：

a + b $\equiv$ c mod m

a * b $\equiv$ d mod m

## 环中的逆元

一般逆元定义：
> 一个可以取消另一给定元素运算的元素

但在这里，乘法逆元定义为：a * a<sup>-1</sup> $\equiv$ 1 mod m 

故 3 * 9 $\equiv$ 1 mod 26 中 9 是 3 的逆元

**gcd(a ,m) = 1时  Z<sub>m</sub> 中 a 的逆元存在**（互质）

# 6.3.2 扩展欧几里得算法(EEA)

gcd()用熟悉的辗转相除法发介绍了如何求最大公因数, EEA 算法则最终可以求得 Z<sub>m</sub> 中 a 的逆元 （前提是存在逆元）

设正整数 r0 ， r1 且 r0>r1

EAA算法最终可以得到两份参数：

1. gcd(r0 ,r1) 返回的最大公因数
2. gcd(r0 ,r1) = s * r1 + t * r0 中的 t 、s

当 gcd( m , a) = 1 时，有 s * m + t * r1 = 1

则 s * 0 + t * r1 $\equiv$ 1 mod m

t 为 r1 的逆元

# 几个基础的函数和定理

> 基础中的基础

## 欧拉函数

> 返回 Z<sub>m</sub> 内与 m 互质的整数的个数

$\phi$( m ) = $\prod_1^n$ ( P<sub>i</sub><sup>e</sup> - P<sub>i</sub><sup>e-1</sup> )

例

$\phi$( 240 ) = 2<sup>4</sup> * 3 * 5 = ( 2<sup>4</sup> - 2<sup>3</sup> )( 3 - 1 )( 5 - 1 ) = 64

## 费马小定理

a<sup>p</sup> $\equiv$ a mod p  --> a<sup>p-1</sup> $\equiv$ 1 mod p

## 欧拉定理
a、m都是整数，且互质，则有:
a <sup>$\phi$( m )</sup> $\equiv$ 1 mod m
















