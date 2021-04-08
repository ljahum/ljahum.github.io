# DemytkoSystem

<!--more-->
# The Demytko System

仿照rsa设计的基于ecc的加密算法

---

> 别问我定理原理 , 我只会用

refer : Public Key Cryptosystems Using Elliptic Curves p47 p28

## 互补曲线的基本性质

$\#{E_p(a,b)}$表示$E$上的点数

## 前提:椭圆曲线圆的点数

定理 2.12： 

$\#{E_p(a,b)}=1+\sum_0^{p-1}(1+(\frac{x^3=ax+b}{p}))$

$E:y^2=x^3+ax+b$

我们在$F_p\times \{ u\sqrt{v},u\in\;F_p\}$的的子集上定义一个组结构

v是一个固定的模p二次非剩余

注意 $\{ u\sqrt{v},u\in\;F_p\}/\{0\}$是一个循环群,对于vg个模p的非剩余

**定义 2.6:**

- 取$p>3$作为一个素数,
- $a,b\in\;F_p$,
- $v$是一个模$p$下的二次非剩余 $v\in\;F_p$

    有曲线     $y^2=x^3+ax+b$

    当$x\in\;F_p,u\in\;F_p,y=u\sqrt{v}\;mod\;p$时

称该曲线$\overline{E_p(a,b)_v}$为${E_p(a,b)}$的一个互补曲线

**定理 2.20:**

对互补曲线上的点的个数(曲线的阶)有一下定理:

(注意,这里的$(\frac{a}{b})$为勒让德符号,不要认成除法了)

因为有 : $\#\overline{E_p(a,b)}=1+\sum_0^{p-1}(1-(\frac{x^3=ax+b}{p}))$

所以有 : $\#\overline{E_p(a,b)}+\#{E_p(a,b)}=2p+2$

## Demytko 加密

对于每一个用户都有一个专属的曲线$P_u\;Q_u$构成$N_u=P*Q$

**Setup:**

$n_1到n_4为p和q曲线以及互补曲线上的总点数$

![](https://i.loli.net/2021/04/08/RApv1EUzgqrluaZ.png)



公钥 $e$满足gcd(e,$N_i$)=1

四个私钥分别满足:

![](https://i.loli.net/2021/04/08/OalX8bFSvxWcHKo.png)





### 加密

$C=e_b*m$

### 解密

![](C:\Users\16953\Pictures\typora图片\3D2TYU41w6k5bfV.png)



### 体现在代码上

```python
def Dec(self, ciphertext):
        x = ciphertext
        # w=y^2?
        w = x^3 + self.a*x + self.b % self.N
#==========这里用CRT得到准确的Y============================================
        P.<Yp> = PolynomialRing(Zmod(self.p))
        fp = x^3 + self.a*x + self.b -Yp^2
        yp = fp.roots()[0][0]

        P.<Yq> = PolynomialRing(Zmod(self.q))
        fq = x^3 + self.a*x + self.b -Yq^2
        yq = fq.roots()[0][0]

        y = crt([int(yp), int(yq)], [self.p, self.q])
#======================================================
        cip_point = self.E.point([x, y])

        legendre_symbol_p = legendre_symbol(w, self.p)
        legendre_symbol_q = legendre_symbol(w, self.q)
        msg_point = self.d[(legendre_symbol_p, legendre_symbol_q)]*cip_point

        return msg_point.xy()[0] >> self.Kbits
```

## 解密正确性证明:

![](https://i.loli.net/2021/04/08/jcMAKn3uhakpXO7.png)





- $w^2=c^3+ac+b$

## 对P

- 如果$(\frac{w}{p})=1$,$c^3+ac+b是一个莫p的二次剩余$

则这里有一个点在$E_P(a,b)$其X为密文数据$C$

$C$是$e_A$和X轴为m的且同样在$E_P(a,b)$上的点相乘得到的

- 如果$(\frac{w}{p})=-1$

$c^3+ac+b是一个莫p的二次非剩余$

则这里有一个点在$E_P(a,b)$的互补曲线  $\overline{E_P(a,b)}_v$  上

其 X 轴为密文数据 $C$

$C$是$e_A$和X轴为m的且同样在$E_P(a,b)$的互补曲线  $\overline{E_P(a,b)}_v$ 上的点相乘得到的

这里的   $v$  指一个模 p 的二次非剩余

对q也可以用同样的描述解释 

 $public\;key\;cryptosystems\;using\;ECC$中

定理 2.23 决定解密操作的正确性

![](https://i.loli.net/2021/04/08/jcMAKn3uhakpXO7.png)




## 定理 2.23(人话版本):

$E_1\in\{E_p(a,b),\overline {E_p(a,b)_v\}}$

$E_2\in\{E_p(a,b),\overline {E_p(a,b)_w\}}$

设$N_n=lcm(\#E_1,\#E_2)$    (#E 为曲线上的点 or 曲线的阶)

$E_n^{c}$由N1 N2 组成

对于所有$P\in E_n^{c}$有:

$(KN_n+1)P=P \;on\;E_n^{c}$

而利用勒让德符号能有效判断$P$是否属于互补曲线上,从而确定$E_1和E_2$的选取

## 其他笔记:

## Theorem 2.13

![](https://i.loli.net/2021/04/08/jQnHJdBrZEsz7gt.png)


## Theorem 2.15

![](https://i.loli.net/2021/04/08/lXIufJTG2kjdt7F.png)

