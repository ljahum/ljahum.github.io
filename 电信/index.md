# socket快乐无边

> 周五打了个电信的破比赛，本来从教室赶回来只剩一个半小时了，刚写好exp服务器有炸了，
> 一气之下直接吃饭 :( 晚上7点才打到flag


# RSA parity oracle

`RSA parity oracle`选择密文攻击的一种，可以解得密文，原理是二分法和模运算的性质，大比赛里面还没怎么见过直接扒wiki打法来出题的（逗比星盟除外）,利用条件也比较苛

## 核心二分 

[wiki](https://deploy-preview-479--ctf-wiki.netlify.app/crypto/asymmetric/rsa/rsa_chosen_plain_cipher/)

    lb = 0
    ub = N
    if server returns 1
        lb = (lb+ub)/2
    else:
        ub = (lb+ub)/2

## exp及其附件

~~附赠一个临时写的垃圾的socket类~~

[github](https://github.com/ljahum/Crypto/tree/main/%E9%80%89%E6%8B%A9%E6%98%8E%E6%96%87)



