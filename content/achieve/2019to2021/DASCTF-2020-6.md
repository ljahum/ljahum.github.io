---
title: 安恒六月赛-2020-re部分
date: 2020-06-27 12:03:36
tags: 
- bin
categories:
- CTF
---
> 端午节回山里玩了一趟，做题时间仅有半天，做了的依旧只有re。。。。

# T0p_Gear

> 签到题，但做法不是很签到

三个判断输入三段字符串最后拿到完整flag

经过动调发现fun1函数是拿来比较的，在fun1入口把字符串copy下来就行了
## flag1 c92bb6a5
flag1密文校验
```cpp
_BOOL8 __usercall chk1@<rax>(YouWouldChk *this@<rdi>, __int64 a2@<rbp>, __m256i a3@<ymm7>)
{
  __int128 v4; // [rsp-28h] [rbp-28h]
  unsigned __int64 v5; // [rsp-10h] [rbp-10h]
  __int64 v6; // [rsp-8h] [rbp-8h]

  __asm { endbr64 }
  v6 = a2;
  v5 = __readfsqword(0x28u);
  v4 = (unsigned __int64)'5a6bb29c';
  std::operator<<<std::char_traits<char>>(&std::cout, "I drew a :", 0LL);
  std::operator>><char,std::char_traits<char>>(&std::cin, YouWouldChk::chk1(void)::JC);
  return (unsigned int)fun1((unsigned __int64)YouWouldChk::chk1(void)::JC, (__int64)&v4, a3) == 0;
}
```
## flag2 a6c30091
flag2要从附件中读取秘钥来解码字符串，在fun1处copy下来就行了
```cpp
  __asm { endbr64 }
  key = a3;                                     // 神必字符串
  v25 = __readfsqword(0x28u);
  v17 = 'txt.tt';
  v18 = 0LL;
  v19 = 0LL;
  v20 = 0LL;
  v21 = 0LL;
  v22 = 0LL;
  v23 = 0LL;
  v24 = 0LL;
  v6 = YouWouldChk::readStrFromFile(this, (char *)&v17, (char *)&v12, (__int64)&key);
  YouWouldChk::deAes(this, (char *)&v12, a2, v6, (__int64)&key, a5, a6);//aes解密
  v13 = 0LL;
  v14 = 0LL;
  v15 = 0LL;
  v16 = 0LL;
  sub_4010D0((unsigned __int64)&v12, (signed __int64)&v13, (unsigned __int64)&v12, a5, a6);
  std::operator<<<std::char_traits<char>>(&std::cout, "Richard steals some:", v7);
  std::operator>><char,std::char_traits<char>>(&std::cin, YouWouldChk::deAesFileAndMore(char *)::RH);
  i_1 = 0;
  for ( i = 0; *((_BYTE *)&v12 + i); ++i )
  {
    if ( *((_BYTE *)&key + i - 144) != 'f' )
    {
      v8 = i_1++;
      *((_BYTE *)&key + v8 - 144) = *((_BYTE *)&key + i - 144);
    }
  }
  *((_BYTE *)&key + i_1 - 144) = 0;
  return (unsigned int)fun1((unsigned __int64)&v12, (__int64)YouWouldChk::deAesFileAndMore(char *)::RH, a4) == 0;
```
## flag3 24566d882d4bc7ee

和flag1一样fun1处复制下来

然后拼到一起
> c92bb6a5a6c3009124566d882d4bc7ee

# maze

> 简单地图，不像上个月有多重路径搞得很迷惑

动调载入地图复制出来，稍微处理一下
```cpp
X         X X X X #
X X X     X         
    X     X X X X   
  X X           X   
  X         X X X   
  X X     X X       
    X     X         
    X X X X                        
```
md5(jkkjjhjjkjjkkkuukukkuuhhhuukkkk )

