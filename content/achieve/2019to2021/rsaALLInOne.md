---
title: RsaAllInOne
date: 2021-06-14 00:07:00
author: ljahum 

tags: 
- codes
categories: 
- posts
---




> 一些RSA垃圾题

## normal
### rsa1 p-q

p q相差太小,直接用费马方法分解

```python
from Crypto.Util.number import *
from gmpy2 import next_prime
import random

## p = getPrime(512)
## q = next_prime(next_prime(p) + random.randint(2 ** 10, 2 ** 15))
## q = p+a
## 费马方法分解n
def fermat_factors(n):
    assert n % 2 != 0
    import gmpy2
    a = gmpy2.isqrt(n)
    b2 = gmpy2.square(a) - n
    while not gmpy2.is_square(b2):
        a += 1
        b2 = gmpy2.square(a) - n
    factor1 = a + gmpy2.isqrt(b2)
    factor2 = a - gmpy2.isqrt(b2)
    return int(factor1), int(factor2)


n = 58469790767119395443619182703965753536155769938155967209185013051235434307443199577853487462032941284716788878629026151008480533108948515487216969655522610052504252431114883354036178747396340974017983797943561003427523330887483816814526450542542017962396566419907954878575664402091503063651747784708370988551
e = 65537
c = 5210792629811531618748922441951091043558836768927486327066208193531783313814007532484357434613606536101786384803692759877417618085066111343186477103523861319942108843549903015256729449831006717779587476869110804323636543879652560144414279029912184440958215613248124698646241943161894215574508920550451749893
q, p = fermat_factors(n)
d = inverse(e, (p - 1) * (q - 1))
print(long_to_bytes(pow(c, d, n)))
```

flag b'flag{this_is_flag}'



### rsa4 ed2n

用以下脚本来求qp

```python
def getpq(n,e,d):
    while True:
        k = e * d - 1
        g = random.randint(0, n)
        while k%2==0:
            k=k//2
            temp=gmpy2.powmod(g,k,n)-1
            if gmpy2.gcd(temp,n)>1 and temp!=0:
                return gmpy2.gcd(temp,n)

```


### rsa 5 EeE

小公钥加密，尝试队c直接开三次方

```python
c=74802199268289650659966949121722134398741724016029787984879330914681382911392412708797079066742193788624174545434065877798917466086322130161380099507267024211772226618358174709389234588799169125
import gmpy2
m = gmpy2.iroot(c,3)
print(m)
```


### rsa7 Radin


1. 使用python脚本

代码：

```python
from Crypto.PublicKey import RSA

rsakey = RSA.importKey(open("public.key", "r").read())
n = rsakey.n
e = rsakey.e
print(,n,e)
```

带私钥版本：

```python

rsakey = RSA.importKey(open("prikey.key", "r").read())
n = rsakey.n
e = rsakey.e
d = rsakey.d
q = rsakey.q
p = rsakey.p
print(e,d,n,p,q)
```

Rabin算法的解密原理是：假设我们知道m%p 和 m%q，那么拿着中国剩余定理立刻可以知道 m%n的值。又有m<n，则m的值就直接拿到了，岂不美哉？

实际上标准的Rabin算法的解密会得到四个结果。我们一一对比就可以找到flag



专门用于解radin的脚本：

```python
from Crypto.Util.number import *
def squareMod(c, mod):          ## 模意义下开根，找到 x, 使得 x^2 % mod = c
    assert(mod % 4 == 3)
    res = gmpy2.powmod(c, (mod+1)//4, mod)
    return res, mod - res

def getPlaintext(x, y, p, q):   ## 假设 m%p=x, m%q=y, 求明文
    res = x*q*gmpy2.invert(q, p) + y*p*gmpy2.invert(p, q)
    return res % (p*q)

def solve(c, p, q):             ## 已知 p,q, 解密 c
    px = squareMod(c, p)
    py = squareMod(c, q)

    for x in px:
        for y in py:
            yield getPlaintext(x, y, p, q)

c = open('flag.enc', 'rb').read()
## print(c)
c = int(c.hex() ,16)

p = 275127860351348928173285174381581152299
q = 319576316814478949870590164193048041239
for msg in solve(c, p, q):
    print(long_to_bytes(msg))
```





