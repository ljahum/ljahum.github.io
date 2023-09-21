---
title: openssl/pem
date: 2021-07-27 15:33:01+08:00
author: ljahum

tags:
- codes
- crypto

categories:
- posts

hiddenFromHomePage: false
math:
  enable: true

---
<!--more-->


# openssl基本使用指北

> 鉴于网上能找到的都写得像shit一样
>
> 求人不如求己了属于是

## rsa

rsa pem密钥文件有 $n,p,q,dp,dq,e,d,[p^{-1}]_q$

  所有参数

  私钥

  ```
  openssl rsa -in ./akey.pem -text
  ```

  公钥

  ```
  openssl rsa -in ./akey.pem -text -pubin
  ```

  生成

  ```
  openssl genrsa > key.pem
  openssl genrsa -out privkey.pem 2048
  ```

## ecc

ec 密钥文件有参数

- 曲线：$a,b,p,生成元G，阶数N$
- 密钥：$私钥d，公钥P=d*G$

  提取

  ```
  openssl ec -in ./p384-key.pem -text
  ```

## 提取csr证书

`openssl req -new -key privkey.pem -out ca.csr`

## 提取密钥信息

### 公私钥模数

`openssl rsa -in .\pubkey.pem -pubout -modulus`

`openssl rsa -in .\pubkey.pem -pubin -modulus`

### 提取所有信息

`openssl asn1parse -i -in privkey.pem`

公钥要指定偏移查看，bit string的偏移是19

![](https://i.loli.net/2020/10/12/eCbmTJQY4NkuAv3.png)

`openssl asn1parse -i -in .\pubkey.pem -strparse 19`



## pem、der格式转化

`openssl rsa -in .\private.pem -outform der  -out .\private.der`

# 除了生孩子什么都能干的python

crypto yyds

`from Crypto.PublicKey import RSA`



## 读取公钥信息
```python

rsakey = RSA.importKey(open("public.key", "r").read())
n = rsakey.n
e = rsakey.e
print("n=%\ne=%d",n,e)

```
## 生成秘钥对文件
```python

rsa = RSA.generate(2048)
public_key = rsa.publickey().exportKey()
f = open("public_key.key", "w")
f.write(public_key.decode())
f.close()
private_key = rsa.exportKey()

f = open("private_key.key", "w")
f.write(private_key.decode())
f.close()
```

## 对文件加解密

载入填充方式

`from Crypto.Cipher import PKCS1_OAEP`

```python
rsakey=RSA.importKey(open("public_key.key","r").read())
rsa = PKCS1_OAEP.new(rsakey)
encrypt = rsa.encrypt(flag.encode())

rsakey=RSA.importKey(open("private_key.key","r").read())
rsa = PKCS1_OAEP.new(rsakey)
decrypt = rsa.decrypt(f.read())
```

## 生成 private_key.key

```python
rsa_components = (n, e, int(d), p, q)
rsa = RSA.construct(rsa_components)
public_key = rsa.exportKey()
# 此rsa与上文随机生成的rsa相同
f = open("public_key.key", "w")
f.write(public_key.decode())
f.close()
```