---
title: CTF中RSA套路
date: 2020-06-06 15:25:15
tags: 
- crypto

categories:
- CTF

math:
  enable: true

---
> 花了两天看把基础数论过了一遍，看这些东西的时候理解起来快了很多


# 基本类型：

## 1. "老奶奶用脚都会做"的板子题

已知e,p,q求d

$d$ = $e^{-1}$ mod $\phi(n)$

## 2. 已知e,d,n求p,q
用已知条件易于求出$N$、$\phi(n)$,则有

$$\begin{cases}
N = p * q \\\\[2ex]
\phi(n)=(q-1)(p-1)
\end{cases}
$$

**联立得**

$$
\begin{cases}
N-\phi(n)+1=p+q \\\\[2ex]
N=p  *  q
\end{cases}
$$

引入变量 X 建立一元二次方程（或者直接解二元一次）解出p和q

也不知道为什么有些地方给的办法那么麻烦。。。

## 3.低加解密指数攻击
1. 已知c和已知e过小就请直接爆破 
2. 低加密指数广播攻击->套用中国剩余定理模板
3. 低解密指数攻击->脚本梭哈,试了一下不好用

## 4.共模攻击

若有：
$gcd(e_1,e_2)=1$ 且 
$$
\begin{cases}
C_1\equiv M^{e_1} mod\;n \\\\[2ex]
C_2\equiv M^{e_2} mod\;n
\end{cases}
$$

一定有 $s_1  *  e_1+s_2  *  e_2= 1$,用扩展欧几里得算出 s1 ，s2

则有
$
\begin{cases}
C_1^{s_1}\equiv M^{e_1 * s_1} mod\;n \\\\[2ex]
C_2^{s_2}\equiv M^{e_2 * s_2} mod\;n
\end{cases}
\quad \Rightarrow C_1^{s_1} * C_2^{s_2}
\equiv M^{e_1  *  s_1 + e_2  *  s_2}mod\;n
\quad \Rightarrow C_1^{s_1} * C_2^{s_2}
\equiv M mod\;n
$

## 5.已知dp,dq求解

> 参考 7.5.2 使用中国余数定理快速加解密（CRT）

## 6.加密指数过大（e，n接近）
- wiener-attack

- [wiener脚本](https://github.com/pablocelayes/rsa-wiener-attack)

## 已知 n,e,d 求 q , p

> 比较冷的常规模板 做法比较...看脸

- [paper](https://www.di-mgt.com.au/rsa_factorize_n.html)

```python

import random


def gcd(a, b):
   if a < b:
	 a, b = b, a
   while b != 0:
	 temp = a % b
	 a = b
	 b = temp
   return a


def getpq(n, e, d):
	p = 1
	q = 1
	while p == 1 and q == 1:
		k = d * e - 1
		g = random.randint(0, n)
		while p == 1 and q == 1 and k % 2 == 0:
			k /= 2
			y = pow(g, k, n)
			if y != 1 and gcd(y-1, n) > 1:
				p = gcd(y-1, n)
				q = n/p
	return (p, q)


def main():
	'''
    n=
    e=
    d=
    '''

	p, q = getpq(n, e, d)
	print( "p="+hex(p))
	print( "q="+hex(q))


if __name__ == '__main__':
	main()

```