## RSA oracle

### rsa10 nctf2020

```python
from Crypto.Util.number import getPrime, inverse, GCD, bytes_to_long
# from secret import flag

flag = b'flag{xxxxxxxxxxxxxxxxxx}'
m = bytes_to_long(flag)

while True:
    p = getPrime(512)
    q = getPrime(512)
    n = p * q
    e = getPrime(32)
    if GCD((p-1)*(q-1), e) == 1:
        d = inverse(e, (p-1)*(q-1))
        break

print(e, n, pow(m, e, n), sep='\n')


for _ in range(10000):
    cc = int(input("> "))
    mm = int.to_bytes(pow(cc, d, n), 1024//8, 'big')
    print(mm.startswith(b"\x00"))
```

先确定flag长度，用二分法无限逼近flag的值

```python
from pwn import *

from Crypto.Util.number import *
r = remote("0.0.0.0",10001)
buf = r.recvline()
e = int(buf.decode().strip())
n = int(r.recvline().decode().strip())
c = int(r.recvline().decode().strip())
max = 2**826    
min = 2**822
while (max-min>1):
    mid = (max+min)//2
    temp = (pow(mid,e,n)*c)%n
    r.sendline(str(temp))
    data = r.recvline()
    print(data)
    ## data = r.recvline()
    if(b'True' in data):
        min = mid 
    else:
        max = mid
print(min,max)
print(long_to_bytes(2**1016//min))
print(long_to_bytes(2**1016//max))

    

```


## rsa 格规约攻击

### rsa 12 fac with hit

利用coppersmith获取p

```python
n = 0x5894f869d1aecee379e2cb60ff7314d18dbd383e0c9f32e7f7b4dc8bd47535d4f3512ce6a23b0251049346fede745d116ba8d27bcc4d7c18cfbd86c7d065841788fcd600d5b3ac5f6bb1e111f265994e550369ddd86e20f615606bf21169636d153b6dfee4472b5a3cb111d0779d02d9861cc724d389eb2c07a71a7b3941da7d
p_fake = 0x5d33504b4e3bd2ffb628b5c447c4a7152a9f37dc4bcc8f376f64000fa96eb97c0af445e3b2c03926a4aa4542918c601000000000000000000000000000000000
pbits = p_fake.nbits()
##kbits = 900
kbits = 128  ##p失去的低位
pbar = p_fake & (2^pbits-2^kbits)
print "upper %d bits (of %d bits) is given" % (pbits-kbits, pbits)
 
PR.<x> = PolynomialRing(Zmod(n))
f = x + pbar
 
x0 = f.small_roots(X=2^kbits, beta=0.4)[0]  ## find root < 2^kbits with factor >= n^0.3
p= x0 + pbar
print p
```




### rsa13 bigger d

### rsa14Partial Key

### rsa 15 Wiener plas

### rsa17 Linear math

一次论文攻击方式复现
```python
[Window Title]
Update Available

[Main Instruction]
A new version of Sublime Text is available, download now?

[Download] [取消]
```


```python
from libnum.common import gcd
from data import b,c
from gmpy2 import *
from Crypto.Util.number import *
import time
import libnum
n = 110384114201475663616747380525627123445579689285630886040168434260829091685143698389730908307058264757595472490646553684116819186297946861038089011634839837455006286876809812362253126020273844190061817352704628672775002140119322152762726493845024580159588306209024297015077439731424454595795781027348887941827
a = 877
t = 541
e = 0x101

strat = time.time()
Pk = []
for k in range(e):
    tot = 1
    ## print(k)
    for k1 in range(e):
        if k!= k1 :
            ## print(gcd((b[k]-b[k1])%n,n))
            tot *= libnum.invmod((b[k]-b[k1])%n,n)
            tot %= n
        tot %= n 
    Pk.append(tot)


v=0
for k in range(e):
    v += (pow(b[k],e,n)*Pk[k])%n
    v %= n

tmp = 0
for k in range(e):
    tmp += (c[k]*Pk[k])%n
    tmp %=n
x = ((libnum.invmod(e,n)*(tmp-v))%n)%n
from Crypto.Util.number import *
print(x*libnum.invmod(a,n)%n)
end = time.time()
print(end - strat)

```



