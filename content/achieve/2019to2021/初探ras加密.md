---
title: 初探ras加密
date: 2020-03-05 00:35:13
tags: 
- crypto
categories:
- posts

math:
  enable: true

---
## 初探RAS加密

### 加密准备:

- 找两个比较大的质数 p 、q

- 设 n = q * p

- 设 f(n) = (p-1) * (q-1)

- 找公钥 e 满足：1<e<f(n) 且 e、f(n) 互质

- 找私钥 d 满足 : (d * e)% f(n) = 1

### 加密算法:

- 明文 M 和密文 C 满足： M<sup>e</sup> % n= c , C<sup>d</sup> % n=M（明文和密文都被事先转换为数字）

### 已知M、e、n时RAS的如何解密（限于q，p不大的离谱的情况进行攻击）:

- 要找 q，p，所以要**对 n 因式分解**

- 有了 f(n) 就可以**算 e*d %f(n) =1的逆模**，利用计算机可以轻松求出 ( 已知 f(n) 又知 e 求 d 反之同理 )

# 多因子RSA加密：

- 找多个质数 P0 ， P1 ………… Pn

- n = P0 * P1 ..... Pn

- phi = (P0-1) *......* （Pn-1）

其余步骤与常规rsa一致