# Magia
```cpp
 for ( i = 31; i_1 < i; --i )
  {
    if ( (flag[i] ^ flag[i_1]) != *(&v59 + i_1) )
    {
      f("No_need_to_scare");
      return 0;
    }
    if ( (flag[i] & flag[i_1]) != *(&v43 + i_1) )
    {
      f("No_need_to_scare");
      return 0;
    }
    if ( (flag[i_1] & 0xF) != *(&v11 + i_1) || (flag[i] & 0xF) != *(&v11 + i) )
    {
      f("No_need_to_scare");
      return 0;
    }
    ++i_1;
  }
```
根据以上判断条件我们可以得到81个符合条件的flag
```cpp
Nep{mYrclU_a^dOmaxooisonotofree}
Nep{mYrclU_a^d_mahooisonotofree}
Nep{mYrclU_a^domaXooisonotofree}
Nep{mYrclU_andOmaxo_isonotofree}
Nep{mYrclU_and_maho_isonotofree}
Nep{mYrclU_andomaXo_isonotofree}
Nep{mYrclU_a~dOmaxoOisonotofree}
Nep{mYrclU_a~d_mahoOisonotofree}
Nep{mYrclU_a~domaXoOisonotofree}
Nep{mYrcle_a^dOmaxoois_notofree}
Nep{mYrcle_a^d_mahoois_notofree}
Nep{mYrcle_a^domaXoois_notofree}
Nep{mYrcle_andOmaxo_is_notofree}
Nep{mYrcle_and_maho_is_notofree}
Nep{mYrcle_andomaXo_is_notofree}
Nep{mYrcle_a~dOmaxoOis_notofree}
Nep{mYrcle_a~d_mahoOis_notofree}
Nep{mYrcle_a~domaXoOis_notofree}
Nep{mYrclu_a^dOmaxooisOnotofree}
Nep{mYrclu_a^d_mahooisOnotofree}
Nep{mYrclu_a^domaXooisOnotofree}
Nep{mYrclu_andOmaxo_isOnotofree}
Nep{mYrclu_and_maho_isOnotofree}
Nep{mYrclu_andomaXo_isOnotofree}
Nep{mYrclu_a~dOmaxoOisOnotofree}
Nep{mYrclu_a~d_mahoOisOnotofree}
Nep{mYrclu_a~domaXoOisOnotofree}
Nep{mirclU_a^dOmaxooisonot_free}
Nep{mirclU_a^d_mahooisonot_free}
Nep{mirclU_a^domaXooisonot_free}
Nep{mirclU_andOmaxo_isonot_free}
Nep{mirclU_and_maho_isonot_free}
Nep{mirclU_andomaXo_isonot_free}
Nep{mirclU_a~dOmaxoOisonot_free}
Nep{mirclU_a~d_mahoOisonot_free}
Nep{mirclU_a~domaXoOisonot_free}
Nep{mircle_a^dOmaxoois_not_free}
Nep{mircle_a^d_mahoois_not_free}
Nep{mircle_a^domaXoois_not_free}
Nep{mircle_andOmaxo_is_not_free}
Nep{mircle_and_maho_is_not_free}
Nep{mircle_andomaXo_is_not_free}
Nep{mircle_a~dOmaxoOis_not_free}
Nep{mircle_a~d_mahoOis_not_free}
Nep{mircle_a~domaXoOis_not_free}
Nep{mirclu_a^dOmaxooisOnot_free}
Nep{mirclu_a^d_mahooisOnot_free}
Nep{mirclu_a^domaXooisOnot_free}
Nep{mirclu_andOmaxo_isOnot_free}
Nep{mirclu_and_maho_isOnot_free}
Nep{mirclu_andomaXo_isOnot_free}
Nep{mirclu_a~dOmaxoOisOnot_free}
Nep{mirclu_a~d_mahoOisOnot_free}
Nep{mirclu_a~domaXoOisOnot_free}
Nep{myrclU_a^dOmaxooisonotOfree}
Nep{myrclU_a^d_mahooisonotOfree}
Nep{myrclU_a^domaXooisonotOfree}
Nep{myrclU_andOmaxo_isonotOfree}
Nep{myrclU_and_maho_isonotOfree}
Nep{myrclU_andomaXo_isonotOfree}
Nep{myrclU_a~dOmaxoOisonotOfree}
Nep{myrclU_a~d_mahoOisonotOfree}
Nep{myrclU_a~domaXoOisonotOfree}
Nep{myrcle_a^dOmaxoois_notOfree}
Nep{myrcle_a^d_mahoois_notOfree}
Nep{myrcle_a^domaXoois_notOfree}
Nep{myrcle_andOmaxo_is_notOfree}
Nep{myrcle_and_maho_is_notOfree}
Nep{myrcle_andomaXo_is_notOfree}
Nep{myrcle_a~dOmaxoOis_notOfree}
Nep{myrcle_a~d_mahoOis_notOfree}
Nep{myrcle_a~domaXoOis_notOfree}
Nep{myrclu_a^dOmaxooisOnotOfree}
Nep{myrclu_a^d_mahooisOnotOfree}
Nep{myrclu_a^domaXooisOnotOfree}
Nep{myrclu_andOmaxo_isOnotOfree}
Nep{myrclu_and_maho_isOnotOfree}
Nep{myrclu_andomaXo_isOnotOfree}
Nep{myrclu_a~dOmaxoOisOnotOfree}
Nep{myrclu_a~d_mahoOisOnotOfree}
Nep{myrclu_a~domaXoOisOnotOfree}
```
但是根据常识可以找到`Nep{mircle_and_maho_is_not_free}`

