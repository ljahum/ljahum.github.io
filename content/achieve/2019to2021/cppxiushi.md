---
title: 修饰函数规则
date: 2020-03-14 15:12:51
tags: 
- codes
categories:
- posts
---
## C编译器的函数名修饰规则 ：
__stdcall在输出函数名前加上一个下划线前缀，函数名后面加上一个“@”符号和其参数的字节数，例如_functionname@number。
__cdecl调用约定仅在输出函数名前加上一个下划线前缀，例如_functionname。
__fstcall在输出函数名前加上一个“@”符号，后面也是一个“@”和其参数的字节数，例如@functionname@number
## C++编译器的函数名修饰规则：
以一个“?”开始，后跟函数名，再后面是参数表的**开始标识**和**按照参数类型代号拼出的参数表**。
__stdcall 开始标识是 “@@YG”
__cdecl 是 “@@YA”
__fastcall是 “@@YI”
#### 参数表的拼写代号如下所示：  
```cpp
X--void    
D--char    
E--unsigned char    
F--short    
H--int    
I--unsigned int    
J--long    
K--unsigned long（DWORD） 
M--float    
N--double    
_N--bool 
U--struct
```
#### 其他参数表示
用PA表示指针，用PB表示const类型的指针。后面的代号表明指针类型，如果相同类型的指针连续出现，以“0”代替，一个“0”代表一次重复。
U表示结构类型，通常后跟结构体的类型名，用“@@”表示结构类型名的结束。函数的返回值不作特殊处理，和函数一样，紧跟着参数表的开始标志，也就是说，函数参数表的第一项实际上是表示函数的返回值类型。

参数表后以“@Z”标识整个名字的结束，如果该函数无参数，则以“Z”标识结束。下面举两个例子，假如有以下函数声明：
```cpp
int Function1 (char *var1,unsigned long); 
其函数修饰名为“?Function1@@YGHPADK@Z”，而对于函数声明： 

?[name][调用方式][参数1][	参数2]....[@z/z]

void Function2(); 
其函数修饰名则为“?Function2@@YGXXZ”
```
### 类的成员函数：
```cpp
主要结构：
?[name][类名][保护类型][参数1][	参数2]....[@z/z]


class CTest 
{ 
private: 
    void Function(int); 
protected: 
    void CopyInfo(const CTest &src); 
public: 
    long InsightClass(DWORD dwClass) const; 
```
成员函数调用方式是__thiscall,在函数名和参数表之间插入“@”字符引导的类名
#### 三种类型表示：
公有 public 标识“@@QAE”
保护 protected 标识是“@@IAE”
私有 private 标识是“@@AAE”

对于成员函数Function，其函数修饰名为“?Function@CTest@@AAEXH@Z”
```cpp
	?Function @CTest @@AAE XH@Z”
			  类名    私有
```

如果函数声明使用了const关键字，则相应的标识应分别为“@@QBE”，“@@IBE”和“@@ABE”。

参数类型是类实例的引用,则使用“AAV1”
`CopyInfo( CTest &src)`
若将其作为const类型的引用，则使用“ABV1”
`CopyInfo(const CTest &src)`

函数CopyInfo只有一个参数，是对类CTest的const引用参数，其函数修饰名为:
```cpp
?CopyInfo @CTest   @@IAE     X       ABV1         @@Z
				  protected    为const类型的引用
```

InsightClass是一个**共有的const函数**，它的成员函数标识是“@@QBE":
```cpp
?InsightClass@CTest  @@QBE   J K  @Z”。 
					public
```


https://www.cnblogs.com/CodeMIRACLE/p/5343660.html
 