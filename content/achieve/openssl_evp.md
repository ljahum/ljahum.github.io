---
title: OpenSSL 调用EVP框架 几种加密模式
date: 2023-02-15 15:39:27+08:00
author: ljahum 

tags: 
- codes

categories: 
- posts

math:
  enable: true


---
<!--more-->
## 简介

EVP系列的函数的声明包含在evp.h里面
封装了openssl加密库里面所有算法的函数。通过这样的统一的封装，使得只需要在初始化参数的时候做很少的改变，就可以使用相同的代码调用不同的加密算法


ctx上下文可以被认为是配置选项生效的“范围”。

加载提供者时，它仅在给定库上下文的范围内加载。

通过这种方式，复杂应用程序的不同组件可以各自使用不同的库上下文，并让不同的提供程序加载不同的配置设置。

主要模块

- 摘要
EVP_MD 
- 对称密码
EVP_CIPHER 
- 消息验证码
EVP_MAC 
- 密钥派生函数
EVP_KDF 
- 密钥交换
EVP_KEYEXCH 
- 非对称密码
EVP_ASYM_CIPHER 
- 公钥密码(相比对称强调秘钥和算法)
EVP_PKEY_CTX 

- 非对称密钥封装
EVP_KEM 
- 编码
OSSL_ENCODER 

## openssl安装

```powershell
scoop install openssl 
```

查看环境变量

```powershell
get-ChildItem env:
...
OPENSSL_CONF                   D:\Scoop\apps\openssl\current\bin\cnf\openssl.cnf
OPENSSL_INCLUDE_DIR            D:\Scoop\apps\openssl\current\include
OPENSSL_LIB_DIR                D:\Scoop\apps\openssl\current\lib
OPENSSL_MODULES                D:\Scoop\apps\openssl\current\bin
```

makefile配置(clion)

```powershell
#cmake_minimum_required(VERSION 3.24)

cmake_minimum_required(VERSION 3.17)
project(openssl1 C)

set(CMAKE_C_STANDARD 99)
set(LINK_DIR $ENV{OPENSSL_LIB_DIR})
set(INC_DIR "D:\\Scoop\\apps\\openssl\\current\\include") # 手动配路径的方法
#include_directories($ENV{OPENSSL_INCLUDE_DIR}) 从系统读路径的方法
include_directories(${INC_DIR})
#link_directories($ENV{OPENSSL_LIB_DIR})
link_directories(${LINK_DIR})

link_libraries(openssl libcrypto)
add_executable(openssl1 publicKeyEnc.c)#这里add的文件只能有一个main
target_link_libraries(openssl1 openssl)
```

## openssl evp 摘要算法


1. new一个md ctx
2. ctx初始化
3. 指定摘要类型的初始化
4. 对消息多次摘要
5. 摘要结束 输出结果

```c
#include <stdio.h>
#include <string.h>
#include "openssl/evp.h"
void test_md5(unsigned char * msg,unsigned char * output){
    printf("start test MD digest\n");
    char * msg1 = msg;
    unsigned char md_value[EVP_MAX_MD_SIZE];
    unsigned int md_len;
    EVP_MD_CTX *pmdctx = EVP_MD_CTX_new();
    EVP_MD_CTX_init(pmdctx);
    EVP_DigestInit(pmdctx,EVP_md5());
    //该函数功能跟EVP_DigestInit_ex函数同样，可是ctx?数能够不用初始化，并且该函数仅仅使用缺省实现的算法。
    EVP_DigestUpdate(pmdctx,msg1, strlen(msg1));
    EVP_DigestFinal(pmdctx,md_value,&md_len);

    md_value[md_len]= (unsigned char) '\0';//加0截断
    //返回参数
    strcpy(output,md_value);
    output[strlen(md_value)] = (unsigned char) '\0';

    EVP_MD_CTX_free(pmdctx);

    //该函数功能跟EVP_DigestFinal_ex函数同样，可是ctx结构会自己主动清除。一般来说，如今新的程序应该使用EVP_DigestInit_ex和EVP_DigestFinal_ex函数
}
void test_sm3(){
    printf("start test SM3 digest\n");
    unsigned char md_value[EVP_MAX_MD_SIZE];        //保存输出的摘要值的数组
    unsigned int md_len;
    EVP_MD_CTX *pmdctx = EVP_MD_CTX_new();          //EVP消息结构体
    char msg1[] = "Test Message1";                  //待计算摘要的消息1
    char msg2[] = "Test Message2";                  //待计算只要的消息2
    int i=0;


    EVP_MD_CTX_init(pmdctx);                        //初始化摘要结构体

    //设置摘要算法和密码算法引擎，这里密码算法使用SM3
    //算法引擎使用OpenSSL默认引擎，即软算法
    EVP_DigestInit_ex(pmdctx,EVP_sm3(),NULL);
    EVP_DigestUpdate(pmdctx,msg1,strlen(msg1));     //调用摘要Update计算msg1的摘要
    EVP_DigestUpdate(pmdctx,msg2,strlen(msg2));     //调用摘要Update计算msg2的摘要
    EVP_DigestFinal_ex(pmdctx,md_value,&md_len);    //摘要结束，输出摘要值


    /* 打印结果 */
    printf("msgis: %s %s：\n",msg1,msg2);
    for(i=0;i<md_len;i++)
    {
        printf("%02X",md_value[i]);
    }
    printf("\n");
    EVP_MD_CTX_free(pmdctx);
}

int main() {
    printf("Hello, OpenSSL!\n");

    /* 加载所有算法 */
    //    OpenSSL_add_all_algorithms();
    test_sm3();



    //MD5
    unsigned char msg1[] = "hello world";
    unsigned char * output;
    test_md5(msg1,output);
    printf("MD5(\"%s\")=",msg1);
    for (int i = 0; i < strlen(output); ++i) {
        printf("%02X",output[i]);
    }
    return 0;
}
```
##  openssl RSA密钥对生成模板


