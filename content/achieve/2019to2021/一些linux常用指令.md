---
title: 一些linux常用指令
date: 2020-05-08 23:38:59
tags: 
- codes
categories:
- posts
---
### 一些（最近用到的）常用指令：

> 没事不想用vim : (

```shell
ls 	-i详细信息显示  [路径(默认当前目录)]
	-a隐藏文件显示
	-h大小按kb显示
	A*.txt 搜索A开头.txt结尾的文件
#对文件：
mv 1 1.txt #1 改为 1.txt
#对文件夹：
mv fld fld2#若fld2存在则移动fld到fld2下，否则改名为fld2
whereis xxxxx #查找文件位置
vim [文件名]

命令(一般)模式下：
i #进入编辑模式
shift + `：`#打开底部命令行
/ #按下后输入字符串可以匹配字符串
gg #回第一行

shift + x #类似windows的backspace
dd #行消除
yy #复制本行
shift + p #粘贴在光标上一行
p #粘贴在光标下一行
u #类似ctrl + z
[输入数字n] + 回车 #向下跳n行
ctrl f#对应uppage
ctrl b#对应downpage
#多行注释
ctrl + v --> 选择多行并shift + i--> 输入字符回车--->字符会复制到选择起来的所有行前面     如果输入`#`或者`//`可以达到多行注释的效果

底部命令模式：
ESC回到命令（一般）模式
q 退出
w 保存
q！强制退出不保存
w！强制保存
1,s/A/B/gc #从第一行开始找 A 并询问是否换成 B
询问选项：y yes
		n next
		a 光标以下的all （可能很有用）
		q quit
编辑模式：
inster #在替换和插入之间切换

添加环境变量：
export PATH=[path] //不需要括号和引号
如：export PATH=$PATH:/mnt/c/Users/16953/bin/

sudo password root//重置密码
sudo -i//切换至sudo用户

chmod 777 file//权限拉满
chmod 000 file//权限全关
      abc

a:User
b:Group
c:Other

权限选项：
r：读
w：写
x：执行


ubuntu config --default-user root 设置默认用户以为root

w
who 当前**在登陆**用户

whoami 查询当前用户

cat /etc/passwd 查看所有用户，但是很丑
lastlog 比上面那个好看一些

apt-get update                  // 更新安装源（Source）
apt-get upgrade                 // 更新已安装的软件包
		dist-upgrade            //更新依赖

apt-cache search PackageName        // 搜索软件包
```


 