之后该flag参与一段smc，众所周知单层smc和没有是一样的

但是这个题后面出现了奇怪的东西

## smc内部：
```cpp
  v2 = 37;
  v3 = 110;
  v4 = 49;
  v5 = 19;
  v6 = 47;
  v7 = 40;
  v8 = 32;
  v9 = 60;
  v10 = 53;
  v11 = 52;
  v12 = 48;
  v13 = 109;
  v14 = 59;
  v15 = 54;
  v16 = 7;
  v17 = 60;
  v18 = 56;
  v19 = 127;
  v20 = 93;
  v21 = 84;
  v22 = 40;
  v23 = 30;
  v24 = 26;
  v25 = 47;
  v26 = 59;
  v27 = 43;
  v28 = 85;
  v29 = 54;
  v30 = 73;
  v31 = 109;
  v32 = 102;
  v33 = 126;
  v34 = 0;
  v35 = 1601399123;
  v36 = 1818588528;
  v37 = 1834967404;
  v0 = __inbyte(0x66u);
  v38 = 8545;
  v39 = BYTE2(dword_405150);
  v40 = 0i64;
  v41 = 0;
  v42 = 0;
  for ( i = 0; i < strlen(flag); ++i )
    *((_BYTE *)&v43 + i - 73) = *((_BYTE *)&v43 + (signed int)i % 18 - 37) ^ *((_BYTE *)&v43 + 4 * i - 205) ^ flag[i];

```
后面并没有对flag进行判断，直接把flag跑出来了。。。。

```cpp
.scode:00403219     loc_403219:
.scode:00403219 0EC xor     ecx, eax
.scode:0040321B 0EC mov     edx, [ebp-0D8h]
.scode:00403221 0EC mov     [ebp+edx-4Ch], cl  <---
.scode:00403225 1D8 jmp     loc_403186

Stack[0000633C]:0019FE58     a8b272473500a45 db '8b272473500a451286ab225413f1debd',0
```

> 8b272473500a451286ab225413f1debd 
 
# 521
看起来花里胡哨的rc4

cpp程序直接对着伪代码调试也可以得到比较好的效果

```cpp
v16 = __readfsqword(0x28u);
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::basic_string(&FLAG, a2);
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::basic_string(&a3, a2, v2);
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::basic_string(&v13, a2, v3);
  init_key((__int64)v15);//秘钥生成
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::operator=(&a3, v15);
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::~basic_string(v15);
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::length();
  if ( v4 != 37 )
  {
    a1 = 0;
    v5 = 0;
  }
  else
  {
    std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::basic_string(v15, &FLAG);
    v6 = chk2((__int64)v15);                    // 判断格式
    std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::~basic_string(v15);
    if ( v6 )
    {
      qmemcpy(v15, &unk_55AD074D7DC0, sizeof(v15));
      encode((__int64)&FLAG_1, (__int64)&FLAG, (__int64)&a3);//RC4加密
      std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::operator=(&v13, &FLAG_1);
      std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::~basic_string(&FLAG_1);
      for ( i = 0; ; ++i )
      {
        std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::length();
        if ( i >= v7 )
          break;
        if ( *(char *)std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::operator[](&v13, i) != v15[i] )
        {
          a1 = 0;
```

