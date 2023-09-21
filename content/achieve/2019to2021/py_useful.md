---
title: python一些常用语法（自认为）
date: 2020-03-20 01:08:00
tags: 
- codes
categories:
- posts
---
# bytes类型:

> Python 3 新增了 bytes 类型，用于代表字节串（这是作者生造的一个词，与字符串对应）。字符串（str）由多个字符组成，以字符为单位进行操作；字节串（bytes）由多个字节组成，以字节为单位进行操作。

bytes实例在python3中得到了大量使用（不知道为什么），在很多时候需要对其他类型的数据用bytes转一下才能用。

 结构：

`bytes(class bytes(source, encoding, errors) )`
根据source的不同，最终得到的输出结果也不同

- 如果 source 为整数，则返回一个长度为 source 的初始化数组；
- 如果 source 为字符串，则按照指定的 encoding 将字符串转换为字节序列；
- 如果 source 为可迭代类型，则元素必须为[0 ,255] 中的整数；
- 如果 source 为与 buffer 接口一致的对象，则此对象也可以被用于初始- 化 bytearray（以后加）
- 果没有输入任何参数，默认就是初始化数组为0个元素 
  1.  整数：
      
      ```python
      b = bytes(3)
      print(b,type(b))
      #输出：
      #b'\x00\x00\x00' <class 'bytes'>
      ```
  2.  字符串：
      
      ```python
      s='1234'
      s2='牛批'
      b = bytes(s , encoding="utf8")
      b2 = bytes(s2 , encoding="utf8")
      print(b,type(b))
      print(b2,type(b2))
      '''
      输出：
      b'1234' <class 'bytes'>
      b'\xe7\x89\x9b\xe6\x89\xb9' <class 'bytes'> 
      '''
      ```
  3.  可迭代类型（list为首）：
      
      ```python
      b = bytes([1, 2, 3])
      print(b, type(b))
      print(str(b))
      #c = bytes([1, 2, 888])   
      # 888不再(0, 256之间)会报错
      '''
      输出
      b'\x01\x02\x03' <class 'bytes'>
      b'\x01\x02\x03'
      '''
      ```
  4. bytea类型的成员函数： ```python
      
      b=b'\x01\x02\x03\x04\x05\x06\x07\x08'
      print(b.hex())
      #0102030405060708 (str)
      ```

## map():

 基本结构：

`map( 函数，可迭代对象1,可迭代对象2......)`

- 函数：chr(),ord().....
- 可迭代对象：list,字符串等可以用for遍历的对象 ##### 例子：
  
  ```python
  def f(x):
      return x+1
  l1=[1,2,3,4]
  it=map(f,l1)
  #py2中直接返回list py3中返回一个迭代器
  print(it)
  print(list(it))
  #print(next(it))
  #next 不能用于解析 map返回 要用list
  ```
  
   输出:
  
  ```python
  <map object at 0x000001A87E5AF788>
  [2, 3, 4, 5]
  ```
  
   多个对象：
  
  ```python
  def f(x,y,z):
      return x+y+z
  l1=[1,2,3,4]
  l2=[4,3,2,1]
  l3=[1,2,3,4,]
  it=map(f,l1,l2,l3)
  print(it)
  print(list(it))
  ```
  
   Output:
  
  ```python
  <map object at 0x000001A544CE55C8>
  [6, 7, 8, 9]
  ```

 例：

```python
arr=[49,50,51,52]
str='1234'
it = map(chr,arr)
it2 = map(ord,str)
print(it)
print(list(it))
print(it2)
print(list(it2))
```

 输出：

```python
<map object at 0x000001B95E7E8308>
['1', '2', '3', '4']
<map object at 0x000001B95D8D2F88>
[49, 50, 51, 52]
```

### join():
```python
s='123'
s1 = "-"
s2 = ""
seq = ("r", "u", "n", "o", "o", "b") # 字符串序列
print (s1.join( seq ))
print (s2.join( seq ))
a="000".join(s)
#等价于 a="000"
#      a.join(s)
print(a)
```

Output：

```python
r-u-n-o-o-b
runoob
123
```

### repr:(好吧这个不是很常用)
把所有输出全部作为字符串输出
```pytohn
>>>s = 'RUNOOB'
>>> repr(s)
"'RUNOOB'"
>>> dict = {'runoob': 'runoob.com', 'google': 'google.com'};
>>> repr(dict)
"{'google': 'google.com', 'runoob': 'runoob.com'}"
>>>
```