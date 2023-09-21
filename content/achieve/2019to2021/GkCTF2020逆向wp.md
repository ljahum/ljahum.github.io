---
title: GkCTF2020逆向wp 
date: 2020-06-16 16:20:44
tags: 
- bin
categories:
- CTF
---
> 发现GKCTF在buu上面挂了好久，干脆做着玩玩好了
# Check_1n

打开一看，是一个 真-虚拟机，vm看了直呼内行



直接找字符串 


解出来是这样的
> Why don't you try the magic brick game

然后又找到一串很像密文的字符串



看一下调用了他的函数
## decode:
```cpp
for ( i = 0; i < str; ++i )
  {
    v3 = byte_175E270[(unsigned __int8)a1[i]];
    if ( v3 < 0 )
    {
      sub_406B70(v8);
      return 0;
    }
    for ( j = &v8[v9 - 1]; j >= str2; --j )
    {
      v2 = 58 * (unsigned __int8)*j + v3;
      *j = v2 % 0x100;
      v3 = BYTE1(v2);
    }
    if ( v3 > 0 )
    {
      if ( --str2 < v8 )
      {
        sub_406B70(v8);
        return 0;
      }
      *str2 = v3;
    }
  }
  for ( k = 0; k < &v8[v9] - str2; ++k )
    v8[k] = str2[k];
  v8[k] = 0;
  return v8;
```
是base58的算法，直接在线解就可以了

> flag{f5dfd0f5-0343-4642-8f28-9adbb74c4ede}

# babyDriver
0x140001380里面是一个迷宫
## code:
```cpp
__int64 __fastcall map_0(__int64 a1, __int64 str_2)
{
  __int64 v2; // rbx
  __int64 str_1; // rdi
  __int64 v4; // rax
  int local; // ecx
  __int16 *str; // rsi
  __int64 Sbox; // rbp
  __int16 s; // dx
  char siep; // dl
  CHAR *v10; // rcx

  v2 = str_2;
  if ( *(_DWORD *)(str_2 + 48) >= 0 )
  {
    str_1 = *(_QWORD *)(str_2 + 24);
    v4 = *(_QWORD *)(str_2 + 56) >> 3;
    if ( (_DWORD)v4 )
    {
      local = value1;
      str = (__int16 *)(str_1 + 2);
      Sbox = (unsigned int)v4;
      while ( *(_WORD *)(str_1 + 4) )
      {
LABEL_28:
        str += 6;
        if ( !--Sbox )
          goto LABEL_29;
      }
      map[local] = 46;
      s = *str;
      if ( *str == '\x17' )
      {
        if ( local & 0xFFFFFFF0 )
        {
          local -= 0x10;
          goto LABEL_21;
        }
        local += 0xD0;
        value1 = local;
      }
      if ( s == '%' )
      {
        if ( (local & 0xFFFFFFF0) != 0xD0 )
        {
          local += 16;
          goto LABEL_21;
        }
        local -= 0xD0;
        value1 = local;
      }
      if ( s == '$' )
      {
        if ( local & 0xF )
        {
          --local;
          goto LABEL_21;
        }
        local += 15;
        value1 = local;
      }
      if ( s != '&' )
        goto LABEL_22;
      if ( (local & 0xF) == 15 )
        local -= 15;
      else
        ++local;
LABEL_21:
      value1 = local;
LABEL_22:
      siep = map[local];
      if ( siep == '*' )
      {
        v10 = "failed!\n";
      }
      else
      {
        if ( siep != '#' )
        {
LABEL_27:
          map[local] = 'o';
          goto LABEL_28;
        }
        v10 = "success! flag is flag{md5(input)}\n";
      }
      value1 = 16;
      DbgPrint(v10);
      local = value1;
      goto LABEL_27;
    }
  }
LABEL_29:
  if ( *(_BYTE *)(v2 + 65) )
    *(_BYTE *)(*(_QWORD *)(v2 + 184) + 3i64) |= 1u;
  return *(unsigned int *)(v2 + 48);
}
```
乍一看 local 的变化有些地方毫无章法,再仔细一看发现是用来判断是否到底的,如果到边界就从另一边出来

