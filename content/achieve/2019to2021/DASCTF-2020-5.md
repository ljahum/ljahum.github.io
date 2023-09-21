---
title: 安恒五月赛-2020-re部分
date: 2020-05-26 01:16:19
tags: 
- bin
categories:
- CTF
---
> 我就是铁废物了，整场比赛只做出来两个题

## ViQinere

exp找不到了。。。orz
总之连接题目给的ip地址，会返回一串密文，分析一下算法就可以回去开开心心写爆破了

## BScript

因为队友吧win环境搞没了，所以我就先搞了两个pe文件，不得不说出题人真有你的啊。。。从0开始的文件操作了解一下

把加密flag的文件拆成800多分，分别放到800多个加了upx壳的文件里面

emmmm，好吧先脱壳

### 脱壳demo：

```cpp
#include <iostream>
#include <stdlib.h>
using namespace std;
int main()
{
    for(int i=0;i<804;i++)
    {

        char x[5];
        memset(x, 0, sizeof(x));
        itoa(i,x,10);

        char s[20] = "upx -d BScript/";
        strcat( s ,x);
        char s1[10] = ".exe";
        strcat(s, s1);
        system(s);
    }
    //system(s);
    system("pause");
}
```

脱完壳后随便打开几个到ida里面看看，发现不同的文件里面的变量ans或变量key里面放了一小段pe文件的信息

```cpp
 __main();
  puts("What a easy RE!");
  while ( i )
  {
    scanf("%d", &v4 + i);
    if ( *(&v4 + i) != ans[63 - i] )//ans里面倒序放置64个 byte 的信息
    {
      puts("Are you a fool?");
      exit(0);
    }
    --i;
```

然后在写几个demo对所有pe文件的类型分析一下，发现800个文件只有两种不同的大小，相同大小的文件中放pe文件的信息的位置是一样的

```cpp
    FILE *fp = NULL;
    fp = fopen("BScript\\1~804.exe" , "rb");
    fseek(fp,0,SEEK_END);
    int size = ftell(fp);//这样可以得到文大小
    cheak(size); 
文件大小  pe信息位置   长度
0xBE03   0x1C40       64 	逆序
0xBDF1   0x1c20       32 	正序
```

然后继续撸脚本把文件提取出来，先新建一个空的“new”文件在执行以下代码

#### demo:

```cpp
#include <iostream>
#include <fstream>
#include <cstring>
#include <vector>
#include <stdlib.h>
using namespace std;
int main()
{
    for (int i = 0; i < 804; i++)
    {
        char fname[20] = "BScript\\";
        char s[20];
        memset(s, 0, sizeof(s));
        itoa(i, s, 10);
        strcat(fname, s);
        strcat(fname, ".exe");
        char s2[100];
        memset(s2, 0, sizeof(s2));
        FILE *fr = NULL;
        fr = fopen(fname, "rb");
        fseek(fr,0,SEEK_END);
        int size = ftell(fr);
        //printf("%X",size);
        if (size == 48643)
        {
            fseek(fr, 0x1C40, SEEK_SET);
            fread(&s2, 1, 0x40, fr);
            fclose(fr);
            FILE *NewFile = NULL;
            NewFile = fopen("new", "ab");
            for (int i = 0x40 - 1; i >= 0; i--)
                fwrite(&s2[i], 1, 1, NewFile);
        }
        else if (size == 48625)
        {
            fseek(fr, 0x1C20, SEEK_SET);
            fread(&s2, 1, 0x20, fr);
            fclose(fr);
            FILE *NewFile = NULL;
            NewFile = fopen("new", "ab");
            for (int i = 0; i < 0x20; i++)
                fwrite(&s2[i], 1, 1, NewFile);
        }
    }
    system("pause");
}

```
> 原谅弟弟我只会用C语言写文件操作，写的还很丑

完工后得到 New 拖进ida发现并没有我们苦苦追寻的字符串。。。那就先看看前几个最有希望的函数，果然在前几个函数中发现了我们感兴趣的代码


在 0x004015C0 处的函数是个标准的base64，交叉引用一下可以小刀输入的位置和判断的位置，进行找一下可以在 0x00401BEE处的函数了找到给密文赋值的语句



