---
title: cpp_vector
date: 2020-03-20 01:26:54
tags: 
- codes
categories:
- posts
---
# 简单实例:

```cpp
#include<iostream>
#include<stdlib.h>
#include<string>
#include<vector>
using namespace std;
class local{
    public:
        int x;
        int y;
};
int main()
{
    vector<int> a;
    vector<string> b;
    vector<local> c;//向量的申明
    a.push_back(1);
    b.push_back("acdc");
    local l1;//类或者结构体要先初始再放进去
    l1.x=1;
    l1.y=2;
    c.push_back(l1);//用于在队尾压入数据push_back成员函数
    cout<<"a[0]="<<a[0]<<endl;
    cout<<"b[0]="<<b[0]<<endl;
    cout<<"c[0].x="<<c[0].x<<"c[0].y="<<c[0].y<<endl;
    a.push_back(2);
    cout<<"a_len="<<a.size()<<endl;//成员函数size()
    a.pop_back();//pop_back()移除队尾元素
    cout<<"a_len="<<a.size()<<endl;
    system("pause");
    return 0;
}
/*
输出：
a[0]=1
b[0]=acdcz
c[0].x=1c[0].y=2
a_len=2
a_len=1
请按任意键继续. .
*/
```
## 其他常用成员函数:

```cpp
clrean()//清空向量
empty()//询问是否还有元素
```

0. ### 迭代器
  
  
  - #### 利用向量输出一个字符三角形：

```cpp
#include<iostream>
#include<stdlib.h>
#include<string>
#include<vector>
using namespace std;
int main()
{
    vector<string> a;
    vector<string>::iterator pa;//常规迭代器：可以修改值
    vector<string>::const_iterator pb;//常量迭代器：不可以修改值
    for(int i=0;i<10;i++)
    {
        string s;
        for(int j=0;j<i+1;j++)
        {
            s=s+'a';
        }
        a.push_back(s);
    }//初始化一个三角形进去
    for(pa=a.begin();pa!=a.end();pa++)
        cout<<*pa<<endl;
    cout<<"\n";
    system("pause");
    return 0;
}
/*
输出：
a
aa
aaa
aaaa
aaaaa
aaaaaa
aaaaaaa
aaaaaaaa
aaaaaaaaa
aaaaaaaaaa
*/
```

- ### 和迭代器相关的常用函数：
  
  ```cpp
  insert(迭代器，元素)//向迭代器指向的前一位插入元素
  erase（迭代器）//删除迭代器指向元素
  ```