## map:
```c
 o X   X X   X X X X X X   X X  
   X     X X X     X   X   X    
   X         X     X   X   X    
   X X X     X X X X   X   X    
       X X       X     X   X X  
   X     X       X     X     X  
   X     X             X     X  
   X     X X X X       X     X  
   X           X       X X X X  
   X X X       X                
     X X       X X X X X X #    
     X
```
因为是驱动文件所以字符是以键盘扫描码的形式输入的

对应表：
https://blog.csdn.net/wenweimin/article/details/105561                          

注意键盘扫描码全是大写

把迷宫走出来转md5

>LKKKLLKLKKKLLLKKKLLLLLL
> flag{403950a6f64f7fc4b655dea696997851}

# Chelly's identity

这个题还比较有意思,可惜还是单字节加密

直接动调到加密位置

## demo

```cpp

_DWORD *flag;//注意flag变成int了

 while ( flag != (_DWORD *)v6 )
  {
    v5 = 0;
    v4 = 0;
    for ( i = (_DWORD *)sub_3B1325(&v8, 0); *i < *flag; i = (_DWORD *)sub_3B1325(&v8, v4) )
      v5 += *(_DWORD *)sub_3B1325(&v8, v4++);
    *flag ^= v5;
    ++flag;
  }
```

函数sub_3B1325里面的东西很不知所云,直接看汇编算了



发现这里从一个素数表取数和flag的值做对比，异或值是素数累加的值，分析得出算法：


## dome

```python
l=[2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97, 101, 103, 107, 109, 113, 127]
# l是素数表
s1=[]#前n个素数和
for i in range(len(l)):
	ans+=l[i]
	print(ans,i,l[i])
	s1.append(ans)

s="Che11y_1s_EG0IST"
s2=[]
for i in s:#是是flag
	for j in range(len(l)):
		if l[j] >= ord(i):
			s2.append((ord(i)^(s1[j-1])))
```

爆破就行了：
## exp
```python
encode=[438,1176,1089,377,377,1600,924,377,1610,924,637,639,376,566,836,830]
s="!\"#$%&\'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\\]^_`abcdefghijklmnopqrstuvwxyz{|}"
tab =[129, 130, 131, 132, 133, 227, 226, 237, 236, 196, 197, 309, 308, 311, 310, 376, 377, 378, 379, 380, 381, 331, 330, 325, 324, 327, 326, 388, 389, 459, 458, 437, 436, 439, 438, 636, 637, 638, 639, 567, 566, 642, 643, 644, 645, 646, 647, 839, 838, 837, 836, 830, 831, 828, 829, 818, 819, 921, 920, 927, 926, 925, 924, 931, 930, 1094, 1095, 1088, 1089, 1263, 1262, 1176, 1177, 1178, 1179, 1335, 1334, 1446, 1447, 1464, 1465, 1611, 1610, 1613, 1612, 1615, 1614, 1601, 1600, 1603, 1602, 1605, 1604]
# tab是s的密文
for i in encode:
	print(s[tab.index(i)],end='')