稍微注意一下赋值的地址就可以把密文拼出来了
```cpp
0x00401AD7处
int __cdecl fun(int a1)
{
  tab = 'Q';
  byte_40D041 = 'k';
  byte_40D042 = 'p';
  byte_40D043 = 'E';
  byte_40D044 = 'e';
  byte_40D045 = '1';
  byte_40D046 = 'd';
  byte_40D047 = 'o';
  byte_40D048 = 'T';
  byte_40D049 = '3';
  byte_40D04A = 'R';
  byte_40D04B = 'f';
  byte_40D04C = 'N';
  byte_40D04D = 'F';
  byte_40D04E = '9';
  byte_40D04F = 'i';
  byte_40D050 = 'Y';
  byte_40D051 = 'W';
  byte_40D052 = 'V';
  byte_40D053 = '1';
  byte_40D054 = 'd';
  v1 = sub_401E7C('2');
  v2 = v1;
  v3 = (_BYTE *)v1;
  for ( i = 49; i >= 0; --i )
    *v3++ = 0;
  v7 = v2;
  v5 = sub_401ECC(a1, (int)v3);
  unk_40E2C0(v7, 41, v5);
  for ( j = 0; ((int (__cdecl *)(int))dword_407C68[0])(v7) > j; ++j )
    ++*(_BYTE *)(j + v7);
  return v7;
}

0x004019FF处
int __usercall fun2@<eax>(int a1@<edx>, int a2, int a3)
{

  f((int)&v4, a1);
  f_0(a2, a3, (int)&v4);
  f_1((int)&v4);
  byte_40D055 = 68;
  byte_40D056 = 70;
  byte_40D057 = 109;
  byte_40D058 = 100;
  byte_40D059 = 84;
  byte_40D05A = 70;
  byte_40D05B = 102;
  byte_40D05C = 99;
  byte_40D05D = 50;
  byte_40D05E = 78;
  byte_40D05F = 121;
  byte_40D060 = 98;
  byte_40D061 = 72;
  byte_40D062 = 66;
  byte_40D063 = 48;
  byte_40D064 = 102;
  byte_40D065 = 81;
  byte_40D066 = 65;
  byte_40D067 = 65;
  return a2;
}
```

然后就可以撸exp了
不要忘了按出题人的意思把flag转成32位小写md5值

### exp:

```cpp
import base64
import hashlib
l = ['Q', 'k', 'p', 'E', 'e', '1', 'd', 'o', 'T', '3', 'R', 'f', 'N', 'F', '9', 'i', 'Y', 'W', 'V', '1','d', 'D', 'F', 'm', 'd', 'T', 'F', 'f', 'c', '2', 'N', 'y', 'b', 'H', 'B', '0', 'f', 'Q', 'A', 'A']
s = b'QkpEe1doT3RfNF9iYWV1dDFmdTFfc2NybHB0fQAA'
s1 = base64.b64decode(s)
print(s1)
flag = b"BJD{WhOt_4_baeut1fu1_scrlpt}"
#"BJD{WhOt_4_baeut1fu1_scrlpt}"

f = hashlib.md5()
f.update(flag)
print(f.hexdigest())


#e801bcbcc42d3120d910ccc46ae640dd
```
如果上交buu的话应该是
flag{e801bcbcc42d3120d910ccc46ae640dd}

> 以下为复现题目

## Blink:

> 这个题做得可以说是相当没有体验了，看到一堆闪瞎狗眼的东西后立马就滚回去补作业了

观赏了表哥们的wp后我才发现那堆x才是二维码的本体（我一直以为色块是二维码的组成部分）

中间产生随机数来确定要不要打印x，所以把随机数判断部分patch掉就好了


为防止手贱前面的屏幕刷新patch掉


然后运行就会得到这种东西


复制出来扫一下