### rsa21 half p

题目给出了dp和一对公钥

已知

$dp+k*(p-1)=d\;mod\;p-1$

$ed=1=k*(p-1)*(q-1)+1\;mod\;(p-1)*(q-1)$

$ed=e*dp\;mod\;p-1$

$k_1*(p-1)*(q-1)+1=k_2(p-1)+dp*e$

可写为

$(p-1)=(dp*e-1)/x$

由式子可知x小于e，若e较小，对e遍历就可以得到p q

```python
import gmpy2
from Crypto.Util.number import bytes_to_long, long_to_bytes
n = 82459095549748227929288050555498384469575272567666482697551651645093561076800636903414213079689167610546931321044499258842967876311854597995877424799325872420793993678223520863171113084551896184041824490162048421763392587578106892564418656377436734864113039255928135310184434095934015134373147964955532123881
dp = 2154496166987404807570061246274794378102105725450715702694091896971248930139905533434215910296644935717134018627774991211321921063947893592214392985692261
e = 65537
c = 31011021137992452431076840432639365407713019786586660934070807380893254876293446768579394082019427413952266893686196624551302747440454439484165217862543849146153312457417254751109386482402322429697563463538223851046832003654970628089968896006236813877570702218773093761582493452679741468626986210359922340714

for i in range(1, e):
    if (e * dp - 1) % i == 0:
        p = (e * dp - 1) // i + 1
        if n % p == 0:
            q = n // p
            phin = (p - 1) * (q - 1)
            d = gmpy2.invert(e, phin)
            print(long_to_bytes(pow(c, d, n)))
            break

```



### rsa26 partial message

构造多项式$f=（m+x）^3-c=0\;mod\;n$



```python
def phase2(high_m, n, c):
    R.<x> = PolynomialRing(Zmod(n), implementation='NTL')
    m = high_m + x
    M = (m^3 - c).small_roots()[0]
    print(M)

n = 13112061820685643239663831166928327119579425830632458568801544406506769461279590962772340249183569437559394200635526183698604582385769381159563710823689417274479549627596095398621182995891454516953722025068926293512505383125227579169778946631369961753587856344582257683672313230378603324005337788913902434023431887061454368566100747618582590270385918204656156089053519709536001906964008635708510672550219546894006091483520355436091053866312718431318498783637712773878423777467316605865516248176248780637132615807886272029843770186833425792049108187487338237850806203728217374848799250419859646871057096297020670904211
c = 15987554724003100295326076036413163634398600947695096857803937998969441763014731720375196104010794555868069024393647966040593258267888463732184495020709457560043050577198988363754703741636088089472488971050324654162166657678376557110492703712286306868843728466224887550827162442026262163340935333721705267432790268517
high_m = 0x464c41477b325e38727361373538393639336663363839633737633566353236326436000000000000000000


from Crypto.Util.number import *
print(long_to_bytes(2519188594271759205757864486097605540135407501571078627238849443561219057751843170540261842677239681908736+phase2(high_m, n, c)))
```

FLAG{2^8rsa7589693fc689c77c5f5262d654272427}

### rsa27 partial d and n c