```
> Che11y_1s_EG0IST

# EzMachine

虚拟机，先找vm的部分：



这个大循环基本就是vm的主体了,通过改变这个 call 调用的函数来传递参数和对flag进行计算

经过漫长的调试发现在取从flag中取字符串的函数后会调用好几个用减法来进行对比的函数,发现程序用这种方式来判断字符串的范围

```cpp
.text:00E81300 cheak proc near                         ; DATA XREF: .data:00EC4954↓o
.text:00E81300 mov     edx, time
.text:00E81306 movzx   eax, byte ptr unk_EC49A2[edx]
.text:00E8130D mov     ecx, ds:off_EC27FC[eax*4]
.text:00E81314 movzx   eax, byte_EC49A1[edx]
.text:00E8131B add     edx, 3
.text:00E8131E mov     time, edx
.text:00E81324 mov     eax, ds:off_EC27FC[eax*4]
.text:00E8132B mov     eax, [eax]
.text:00E8132D sub     eax, [ecx]  <------ flag值 减去 对比值 通过结果的大小来判断值的范围
.text:00E8132F mov     cheak_return, eax
.text:00E81334 retn
.text:00E81334 cheak endp
```
之后几个计算的函数有无又计算符的关系还比较方便分析,最后可以搞定加密的代码,再观察最后对比的方式可以找到传参方式

```python
cipher=[]
or i in s1:
	print(i,end=' ')
	if "abcdefghijklmnopqrstuvwxyz".find(i) != -1:
		cipher.append(((ord(i)^0x47)+1)//0x10)
        cipher.append(((ord(i)^0x47)+1)%0x10)
	if "ABCDEFGHIJKLMNOPQRSTUVWXYZ".find(i) != -1:
		cipher.append(((ord(i)^0x4b)-1//0x100))
		cipher.append(((ord(i)^0x4b)-1%0x100))
	elif '_{}'.find(i) !=-1:
		cipher.append(ord(i)//100)
        cipher.append(ord(i)%100)
cipher=cipher[::-1]#最后对比的时候在内存里面的值有点魔幻,但基本能弄出来

```
又是单字节加密,把所有科显示字符弄出来一一对应再去掉题目不要求的字符就可以了

```python
s1='!\"#$%&\'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\\]^_`abcdefghijklmnopqrstuvwxyz{|}'
tab =[33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 9, 8, 7, 14, 13, 12, 11, 2, 1, 0, -1, 6, 5, 4, 3, 26, 25, 24, 23, 30, 29, 28, 27, 18, 17, 16, 91, 92, 93, 94, 95, 96, 39, 38, 37, 36, 35, 34, 33, 48, 47, 46, 45, 44, 43, 42, 41, 56, 55, 54, 53, 52, 51, 50, 49, 64, 63, 62, 123, 124, 125]
ans=[0x22,0x2c,0x27,0x21,0x7b,0x17,0x33,0x25,0x30,0x5f,0x9,0x5f,0xd,0x10,0x1c,0x5,0x7d]
for i in ans: 
	for j in range(len(tab)):
		if tab[j] == i:
			print(s1[j],end='')
```
然后把题目不要求的字符去掉
> f,l'a!g{S3u%c0h_A_EZVM}
> flag{Such_A_EZVM}

# DbgIsFun

> 晚上闲着没事就干脆做了，明天把剩下的都做了吧，找不到wp反而更刺激了

开局就看到TLS,反调试预定，调试后发现中间的异或是再解smc，众所周知单层smc和没有是一样的（x）



下断点看看 sub_4010F0 里面

## demo:

```cpp
while ( !dword_41A8E0 )
    Sleep(0x3E8u);
  for ( i = 0; i < 140; ++i )
    byte_41A8DE += *((_BYTE *)&loc_401540 + i);
  for ( i = 0; flag[i]; ++i )
    flag[i] ^= byte_41A8DE;
  v10 = 71;
  v11 = 75;
  v12 = 67;
  v13 = 84;
  v14 = 70;
  for ( j = 0; j < 256; ++j )
    list[j] = j;
  for ( j = 0; j < 256; ++j )
    v8[j] = *((_BYTE *)&v10 + 4 * (j % 5));
  v20 = 0;
  for ( j = 0; j < 256; ++j )
  {
    v20 = (v8[j] + list[j] + v20) % 256;
    v17 = list[j];
    list[j] = list[v20];
    list[v20] = v17;
  }
  v18 = len;
  v16 = 0;
  v20 = 0;
  j = 0;
  while ( 1 )
  {
    v4 = v18--;
    if ( !v4 )
      break;
    j = (j + 1) % 256;
    v20 = (list[j] + v20) % 256;
    v17 = list[j];
    list[j] = list[v20];
    list[v20] = v17;
    v15 = (list[v20] + list[j]) % 256;
    Sbox[v16++] = list[v15];
  }
  for ( j = 0; j < len; ++j )
    v23[j] = LOBYTE(Sbox[j]) ^ flag[j];
  v22 = v23;
  if ( j != len && j == len )
    JUMPOUT(0xD04FE8DB);
  v5 = 0;
  while ( v5 < len )
  {
    if ( v22[v5] != *(_BYTE *)(v5 + 4199594) )
    {
      sub_401640(4199659, v25);
      sub_405183(0);
      break;
    }
    if ( ++v5 )
    {
      if ( !v5 )
        JUMPOUT(0x6EAF8766u);
    }
  }
  sub_401640("right\n", v24);
  sub_403AFA();
  return 0;
}
```

十分明显的RC4，rc4之前将flag每个字符的值异或上了0xc9

题目到这里已经做完了，到0x4014AA里面去拿数据直接rc4就完事了

但是注意，动调时和非动调时内存0x4014AA里面的值因为smc解码的原因，所以是不一样的




然后继续动调可以发现前面的rc4函数是可以进去的，但是因为反调试，flag的输入被跳过了，但是这并不影响后续流程

这里Sleep的计时器因为没有pdb文件所以会出错,改dword_41A8E0的值跳过

> 出现在数据段的神必字符串：
> X:\I_am_afraid_there_is_no_PDB_file_for_you_my_friend:D

```cpp
while ( !dword_41A8E0 )
    Sleep(0x3E8u);
```

因为没有 flag 输入所以 len 是0，这里是可以自己算的，但作为一条懒狗完全体，能让程序做的事情我们坚决不做

## braekpoint

```cpp
  v18 = len; <---改大一点
  v16 = 0;
  v20 = 0;
  j = 0;
  while ( 1 )
  {
    v4 = v18--;
    if ( !v4 )
      break;
    j = (j + 1) % 256;
    v20 = (list[j] + v20) % 256;
    v17 = list[j];
    list[j] = list[v20];
    list[v20] = v17;
    v15 = (list[v20] + list[j]) % 256;
    Sbox[v16++] = list[v15];
  }
  for ( j = 0; j < len; ++j )  <---下断点拿数据
    v23[j] = LOBYTE(Sbox[j]) ^ flag[j];
```
## exp
```python
encode=[0x2d,0xd4,0xf,0xd0,0x54,0xee,0x75,0xd0,0xe0,0x30,0x96,0xe1,0x79,0x8a,0xe0,0xfe,0x18,0x3a,0x27,0xe7,0x2f,0x86,0xc9,0xfe,0x66,0x43,0xa7,0x75,0x33,0xdb,0x8b,0xd,0xe4,0xa8,0x41,0x0,0x3b,0xd9,0x7d,0x38,0x8b,0x85,0xf8,0xff,0xfe,0xff,0x8a,0x4,0x18,0x8a,0x93,0xaa,0x14,0x40,0x0,0x3a,0xc2,0x75,0x14,0x43,0x74,0xe0,0x75,0xde]

key = [0x82,0x71,0xa7,0x7e,0xe6,0x12,0xc8,0x78,0x50,0xcd,0x28,0x49,0xc9,0x5,0x5b,0x7,0xbc,0xcb,0x9c,0x4b,0x87,0x24,0x70,0x7,0xc6,0xe4,0x1a,0xc1,0xe5,0x1b,0x13,0x3c,0x87,0x85,0xde,0xa4,0x77,0xae,0xdb,0x9b,0xb7,0x40,0x11,0x4a,0xc3,0xc,0x1b]
print('')
for i in range(28):
    print(chr((key[i]^encode[i])^0xc9),end='')
```

> flag{5tay4wayFr0m8reakp0int}