`BJD{TW1NKLE_TW1NKLE_L1TTLE_5TAR}

**宁是来出misc的吧**


## log1cal 

拖进ida动调一下先判断格式和长度

`BJD{64个字符}`

然后后面的位移操作没什么难的，只有重点在加密上面
### 头皮发麻的加密：

```cpp
for ( i = 0; i <= 63; ++i )
  {
    *flag = flag[1] & ~(((*flag << 28) & ~((*flag >> 36) & (*flag << 28)) | ~((*flag >> 36) & (*flag << 28)) & (*flag >> 36)) & flag[1]) | ~(((*flag << 28) & ~((*flag >> 36) & (*flag << 28)) | ~((*flag >> 36) & (*flag << 28)) & (*flag >> 36)) & flag[1]) & ((*flag << 28) & ~((*flag >> 36) & (*flag << 28)) | ~((*flag >> 36) & (*flag << 28)) & (*flag >> 36));
    flag[1] = flag[2] & ~(((flag[1] << 22) & ~((flag[1] >> 42) & (flag[1] << 22)) | ~((flag[1] >> 42) & (flag[1] << 22)) & (flag[1] >> 42)) & flag[2]) | ~(((flag[1] << 22) & ~((flag[1] >> 42) & (flag[1] << 22)) | ~((flag[1] >> 42) & (flag[1] << 22)) & (flag[1] >> 42)) & flag[2]) & ((flag[1] << 22) & ~((flag[1] >> 42) & (flag[1] << 22)) | ~((flag[1] >> 42) & (flag[1] << 22)) & (flag[1] >> 42));
    flag[2] = flag[3] & ~(((flag[2] << 16) & ~((flag[2] >> 48) & (flag[2] << 16)) | ~((flag[2] >> 48) & (flag[2] << 16)) & (flag[2] >> 48)) & flag[3]) | ~(((flag[2] << 16) & ~((flag[2] >> 48) & (flag[2] << 16)) | ~((flag[2] >> 48) & (flag[2] << 16)) & (flag[2] >> 48)) & flag[3]) & ((flag[2] << 16) & ~((flag[2] >> 48) & (flag[2] << 16)) | ~((flag[2] >> 48) & (flag[2] << 16)) & (flag[2] >> 48));
    flag[3] = flag[4] & ~(((flag[3] << 58) & ~((flag[3] >> 6) & (flag[3] << 58)) | ~((flag[3] >> 6) & (flag[3] << 58)) & (flag[3] >> 6)) & flag[4]) | ~(((flag[3] << 58) & ~((flag[3] >> 6) & (flag[3] << 58)) | ~((flag[3] >> 6) & (flag[3] << 58)) & (flag[3] >> 6)) & flag[4]) & ((flag[3] << 58) & ~((flag[3] >> 6) & (flag[3] << 58)) | ~((flag[3] >> 6) & (flag[3] << 58)) & (flag[3] >> 6));
    flag[4] = flag[5] & ~(((flag[4] << 52) & ~((flag[4] >> 12) & (flag[4] << 52)) | ~((flag[4] >> 12) & (flag[4] << 52)) & (flag[4] >> 12)) & flag[5]) | ~(((flag[4] << 52) & ~((flag[4] >> 12) & (flag[4] << 52)) | ~((flag[4] >> 12) & (flag[4] << 52)) & (flag[4] >> 12)) & flag[5]) & ((flag[4] << 52) & ~((flag[4] >> 12) & (flag[4] << 52)) | ~((flag[4] >> 12) & (flag[4] << 52)) & (flag[4] >> 12));
    flag[5] = flag[6] & ~(((flag[5] << 46) & ~((flag[5] >> 18) & (flag[5] << 46)) | ~((flag[5] >> 18) & (flag[5] << 46)) & (flag[5] >> 18)) & flag[6]) | ~(((flag[5] << 46) & ~((flag[5] >> 18) & (flag[5] << 46)) | ~((flag[5] >> 18) & (flag[5] << 46)) & (flag[5] >> 18)) & flag[6]) & ((flag[5] << 46) & ~((flag[5] >> 18) & (flag[5] << 46)) | ~((flag[5] >> 18) & (flag[5] << 46)) & (flag[5] >> 18));
    flag[6] = flag[7] & ~(((flag[6] << 40) & ~((flag[6] >> 24) & (flag[6] << 40)) | ~((flag[6] >> 24) & (flag[6] << 40)) & (flag[6] >> 24)) & flag[7]) | ~(((flag[6] << 40) & ~((flag[6] >> 24) & (flag[6] << 40)) | ~((flag[6] >> 24) & (flag[6] << 40)) & (flag[6] >> 24)) & flag[7]) & ((flag[6] << 40) & ~((flag[6] >> 24) & (flag[6] << 40)) | ~((flag[6] >> 24) & (flag[6] << 40)) & (flag[6] >> 24));
    flag[7] = *flag & ~(((flag[7] << 34) & ~((flag[7] >> 30) & (flag[7] << 34)) | ~((flag[7] >> 30) & (flag[7] << 34)) & (flag[7] >> 30)) & *flag) | ~(((flag[7] << 34) & ~((flag[7] >> 30) & (flag[7] << 34)) | ~((flag[7] >> 30) & (flag[7] << 34)) & (flag[7] >> 30)) & *flag) & ((flag[7] << 34) & ~((flag[7] >> 30) & (flag[7] << 34)) | ~((flag[7] >> 30) & (flag[7] << 34)) & (flag[7] >> 30));
    v6 = 1;
  }