```python
from Crypto.Util.number import *
def getFullP(low_p, n):
    R.<x> = PolynomialRing(Zmod(n), implementation='NTL')
    p = x*2^512 + low_p
    root = (p-n).monic().small_roots(X = 2^128, beta = 0.4)
    if root:
        return p(root[0])
    return None
    
def phase4(low_d, n, c):
    maybe_p = []
    for k in range(1, 4):
        p = var('p')
        p0 = solve_mod([3*p*low_d  == p + k*(n*p - p^2 - n + p)], 2^512)
        maybe_p += [int(x[0]) for x in p0]
    print(maybe_p)
    
    for x in maybe_p:
        P = getFullP(x, n)
        if P: break
    
    P = int(P)
    Q = n // P
    
    assert P*Q == n
    
    d = inverse_mod(3, (P-1)*(Q-1))
    ## print(hex()[2:])
    print(long_to_bytes(power_mod(c, d, n)))


n = 92896523979616431783569762645945918751162321185159790302085768095763248357146198882641160678623069857011832929179987623492267852304178894461486295864091871341339490870689110279720283415976342208476126414933914026436666789270209690168581379143120688241413470569887426810705898518783625903350928784794371176183
c = 56164378185049402404287763972280630295410174183649054805947329504892979921131852321281317326306506444145699012788547718091371389698969718830761120076359634262880912417797038049510647237337251037070369278596191506725812511682495575589039521646062521091457438869068866365907962691742604895495670783101319608530
low_d = 787673996295376297668171075170955852109814939442242049800811601753001897317556022653997651874897208487913321031340711138331360350633965420642045383644955
phase4(low_d, n, c)

## FLAG{2^8rsa5ab086745f6ec745619a8b65fe4ec560}
```

### rsa 28 bit Oracle

```python
e = 0x10001
n = 0x0b765daa79117afe1a77da7ff8122872bbcbddb322bb078fe0786dc40c9033fadd639adc48c3f2627fb7cb59bb0658707fe516967464439bdec2d6479fa3745f57c0a5ca255812f0884978b2a8aaeb750e0228cbe28a1e5a63bf0309b32a577eecea66f7610a9a4e720649129e9dc2115db9d4f34dc17f8b0806213c035e22f2c5054ae584b440def00afbccd458d020cae5fd1138be6507bc0b1a10da7e75def484c5fc1fcb13d11be691670cf38b487de9c4bde6c2c689be5adab08b486599b619a0790c0b2d70c9c461346966bcbae53c5007d0146fc520fa6e3106fbfc89905220778870a7119831c17f98628563ca020652d18d72203529a784ca73716db
c = 0x4f377296a19b3a25078d614e1c92ff632d3e3ded772c4445b75e468a9405de05d15c77532964120ae11f8655b68a630607df0568a7439bc694486ae50b5c0c8507e5eecdea4654eeff3e75fb8396e505a36b0af40bd5011990663a7655b91c9e6ed2d770525e4698dec9455db17db38fa4b99b53438b9e09000187949327980ca903d0eef114afc42b771657ea5458a4cb399212e943d139b7ceb6d5721f546b75cd53d65e025f4df7eb8637152ecbb6725962c7f66b714556d754f41555c691a34a798515f1e2a69c129047cb29a9eef466c206a7f4dbc2cea1a46a39ad3349a7db56c1c997dc181b1afcb76fa1bbbf118a4ab5c515e274ab2250dba1872be0
from pwn import *
left=0
right=n
num=0
import gmpy2
while right-left>2:
    num+=1
    tmp=gmpy2.powmod(2,num*e,n)
    senddata=hex((c*tmp)%n)[2:]
    ## 111.200.241.244:62839
    io=remote('111.200.241.244',62839)
    io.recv()
    io.sendline(senddata)
    ans=io.recvline().decode()[:-1]
    print(ans)
    if ans=='odd':
        left=(left+right)//2 if (left+right)%2==0 else (left+right)//2+1
    else:
        right=(left+right)//2 if (left+right)%2==0 else (left+right)//2+1
    print(right-left)
    print(num)
    io.close()
print((left,right))
import gmpy2
while gmpy2.powmod(left,e,n)!=c:
    left-=1
print(left)
from  Crypto.Util import number
print(number.long_to_bytes(left))

```