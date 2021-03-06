# 爱德华曲线-蒙哥马利.md


[toc]

# sm2

SM2是[国家密码管理局](https://baike.baidu.com/item/国家密码管理局/2712999)于2010年12月17日发布的椭圆曲线公钥密码算法。

## ecc回顾



1. 选取曲线 Ep(a,b)
1. Alice取 基点G和私钥k,计算公钥 pubk.传pubk和G给bob
1. bob计算 C1=M+rK；C2=rG传给alice
1. A接到信息后，计算C1-kC2，结果就是点M

描述一条Fp上的椭圆曲线，常用到六个参量：T=(p,a,b,G,n,h)

p a b用来确定曲线

G为基点，n为点G的阶，h 是椭圆曲线上所有点的个数m与n整除

# 爱德华曲线简介

形如

$x^2+y^2=1-d\cdot x^2 \cdot y^2$

## 加法

p1+p2

![https://img.learnblockchain.cn/2020/10/21/16032559470087.jpg](https://img.learnblockchain.cn/2020/10/21/16032559470087.jpg)

2*p

![img](https://img.learnblockchain.cn/2020/10/21/16032561222472.jpg)

逆元

（x，y）->(-x,y)

零元

（0，1）

#  爱德华曲线运算的几何意义



先考虑常规圆

先建立一个但为远$x^2+y^2=1$

<img src="https://dm2302files.storage.live.com/y4mEDmTtnbaTfHUnckxRG7aJo7v8sGJCk3FgfjnoOHzJeCAGSDQuYXHLZg73-yu8AADKPxf9e8QGqAcSK6jiJj3W2ctR6t_Ptd2k1CyytU7BBIAh0Xs8tyDMgY5Cr8dBwB_yoL1GR33PTTLwQZiX9PeRG-gZEj-Nbqyyd8IbiuaQWIIWgpwxo4VYC84P-docN3m?width=660&height=297&cropmode=none" width="660" height="297" />



考虑一个爱德华单位曲线

<img src="https://dm2302files.storage.live.com/y4mmj6OL31DJou99Ko3_sfF75WNWs9Kq6i31VQ9-tM_B2d_mdgbRjO-vvk3u7M8nrWBUTKao2SQYFw5yV2tN1kPsgCbFnF2sa9yQFROSb6IH96DO3pBKHBBSsOpNN7CHZhFeKMoh_4-Qn8W_ATLE8D6aphkEZwvqPK95Q2-ixfxzEBI7Q-ZdrmhvFbMJTFEp4vJ?width=615&height=660&cropmode=none" width="615" height="660" />

常规爱德华曲线基本性质就这些了

# 蒙哥马利曲线 Ed25519 签名

蒙哥马利曲线（Montgomery curve）是另一种形式的椭圆曲线

常规ecc

$y^2+a_1xy+a_3y=x^3+a_2x^2+a_4x+a_6$

蒙哥马利曲线

$By^2=x^3+Ax^2+x$

<img src="https://dm2302files.storage.live.com/y4mwDuFHPEZelA3A5TwaWOTHNhnkuy2MdGETIieJIYv_Ifakm_FCixgDn0KjCxZDpXqB9d1tnEtFHPuiv53_uhcUbSxkIO5iIyavN9tg324OUZfwdzNJbMHWvh90TPsBBvL60LqQrw3-HaKPC7iB_yHxod_iiJ91UoclTYz7qnvLNmZjab_sp34fDGX04gi1Kuh?width=476&height=660&cropmode=none" width="476" height="660" />