```
但是我们把每一个算式拆成下面的形式：
```cpp
a = flag<<n1
b = flag>>n2

*flag =
flag[1] 
& 
~((a & ~(b & a) | ~(b & a) & b)
& flag[1]) 
|
 ~((a & ~(b & a) | ~(b & a) & b) 
& flag[1])
& 
(a & ~(b & a) | ~(b & a) & b);
```
可以发现`(a & ~(b & a) | ~(b & a) & b)`是作为一个整体出现的，得到的结果就是 a^b 的值
化简后

`*flag = flag[1] & ~((a^b)& flag[1]) | ~((a^b) & flag[1]) & (a^b);`

又发现整个算式就是刚才的那个结构，于是把加密进行简化为

`flag[0]=(a^b)^flag[1]`

再加之a和b的两个位移指正好是64，这样就可以开始逆了

### Demo:

```python
import hashlib
l = [0x08CD53D0EAE56FDE,0xE0310C8244BA1FA3,0x45B42002CE1B213D,0x16FDC411224CB2DF,0x2FD8108A59461BCC,0x8F6990725EB01982,0x9BA5ADE29A2A17D8,0x4DEAA99F5D9F6605]
n1 = []
n2 = []
for i in range(64):
    l[7] = ((l[7] ^ l[0]) >> 34) & 0xffffffffffffffff ^ (((l[7] ^ l[0])) << 30) & 0xffffffffffffffff
    l[6] = ((l[6] ^ l[7]) >> 40) & 0xffffffffffffffff ^ (((l[6] ^ l[7])) << 24) & 0xffffffffffffffff
    l[5] = ((l[5] ^ l[6]) >> 46) & 0xffffffffffffffff ^ (((l[5] ^ l[6])) << 18) & 0xffffffffffffffff
    l[4] = ((l[4] ^ l[5]) >> 52) & 0xffffffffffffffff ^ (((l[4] ^ l[5])) << 12) & 0xffffffffffffffff
    l[3] = ((l[3] ^ l[4]) >> 58) & 0xffffffffffffffff ^ (((l[3] ^ l[4])) << 6) & 0xffffffffffffffff
    l[2] = ((l[2] ^ l[3]) >> 16) & 0xffffffffffffffff ^ (((l[2] ^ l[3])) << 48) & 0xffffffffffffffff
    l[1] = ((l[1] ^ l[2]) >> 22) & 0xffffffffffffffff ^ (((l[1] ^ l[2])) << 42) & 0xffffffffffffffff
    l[0] = ((l[0] ^ l[1]) >> 28) & 0xffffffffffffffff ^ (((l[0] ^ l[1])) << 36) & 0xffffffffffffffff

s=''
for i in l:
    t = i
    while t>0:
        s+=chr(t%0x100)
        t = t//0x100
flag =''

s1 = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'
s2 = '456789+/YZabcdefopqrstuvwxyz0123ghijklmnABCDEFGHQRSTUVWXIJKLMNOP'
for i in s1:
    flag += s[s2.find(i)]
print(flag)
# easy_logical_algorithm_for_freshman_and_try_to_slove_it_yourself

m=hashlib.md5()
m.update(bytes(flag,encoding="utf-8"))
print(m.hexdigest().upper())
#A6FFB35FEF107F2A0DAAE19BCD7B2297

```

其实在看到加密部分只有与、或、非的时候就应该往与、或、非实现异或的方向想了，因为没有加减乘除的参与，单纯的与、或、非很难把flag逆回来

## Py2

pyc文件很完整,可以直接用工具反出来,拿到源码是这样的

### demo：
```py
#! /usr/bin/env python 2.7 (62211)
#coding=utf-8
# Compiled at: 2020-04-23 03:22:50
#Powered by BugScaner
#http://tools.bugscaner.com/
#如果觉得不错,请分享给你朋友使用吧!

