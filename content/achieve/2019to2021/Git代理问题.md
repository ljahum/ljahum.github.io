---
title: Git代理问题
date: 2020-05-07 15:33:37
tags: 
- codes
categories:
- posts
---
## 被一个很坑爹的问题折腾了一天-解决git下载速度过慢


> 碰到的问题都是比较个例的，所以才会放在博客

### 几种常见的解决方案：

###  1. 更改hosts文件：

进入这个网站[](https://www.ipaddress.com/)

查询以下网址的 ip

```
github.com
github.global.ssl.fastly.net
codeload.github.com
```

用cmd指令 ping 一下ip

```
ping xxx.xxx.xxx.xxx
```

如果ping得通就按以下格式添加到hosts文件中

hosts文件位置：C:\Windows\System32\drivers\etc

像这样：

```
# Copyright (c) 1993-2009 Microsoft Corp.
#
# This is a sample HOSTS file used by Microsoft TCP/IP for Windows.
#
# This file contains the mappings of IP addresses to host names. Each
# entry should be kept on an individual line. The IP address should
# be placed in the first column followed by the corresponding host name.
# The IP address and the host name should be separated by at least one
# space.
#
# Additionally, comments (such as these) may be inserted on individual
# lines or following the machine name denoted by a '#' symbol.
#
# For example:
#
#      102.54.94.97     rhino.acme.com          # source server
#       38.25.63.10     x.acme.com              # x client host

# localhost name resolution is handled within DNS itself.
# 127.0.0.1       localhost
# ::1             localhost
# Copyright (c) 1993-2009 Microsoft Corp.
#
# This is a sample HOSTS file used by Microsoft TCP/IP for Windows.
#
# This file contains the mappings of IP addresses to host names. Each
# entry should be kept on an individual line. The IP address should
# be placed in the first column followed by the corresponding host name.
# The IP address and the host name should be separated by at least one
# space.
#
# Additionally, comments (such as these) may be inserted on individual
# lines or following the machine name denoted by a '#' symbol.
#
# For example:
#
#      102.54.94.97     rhino.acme.com          # source server
#       38.25.63.10     x.acme.com              # x client host

# localhost name resolution is handled within DNS itself.
# 127.0.0.1       localhost
# ::1             localhost



140.82.114.4 github.com

199.232.69.194 github.global.ssl.fastly.net

140.82.112.9 codeload.github.com

#以下网址可有可无
185.199.108.153 assets-cdn.github.com
199.232.68.133 raw.githubusercontent.com
199.232.68.133 cloud.githubusercontent.com
199.232.68.133 camo.githubusercontent.com
199.232.68.133 avatars0.githubusercontent.com
199.232.68.133 avatars1.githubusercontent.com
199.232.68.133 avatars2.githubusercontent.com
199.232.68.133 avatars3.githubusercontent.com
199.232.68.133 avatars4.githubusercontent.com
199.232.68.133 avatars5.githubusercontent.com
199.232.68.133 avatars6.githubusercontent.com
199.232.68.133 avatars7.githubusercontent.com
199.232.68.133 avatars8.githubusercontent.com

```

执行`ipconfig /flushdns`命令，刷新 DNS 缓存

### 2. 修改代理（较为有效）：

   首先查看gitconfig文件是否已经有其他代理：

   位置：`C:\Users\xxxxx\.gitconfig`

   ```
   [user]
    email = 1695325350@qq.com
    name = ljahum
   #把这行字以下的设置全部注释掉
   #[http "https://github.com"]
   #  proxy = https://127.0.0.1:1086
   #[https "https://github.com"]
   #  proxy = https://127.0.0.1:10863
   #[http "http://github.com"]
   #  proxy = http://127.0.0.1:10808
   #[http "https://github.com"]
   #  proxy = http://127.0.0.1:10808
   [http]
           proxy = socks5://127.0.0.1:4781
   [https]
           proxy = socks5://127.0.0.1:4781
   ```

   动动小手打开 vpn 查看 http(s) 端口和 socks 端口的值

   >  注意如果租的梯子一定要到卖家哪里更新客户端再查看

   按以下格式写入 .gitconfig 中：

   ```
   [http]
           proxy = socks5://127.0.0.1:[socks 端口的值]
   [https]
           proxy = socks5://127.0.0.1:[socks 端口的值]
   ```

   也可以改成http（s）的那种，最终效果如下：
   ```
[user]
#用户名与邮箱，不用管
  email = 1695325350@qq.com
  name = ljahum
#socks5:
[http]
        proxy = socks5://127.0.0.1:4781
[https]
        proxy = socks5://127.0.0.1:4781
#http(s):        
[http]
  proxy = 127.0.0.1:4780
[https]
  proxy = 127.0.0.1:4780


   ```

   ### 3. 把所需的库搬到gitee上

   教程很多就不写了,网上到处都是，缺点是有些东西在安装的时候会访问github的库。

作为个人或者团队之间的仓库快的飞起。

  



