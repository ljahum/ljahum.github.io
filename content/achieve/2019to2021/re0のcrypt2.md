---
title: 从零开始的深入浅出密码学day 2
date: 2020-06-05 17:22:55
tags: 
- crypto
categories:
- posts

math:
  enable: true

---

# 秘钥生成与正确性

> 算法不再赘述

## RSA秘钥生成

1. 选择两个大素数 p ,q
2. 计算 $n$ = $p * q$
3. 计算 $\phi$( n ) = $( p - 1 )( q - 1 )$
4. 选择 e $\in$ $Z^m$ 且 gcd（m，e）= 1
5. 计算私钥 d 满足 d * e = 1 mod $\phi$(n)


## RSA 核心原理

$C^d$ $\equiv$ $M^{d * e}$ $\equiv$ $M$ mod $n$

细节如下：

- $d * e = 1 + t * \phi(n)$
- $M^{d * e}\equiv M^{1 + t * \phi(n)}\equiv M * M^{t * \phi(n)}\equiv 1 * M$ mod $n$

# 7.5.2 使用中国余数定理快速加解密（CRT）

1. 约简基元素 $M$：

    $M_p$ $\equiv$ $M$ mod $p$

    $M_q$ $\equiv$ $M$ mod $q$

2. 计算两个指数并进行列出的指数运算
    
    两个指数：

    $d_p\equiv d$ mod $(p-1)$
    
    $d_q\equiv d$ mod $(q-1)$

    运算：

    $C_p\equiv M^{d_p}_p$ mod $p$

    $C_q\equiv M^{d_q}_q$ mod $q$

3. 逆向换到问题域：

    $C$ $\equiv$ $C_p * M_p * q$ + $C_q * M_q * p$ mod $n$

> 如果要解密也可以用这种算法快速算出结果