import ctypes
from base64 import b64encode, b64decode

def decode():
    fd = open('./libc.so', 'rb')
    data = fd.read()
    fd.close()
    print(123)
    fd = open('./libc.so', 'wb')
    fd.write(b64decode(data))
    fd.close()
def check():
    if b64encode(pwd) == 'YmpkMw==':  # bjd3
        decode()
        dl = ctypes.cdll.LoadLibrary
        lib = dl('./libc.so')
        reply = lib.check
        reply(int(flag[:length // 2], 16), int(flag[length // 2:], 16), int(pwd.encode('hex'), 16))
        print 'your input is BJD' flag.decode('hex')
    else:
        print 'your password is wrong!'


if __name__ == '__main__':
    print 'Please input your flag:'
    flag = raw_input()

    flag = flag.encode('hex')

    length = len(flag)
    print 'Please input your password:'
    pwd = raw_input()
    check()
    #decode()
```

分析一下流程可以发现代码调用了 libc.so 动态链接库里面的cheak函数进行加密

py源码好就好在可以对流程随意修改,我们让程序对文件解base64后得到 libc.so里面的cheak函数是长这样的

### cheak函数
```cpp
  v3 = a1;
  v7 = a2;
  v6 = a2;
  v5 = a2;
  v4 = a2;
  code((unsigned __int64 *)&v3, &v4);
  if ( v3 == __PAIR128__(0xD760262509C2F6D0LL, 0xAF9D869B6947017DLL) )
    puts("you win!");
  else
    puts("you failed!");
```
### code函数:
```cpp
  v4 = *a1;
  v5 = a1[1];
  sum = 0LL;
  v7 = 32LL;
  while ( 1 )
  {
    v2 = v7--;
    if ( !v2 )
      break;
    sum += 0x9E3779B9LL;
    v4 += (v5 + sum) ^ (16 * v5 + *key) ^ ((v5 >> 5) + key[1]);
    v5 += (v4 + sum) ^ (16 * v4 + key[2]) ^ ((v4 >> 5) + key[3]);
  }
  *a1 = v4;
```
code函数内是标准的tea加密,早知道加密方式这么简单我就先来做py2了。。。

但是反出来的这行代码又很奇怪`if ( v3 == __PAIR128__(0xD760262509C2F6D0LL, 0xAF9D869B6947017DLL) )`于是干脆看汇编确定判断时的细节


可以确定

0xAF9D869B6947017D,0xD760262509C2F6D0

对应第1、2个flag

tea的解密脚本很好写，但是要注意题目用的是64位int

### exp:
```cpp
#include <stdint.h>
#include <stdio.h>
#include <iostream>
#include <iomanip>
using namespace std;
void decrypt(unsigned __int64 *v, unsigned __int64 *k)
{
    unsigned __int64 v4 = v[0], v5 = v[1], v2 = v[2], v3 = v[3], sum = 0;
    unsigned __int64 k0 = k[0], k1 = k[1], k2 = k[2], k3 = k[3];
    for (int i = 0; i < 32; i++)
    {
        sum += 0x9E3779B9;
    }
    for (int i = 0; i < 32; i++)
    {
        v5 -= ((v4 * 16) + k2) ^ (v4 + sum) ^ ((v4 >> 5) + k3);
        v4 -= ((v5 * 16) + k0) ^ (v5 + sum) ^ ((v5 >> 5) + k1);
        sum -= 0x9E3779B9;
    }
    v[0] = v4;
    v[1] = v5;
}
int main()
{
    unsigned __int64 flag[2] = { 0xAF9D869B6947017D,0xD760262509C2F6D0};
    unsigned __int64 key[4] = {0x626a6433, 0x626a6433, 0x626a6433, 0x626a6433};
    decrypt(flag, key);
    for (auto &&i : flag)
    {
        cout << hex << i<< endl;
    }
    system("pause");
    return 0;
}
//676f745f
//74656121
```
把结果转成字符串

> got_tea!

> 这几个re除了两个misc确实难度感觉都比较适中。。。应该把所有题都看一遍的


