---
title: wsl安装angr
date: 2020-05-08 22:53:30
tags: 
- codes
  
categories:
- posts
---
## 在wsl上安装angr框架

> 折腾了好几天终于是把这套东西给折腾完了，windows上折腾一天最后还是无法解决报错，干脆按在了wsl上。就把从头到尾踩过的坑全部拿出来写一下好了

先检查新装好的 wsl 有没有安装 pip ，没有的话系统会提示用apt安装。

 ### 安装 virtualenv：

```shell
pip3 install virtualenv
```

> 在python开发中，我们可能会遇到一种情况，就是当前的项目依赖的是某一个版本，但是另一个项目依赖的是另一个版本，这样就会造成依赖冲突，而virtualenv就是解决这种情况的，virtualenv通过创建一个虚拟化的python运行环境，将我们所需的依赖安装进去的，不同项目之间相互不干扰，如下所示。 

意思就是说angr会影响python的环境所以要新建一个独立的python环境（具体是一个文件夹）

先新建到一个angr专用文件夹：

```shell
ljahum@LAPTOP-3PRLLIJT:/mnt/c/Users/16953$ mkdir angr_enviroment
ljahum@LAPTOP-3PRLLIJT:/mnt/c/Users/16953$ cd angr_enviroment/
```

### 新建一个 python 独立运行环境：

```shell
virtualenv -p python位置 myenv
```

python位置这样获得：

```shell
ljahum@LAPTOP-3PRLLIJT:/mnt/c/Users/16953$ whereis python
python: /usr/bin/python3.6 /usr/bin/python3.6-config /usr/bin/python3.6m /usr/bin/python3.6m-config /usr/lib/python2.7 /usr/lib/python3.6 /usr/lib/python3.7 /usr/lib/python3.8 /etc/python3.6 /usr/local/lib/python3.6 /usr/include/python3.6 /usr/include/python3.6m /usr/share/python /mnt/c/Program Files (x86)/NetSarang/Xshell 6/python34.dll /mnt/c/Program Files (x86)/NetSarang/Xshell 6/python34.zip /mnt/c/py374/python.pdb /mnt/c/py374/python3.dll /mnt/c/py374/python3.exe /mnt/c/py374/python37.dll /mnt/c/py374/python37.pdb /mnt/c/Windows/system32/python27.dll /mnt/c/p27/python2.exe /mnt/c/Users/16953/AppData/Local/Microsoft/WindowsApps/python.exe /mnt/c/Users/16953/AppData/Local/Microsoft/WindowsApps/python3.exe
ljahum@LAPTOP-3PRLLIJT:/mnt/c/Users/16953$
```

复制粘贴第一个`/usr/bin/python3.6`

> 具体位置可能会有差异

运行：`ljahum@LAPTOP-3PRLLIJT:/mnt/c/Users/16953/bin/wslhome/angr_enviroment$ virtualenv -p /usr/bin/python3.6 myenv`

应该可以用windows资源管理器在angr_enviroment文件夹下看的一个新建的myenv文件夹，

### 安装angr：

```shell
#启动环境
ljahum@LAPTOP-3PRLLIJT:/mnt/c/Users/16953/bin/wslhome/angr_enviroment$ source myenv/bin/activate
#安装angr
(myenv) ljahum@LAPTOP-3PRLLIJT:/mnt/c/Users/16953/bin/wslhome/angr_enviroment$ pip3 install angr
。
。
安
装
中
。
。
(myenv) ljahum@LAPTOP-3PRLLIJT:/mnt/c/Users/16953/bin/wslhome/angr_enviroment$ python3
Python 3.6.9 (default, Apr 18 2020, 01:56:04)
[GCC 8.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import angr
>>>
#未报错则成功
#关闭环境
(myenv) ljahum@LAPTOP-3PRLLIJT:/mnt/c/Users/16953/bin/wslhome/angr_enviroment$ deactivate
ljahum@LAPTOP-3PRLLIJT:/mnt/c/Users/16953/bin/wslhome/angr_enviroment$
```



之后要使用angr的时候也要开启环境所以尽量装在wsl初始化界面附近

对python虚拟环境的一点点解释：（感觉其实就是备份了一个拿来给特定应用折腾）

> 针对每个应用创建独立运行的python环境，这样就可以对每个应用的pyhton环境进行隔离。原理就是把系统python复制一份到virtualenv的环境，用命令source venv/bin/activate进入一个virtualenv环境的时候，virtualenv会修改相关的环境变量，让命令python和pip均指向当前的virtualenv环境。这个时候命令提示符号就变了，前面有一个（venv)前缀  :3




