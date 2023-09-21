---
title: python密码学常用库
date: 2020-03-30 00:05:31
tags: 
- codes 
- crypto
categories:
- posts

---
### SymPy中数论部分使用说明

> sympy是一个Python的科学计算库，用一套强大的符号计算体系完成诸如多项式求值、求极限、解方程、求积分、微分方程、级数展开、矩阵运算等等计算问题 :D

sympy.factorint

list(sympy.sieve.primerange(数字1,数字2))列出大于等于数字1，小于数字2的所有素数

sympy.prime(n)返回第n个素数

sympy.isprime(n)素性检测

sympy.primepi(n)返回小于n的素数的总数

sympy.nextprime(89)返回下一个素数，这里结果是97

sympy.prevprime(96)或sympy.prevprime(97)返回上一个素数，结果都是89

sympy.randprime(1,30)返回1到30之间的一个大于等于1小于30的随机素数 range [a, b)

```python
sympy.primorial

>>> primorial(4) # the first 4 primes are 2, 3, 5, 7

210

>>> primorial(4, nth=False) # primes <= 4 are 2 and 3

6
```
### libnum库
> 大体功能和 sympy 比较相似，python3.7+的小伙伴可以用这个

has_invmod (e，n)检测是否有逆模

d = invmod (e, n) 求逆模，满足关系 d * e = 1  mod n (gmpy2.invert)

gcd(a, b) 求两数最大公约数(欧几里得算法)

xgcd(a, b) 扩展欧几里得 返回（x，y，g）：a * x + b * y = gcd（a，b）= g

总之功能就是非常多啦。。。https://github.com/JafarAkhondali/python3-libnum

### Crypto.Util.number
>  C y p t o , 永 远 滴 神 ~~~

l = bytes_to_long(b)
 
b = long_to_bytes(l) 这两个不用说好用到爆

getPrime(n) 返回一个随机的N位bit的素数

d = inverse(e, n) 同样求逆模


https://www.pycryptodome.org/en/latest/src/introduction.html