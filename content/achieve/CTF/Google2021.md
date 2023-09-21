---
title: "Google CTF 2021"
date: 2021-07-18 12:20:30+08:00
author: ljahum 
description: "Google ctf 2021"
categories: 
- CTF
tags:
- crypto
hiddenFromHomePage: false
math:
  enable: true
---
<!--more-->

![](https://raw.githubusercontent.com/ljahum/images/main/img/20210731190431.png)

[toc]

> google 的题真的太有意思了，可惜自己做出来的就只有这一个😥😥😥
> 有时间看看AESGCM怎么Oracle的

## FILESTORE

TASK

```python
# Copyright 2021 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

import os, secrets, string, time
# from flag import flag
flag = 'CTF{CR1M3_0f_d3dup1ic4ti0n}'


def main():
    # It's a tiny server...
    blob = bytearray(2**16)
    files = {}
    used = 0

    # Use deduplication to save space.
    def store(data):
        nonlocal used
        MINIMUM_BLOCK = 16
        MAXIMUM_BLOCK = 1024
        part_list = []
        while data:
            prefix = data[:MINIMUM_BLOCK]
            ind = -1
            bestlen, bestind = 0, -1
            while True:
                ind = blob.find(prefix, ind+1)
                if ind == -1: break
                length = len(os.path.commonprefix([data, bytes(blob[ind:ind+MAXIMUM_BLOCK])]))
                if length > bestlen:
                    bestlen, bestind = length, ind

            if bestind != -1:
                part, data = data[:bestlen], data[bestlen:]
                part_list.append((bestind, bestlen))
            else:
                part, data = data[:MINIMUM_BLOCK], data[MINIMUM_BLOCK:]
                blob[used:used+len(part)] = part
                part_list.append((used, len(part)))
                used += len(part)
                assert used <= len(blob)

        fid = "".join(secrets.choice(string.ascii_letters+string.digits) for i in range(16))
        files[fid] = part_list
        return fid

    def load(fid):
        # print(files)
        data = []
        for ind, length in files[fid]:
            data.append(blob[ind:ind+length])
        return b"".join(data)

    print("Welcome to our file storage solution.")

    # Store the flag as one of the files.
    store(bytes(flag, "utf-8"))

    while True:
        print()
        print("Menu:")
        print("- load")
        print("- store")
        print("- status")
        print("- exit")
        choice = input().strip().lower()
        if choice == "load":
            print("Send me the file id...")
            fid = input().strip()
            data = load(fid)
            print(data.decode())
        elif choice == "store":
            print("Send me a line of data...")
            data = input().strip()
            fid = store(bytes(data, "utf-8"))
            print("Stored! Here's your file id:")
            print(fid)
        elif choice == "status":
            print("User: ctfplayer")
            print("Time: %s" % time.asctime())
            kb = used / 1024.0
            print(len(blob))
            kb_all = len(blob) / 1024.0
            print("Quota: %0.3fkB/%0.3fkB" % (kb, kb_all))
            print("Files: %d" % len(files))
        elif choice == "exit":
            break
        else:
            print("Nope.")

try:
    main()
except Exception:
    print("Nope.")
time.sleep(1)
```

设计了一个字符串储存系统,会复用flag的字符串来达到减少内存的目的,观察内存可以判断输入的字符串是否位flag的子字符串

```shell
b'COqrVqqo621exq2q\n'
b'\nMenu:\n- load\n- store\n- status\n- exit\n'

ic| flag_: 'CTF{CR1M3_0f_d3d0f_d3dup1ic4ti0n0f_d3dup1ic4ti0n0f_d3dup1ic4ti0n0f_d3dup1ic4ti0n0f_d2'
ic| mome: 0.041, base: 0.036
b'\nFiles: 4\n\nMenu:\n- load\n- store\n- status\n- exit\n'
b'Send me a line of data...\n'
b"Stored! Here's your file id:\n"
b'a6m4SqnnLQPIYEIx\n'
b'\nMenu:\n- load\n- store\n- status\n- exit\n'

ic| flag_: 'CTF{CR1M3_0f_d3d0f_d3dup1ic4ti0n0f_d3dup1ic4ti0n0f_d3dup1ic4ti0n0f_d3dup1ic4ti0n0f_d3'
ic| mome: 0.041, base: 0.041
[*] Closed connection to 0.0.0.0 port 10001
[◐] Opening connection to 0.0.0.0 on port 10001: Done
Traceback (most recent call last):
```

exp

```python
flag_len = 26
import io
from pwn import *
from string import printable
from MyRE import CatData
host = 'filestore.2021.ctfcompetition.com'
prot= 1337

# flag='CTF{CR1M3_0f_d3dup1ic4ti0n}'
flag ='CTF'
tab = '`{|}0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!"#$%&\'()*+,-./:;<=>?@[\\]^_`~ \t\n\r\x0b\x0c'

def getmome():
    print(io.recvuntil('exit\n'))
    io.sendline('status')
    buf = io.recvuntil('/64.000kB')
    mome = float(CatData(buf,'Quota: ','kB')[0])
    return mome
def sendpayload(flag_):
    print(io.recvuntil('exit\n'))
    io.sendline('store')
    print(io.recvline())
    io.sendline(flag_)
    print(io.recvline())
    id = io.recvline()
    print(id)
# flag1 = 'CTF123123'
for i in range(100):
    io = remote(host, prot)
    base = getmome()
    for ch in printable:
        flag_ = flag+ch
        sendpayload(flag_)            
        mome = getmome()
        from icecream import *
        ic(flag_)
        ic(mome,base)
        if(mome>base):
            base = mome
        else:
            flag = flag +ch
            io.close()
            break
# flag1 = '{CR1M3_0f_d3dup1ic4ti0n}'
```

验证子序列的地方又一点问题，需要自行截断

