---
title: "A simple tour of C"
subtitle: 
date: 2021-09-27 00:07:00+08:00
# weight: 1000
draft: true
author: "ljahum"
description: "review C"
tags: 
- codes
# crypto 25math 6codes 5bin 4Nuil 
categories: 
- posts
# - CTF posts notes 其他


hiddenFromHomePage: false
math:
  enable: true
---
<!--more-->



> 花两天时间复习了一下c

## gmp

GNU MP库是一个大整数和多精度浮点数的运算库。它本身是用C语言写成的，但也提供了C++绑定。

当用C++写程序时，如果你不是自虐狂或者狂热的手动编译器变换爱好者，那么用C++绑定毫无疑问是更好的选择。

这是因为，C语言版本的绑定把所有操作都封装成了类似于汇编语言中的指令。比如说，如果要算一个大整数版本的`a+b`，那么应该这么写：

```c
mpz_t a, b, c;
mpz_init_set_ui(a, 1);
mpz_init_set_ui(b, 2);
mpz_add(c, a, b);
mpz_clear(a);
mpz_clear(b);
mpz_clear(c);
```

光是定义变量和清理内存就已经把孩子整吐了	

这种情况的根本原因是，C语言中没有方便的手段来进行内存资源管理和快速结构构造

导致C语言虽然可以实现**表达式求值模型**，但无法方便地实现**自定义类型的表达式求值模型**

C++绑定这时堪称救世主，极大地减轻了被python毒害的人的思考负担：

```c++
const mpz_class a {1}, b {2};
const auto c = a + b;
```

这段代码可以执行和上面C语言代码完全相同的行为

看了一眼源码，只能说学到很多

运算符重载：

```c++
  // compound assignments
  __GMP_DECLARE_COMPOUND_OPERATOR(operator+=)
  __GMP_DECLARE_COMPOUND_OPERATOR(operator-=)
  __GMP_DECLARE_COMPOUND_OPERATOR(operator*=)
  __GMP_DECLARE_COMPOUND_OPERATOR(operator/=)

  __GMP_DECLARE_COMPOUND_OPERATOR_UI(operator<<=)
  __GMP_DECLARE_COMPOUND_OPERATOR_UI(operator>>=)

  __GMP_DECLARE_INCREMENT_OPERATOR(operator++)
  __GMP_DECLARE_INCREMENT_OPERATOR(operator--)
```



实现一个pow：

可以看到对于一般的运算符编写来说，我们习惯于去疯狂嵌套一个公式出来

但gmpxx涉及到临时变量的地方，需要我们手动设置一个专用的临时变量（和其他运算符重载的操作是一样的）

```c++
mpz_class powerMod(mpz_class b, mpz_class n, mpz_class m)
{
	mpz_class a {1};
	int i;
    int  k=0;
    mpz_class  num = n;
    mpz_class tmp;
	
	while(num)
	{
		num = num>>1;
		++k;
	}

	for(i=0; i<k; ++i)
	{
		
        tmp = n>>i;
        tmp = tmp &1;
		if(tmp == 1)
			a = a*b %m;
		b = b*b %m;
	}

	return a;
}
```

## RAII



RAII，也称为“资源获取就是初始化”，是C++等编程语言常用的管理资源、避免内存泄露的方法。它保证在任何情况下，使用对象时先构造对象，最后析构对象。

> 智能指针（std::shared_ptr和std::unique_ptr）即RAII最具代表的实现，使用智能指针，可以实现自动的内存管理，再也不需要担心忘记delete造成的内存泄漏。

但根据pwn👴的描述,还是有洞可以打,但是这项技术拔👴的工作量降低了,还是整挺好

## 与cmake斗智斗勇

一个模板cmakelist：

```cmake
# 版本号
cmake_minimum_required(VERSION 3.0)
PROJECT(Test)
# 设置变量
set(PROJECT_ROOT ${CMAKE_CURRENT_LIST_DIR})

message("project dir:${PROJECT_ROOT}")
# 头文件目录
INCLUDE_DIRECTORIES(./include/)
# INCLUDE_DIRECTORIES(/usr/local/include/)
# 设置源码文件位置
AUX_SOURCE_DIRECTORY(./src/  DIR_SRCS)
# 设置编译器版本
SET(CMAKE_CXX_STANDARD 11)
# bin文件位置
add_executable(test ${DIR_SRCS})
```

运行完事了就会有一个熟悉的build目录了

![](https://raw.githubusercontent.com/ljahum/images/main/img/20210928191522.png)