---
title: star CTF2021
date: 2021-01-05 19:05:14
tags:
- crypto
categories:
- CTF

math:
  enable: true

---
# *ctf2021

## myenc

```python
	ct=iv
	for i in range(1,8):
		if keystream[cnt]=='1':
			ct+=pow(m^q,i**i**i,n)
			ct%=n
		cnt=(cnt+1)%len(keystream)
	print("done:",ct)

```

主要难点在这里

`pow(m^q,i**i**i,n)`无法正常求解，直接看做7个不变的未知数，外加iv，就是8个不变的未知数，keystream看做向量k，可以把原题看做以下矩阵
$$

\begin{cases}
k_1x_1 +&  \cdots & k_7x_7\;\;+iv=C_1\\\\
k_8x_1 + & \cdots & k_{14}x_7 \;+iv=C_2\\\\
\vdots  & \ddots & \vdots \\\\
k_{113}x_1 +& \cdots & k_{119}x_7  \\;+iv=C_{17}\\\\
k_{120}x_1 +&  \cdots & k_6x_7\\;\\;\\;+iv\;=C_{18}\\\\
k_{7}x_1 +&  \cdots & k_{13}x_7\\;\\;+iv\\;=C_{19}\\\\
\vdots
\end{cases}


$$

对k1到k7*8爆破是不明智的，选取其他等式组合一下，目标是尽可能用少的k覆盖多的x：
$$
\begin{pmatrix}
k_8 & \cdots & k_{14}&1\\\\
k_7 &  \cdots & k_{13} &1\\\\
\vdots & \vdots & \ddots & \vdots \\\\
k_{1}& \cdots & k_{7}  &1
\end{pmatrix}

\begin{pmatrix}
x_1\\\\
x_{2}\\\\
\end{pmatrix}

\begin{pmatrix}
C_2\\\\
C_{19}\\\\
\end{pmatrix}

$$

这个时候对 K 矩阵爆破的复杂度只有$O(2^{14})$写脚本获取121+组数据一把梭哈就ok了

接收数据并全部写入文件

```
from pwn import *
from hashlib import md5, sha256
from itertools import product
from string import digits, ascii_letters
from icecream import *

def getxxxx():
    r.recvuntil(b'xxxx+')
    buf = r.recvuntil(b') ')[:-2]
    s2 = buf
    r.recvuntil(b'= ')
    buf = r.recvuntil(b'\\n')[:-1]
    hashenc = (buf.decode())

    ic(s2, hashenc)
    tab = bytes(digits + ascii_letters, encoding='utf-8')
    for s1 in product(tab, repeat=4):
        data = bytes(s1) + s2
        hash_data = sha256(data).hexdigest()

        if hash_data == hashenc:
            ic(hash_data, hashenc)

            buf = r.recv(1024)
            print(buf)
            print(bytes(s1))
            r.sendline(bytes(s1))
            # buf = r.recv(1024)
            # print(buf)
            return bytes(s1)
        # input()

r = remote( '52.163.228.53', 8081)
getxxxx()
# r = remote('0.0.0.0', 10000)
f = open('./远程记录的密文.py', 'a')
f.seek(0)
f.truncate()

buf = r.recvuntil(b'\\ngive me a number:\\n')
print(buf)
n = int(buf[len('n: '):-len('\\ngive me a number:\\n')])
print(n)
f.write('n='+str(n)+'\\n')

r.sendlines('0')
ans = []
for i in range(125):
    buf = r.recvuntil(b'\\ngive me a number:\\n')
    # print(buf)
    t = int(buf[len('done: '):-len('\\ngive me a number:\\n')])
    print(t)
    ans.append(t)
    r.sendlines('0')
print(ans)

f.write('enc=[')
for i in ans:
    f.write(str(i)+',\\n')
f.write(']')
# input("按任意键继续")

```

sage爆破矩阵脚本

```
from 远程记录的密文 import n, enc

from Crypto.Util.number import *

def getall(l):
     x = l[:7]
     iv = l[7]
     tab = []
     for i in range(128):
         bin_arr = (bin(i)[2:].rjust(7, '0'))
     #     print(bin_arr)
         temp = 0
         for j in range(len(bin_arr)):
             temp += int(bin_arr[j])*x[j]
         temp += iv
         temp %= n
         # print(temp)
         # input()
         tab.append(temp)
     flag = ''
     for i in enc:
          # print(tab.index(i))
          flag += bin(tab.index(i))[2:].rjust(7, '0')
     flag = flag[:120]
     flag = (int(flag, 2))
     print(long_to_bytes(flag))

def get_x(c, m, _):
     # 矩阵行列化
     # x = [[i]for i in x]
     c = [[i] for i in c]
     m = m[::-1]

     c = Matrix(Zmod(n), c)
     m = Matrix(Zmod(n), m)
     x1 = (m.inverse()*c)
     # print(x1)
     l = []
     for i in x1:
          l.append(i[0])
     getall(l)

c = []
for i in range(8):
    j = i*17+2
    c.append(enc[j-1])

for _ in range(0x4000):
     key = bin(_)[2:].rjust(14, '0')
     # key = '00101010010000'
     m = []
     l = []
     for i in range(8):
          for j in range(i, i+7):
               # print(j,end=' ')
               l.append(int(key[j]))
          l.append(1)
          m.append(l)
          l = []
     # for i in m:
     #      print(i)
     # ============
     # get_x(c, m, _)
     try:
         get_x(c, m, _)
     except:
          continue
# *CTF{yOuG0t1T!}

```

# myCurve

二元爱德华兹曲线（Binary Edwards Curves）

> 反正🧓也没打算大二完全搞明白这种数学👴玩的玩意


sage

```python

from Crypto.Util.number import *

def To_Birational(point):
    x, y = point
    return (3*(x+y)) / (x*y + x + y),  3*(x / (x*y+x+y) + 2)

F=GF(2**100)
R.<x,y>=F[]
d1=F.fetch_int(1)
d2=F.fetch_int(1)
x,y=(698546134536218110797266045394L, 1234575357354908313123830206394L)
G=(F.fetch_int(x),F.fetch_int(y))
x, y = (403494114976379491717836688842L, 915160228101530700618267188624L)
P =(F.fetch_int(x),F.fetch_int(y))

G = To_Birational(G)
P = To_Birational(P)

E = EllipticCurve(GF(2**100), [1, 2, 0, 0, 3])
G = E(G)
P = E(P)
flag = G.discrete_log(P)
print(long_to_bytes(flag))

```

爱德华曲线运算在区块链中应用广泛

# little case

先用维纳解p的值，再尝试把$\phi(n)$分解，瞬间出来两个4500左右的值和一个很大的解不动的值，nctf2019有一个题有这个开方脚本，这个题 e 的大小甚至和nctf2019那个题是一样的。。。。

> 当时完全没有注意到去检查参数长度。。。太拉了。。。满脑子LLL