```cpp
{
  __int64 result; // rax
  unsigned int v5; // eax
  unsigned int v6; // [rsp+1Ch] [rbp-Ch]
  int v7; // [rsp+20h] [rbp-8h]
  unsigned int i; // [rsp+24h] [rbp-4h]

  rc4_init(a3, a4);
  v6 = 0;
  v7 = 0;
  for ( i = 0; ; ++i )
  {
    result = i;
    if ( a2 <= i )
      break;
    v6 = (unsigned __int8)(((unsigned int)((signed int)(v6 + 1) >> 31) >> 24) + v6 + 1)
       - ((unsigned int)((signed int)(v6 + 1) >> 31) >> 24);
    v5 = (unsigned int)((v7 + tab[v6]) >> 31) >> 24;
    v7 = (unsigned __int8)(v5 + v7 + tab[v6]) - v5;
    exchange((char *)&tab[v6], (char *)&tab[v7]);
    *(_BYTE *)((signed int)i + a1) ^= tab[(unsigned __int8)(tab[v6] + tab[v7])];
  }
  return result;
}
```
很明显的rc4,对着异或点看数据就行了

```python
l = [0x80, 0x59, 0x23, 0x35, 0x2b, 0x4, 0x8f, 0x1e, 0x55, 0x26, 0x32, 0xe8, 0x50, 0x57, 0x81, 0xa, 0xc4, 0x94, 0x25,
     0xdc, 0x84, 0x69, 0x76, 0xe6, 0x54, 0xb, 0x6e, 0xf3, 0x53, 0x31, 0x62, 0x49, 0xc, 0xff, 0xff, 0xfa, 0x22, 0x0]
flag = [0x4e, 0x65, 0x70, 0x7b, 0x31, 0x32, 0x33, 0x41, 0x42, 0x43, 0x30, 0x31, 0x32, 0x33, 0x34, 0x35, 0x36, 0x37,
        0x38, 0x41, 0x42, 0x43, 0x44, 0x45, 0x46, 0x47, 0x48, 0x49, 0x4a, 0x4b, 0x4c, 0x4d, 0x4e, 0x4f, 0x50, 0x51, 0x7d, 0x0]
encode = [0x80, 0x59, 0x23, 0x35, 0x22, 0x73, 0x8d, 0x1a, 0x51, 0x5d, 0x30, 0xe8, 0x57, 0x26, 0xf6, 0x7, 0xc6, 0x92,
          0x5e, 0xdc, 0x83, 0x1f, 0x76, 0x92, 0x25, 0xf, 0x65, 0xfb, 0x2e, 0x4d, 0x6b, 0x45, 0x3, 0x87, 0xe9, 0x9f, 0x22, 0x0]
for i in range(37):		  
	print(chr(encode[i]^(l[i]^flag[i])),end='')
```
> Nep{8E1EF8215BC841CAE5D17CCA77EAA7F4}
# pyCharm

我摊牌了,这题我蒙的 :D
![](http://roomoflja.cn/wp-content/uploads/2020/06/QQ截图20200627135651.jpg)

把很可疑的字符串拿出来,试了试是不行的,又觉得a很可疑,把a全去掉就可以正常解base64了....


```python
YamaNalaZaTacaxaZaDahajaYamaIa0aNaDaUa3aYajaUawaNaWaNajaMajaUawaNWI3M2NhMGM=
>>
YmNlZTcxZDhjYmI0NDU3YjUwNWNjMjUwNWI3M2NhMGM=
>>
bcee71d8cbb4457b505cc2505b73ca0c
```