```c

#include <stdio.h>
#include <string.h>
#include <openssl/evp.h>
#include "openssl/rsa.h"


void Test_Rsa(){

/*1.new一个公钥ctx对象 指定加密算法类型
 * 2.对其初始化
 * 3.设置rsa参数
 * 4.正式生成密钥
 * 5.释放ctx
 * */
    printf("hello RSA\n");
    EVP_PKEY * Rsa_Pkey=NULL;
    EVP_PKEY_CTX *Rsa_ctx = EVP_PKEY_CTX_new_id(EVP_PKEY_RSA,NULL);
    EVP_PKEY_keygen_init(Rsa_ctx);
    //前两步均在初始化 （不能合到一起吗）

//EVP_PKEY_CTX_new() 函数使用pkey密钥类型和 ENGINE e分配公钥算法上下文。
//
//EVP_PKEY_CTX_new_id() 函数使用id和 ENGINE e指定的密钥类型分配公钥算法上下文
    EVP_PKEY_CTX_set_rsa_keygen_bits(Rsa_ctx,1024);
    EVP_PKEY_keygen(Rsa_ctx,&Rsa_Pkey);

    EVP_PKEY_CTX_free(Rsa_ctx);

    BIGNUM *d=0;
//    BIGNUM *e=0;
//    BIGNUM *n=0;
    EVP_PKEY_get_bn_param(Rsa_Pkey,"d",&d);
//    EVP_PKEY_get_bn_param(Rsa_Pkey,"e",&e);
//    EVP_PKEY_get_bn_param(Rsa_Pkey,"n",&n);
    char *Hex_d;
    Hex_d = BN_bn2hex(d);//大数转化为十六进制字符串
//    BN_bn2dec() 十进制
    printf(" rsa私钥长度=%dbits 私钥十六进制形式:\n",strlen(Hex_d)*4);
    printf("\n");
    for (int i = 0; i < strlen(Hex_d); ++i) {
        printf("%C",Hex_d[i]);
    }
    printf("\n");
    EVP_PKEY_CTX_free(Rsa_ctx); //释放ctx


}

int main(){
    Test_Rsa();
    return 0;


}

```



## openssl 对称密码模板 aesecb

```c
//
// Created by 16953 on 2023/2/14.
//
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <openssl/evp.h>
#include <openssl/aes.h>
int main(void)
{
/*
 * 1. new aesctx
 * 2.initex
 * 3.update
 * 4.final
 * 5.free
 * */


    unsigned char key[32] = {1};
    unsigned char iv[16] = {0};
    unsigned char *inStr = "this is test string";
    int inLen = strlen(inStr);
    int encLen = 0;
    int outlen = 0;
    unsigned char encData[1024];

    printf("source: %s\n",inStr);

    //加密
    EVP_CIPHER_CTX *ctx= EVP_CIPHER_CTX_new();
//    ctx = EVP_CIPHER_CTX_new();

    EVP_CipherInit_ex(ctx, EVP_aes_256_ecb(), NULL, key, iv, 1);
    EVP_CipherUpdate(ctx, encData, &outlen, inStr, inLen);
    encLen = outlen;
    EVP_CipherFinal(ctx, encData+outlen, &outlen);
    encLen += outlen;
    EVP_CIPHER_CTX_free(ctx);


    //解密
    int decLen = 0;
    outlen = 0;
    unsigned char decData[1024];
    EVP_CIPHER_CTX *ctx2= EVP_CIPHER_CTX_new();
    EVP_CipherInit_ex(ctx2, EVP_aes_256_ecb(), NULL, key, iv, 0);
    EVP_CipherUpdate(ctx2, decData, &outlen, encData, encLen);
    decLen = outlen;
    EVP_CipherFinal(ctx2, decData+outlen, &outlen);
    decLen += outlen;
    EVP_CIPHER_CTX_free(ctx2);

    decData[decLen] = '\0';
    printf("decrypt: %s\n",decData);
}

```

EVP_对底层接口进行了封装,但对初学者和对密码体系了解不到位的同学来说会比直接调用低级接口抽象许多

在调用细节上加强对资源销毁(EVP_FREE)的重视