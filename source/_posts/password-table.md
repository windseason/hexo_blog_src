---
title: 对抗反编译之使用密码表来保护代码中的字符串
tags: iOS, android
categories: iOS, android
date: 2018-07-13 12:30:24
---


## 对抗反编译之使用密码表来保护代码中的字符串

在最近几个月的时间内，由于工作的关系，一直在对抗互联网的黑色产业，就是传说中的众多刷机工作室。他们拥有数量众多的越狱设备，只要有利可图，就会对目标ipa进行破解来达到获利的目的。

众所周知，苹果设备和app的安全性要比隔壁安卓高出许多，但这个是在有苹果爸爸设计的一系列安全保护措施下面，也就是在非越狱环境下。一旦app跑在越狱的环境下，并且开发者没有做任何保护措施的话，那就相当于裸奔了。

安全这个问题很大，我仅分享给大家工作中我学到的经验，之后我会一点一点更新我的blog，如果有更多时间，还有开源一些代码到我的github上。今天先来谈谈程序中字符串的安全问题。

### 一般程序中对敏感字符串处理的例子

一般程序员因为工期赶或者没有安全性上的顾虑，对app中敏感的字符串，尤其是跟自家服务器通信的私钥啦，身份验证key啦，都是用一个全局字符串常量来保存，就跟下面代码一样：

#### secret.h 文件

```objective-c
extern NSString * const secretKey;
```

#### secret.m 文件

```objective-c
NSString * const secretKey = @"nzPHmNrbjuVOO7dX";
```

这样做的结果是，图谋不轨的人只要利用反编译工具就能够轻而易举获得app中任意敏感信息，对公司利益造成巨大损失。有不少反编译工具可以使用，例如[Hopper Disassembler](https://www.jianshu.com/p/10873c5c1e08), [IDA Pro](https://www.hex-rays.com/products/ida/)等工具。

### 如何有效防止字符串泄露敏感信息

办法肯定有不少，我这里分享我们自己在用的办法。

1. 基本想法就是预先配置好一个字符表
2. 然后将需要隐藏的敏感信息字符串一个字符一个字符从表里查到出相应的位置记下来
3. 程序使用的时候跟第2步的位置组成的字符数组，把字符串还原出来

#### 代码分享

> 代码在iOS和android两个平台都能够使用

##### 生成密码表

假设我们现在预先配置的字符表为`ABCD123456`

先通过这个网站[http://www.online-toolz.com/tools/text-hex-convertor.php](http://www.online-toolz.com/tools/text-hex-convertor.php)获得这个字符串的hex code `41424344313233343536`，然后在程序中声明一个数组`hex_table`来存储它（见下面的代码）

```c
//PasswordTable.c

//source string: ABCD123456
static const uint8_t hex_table[TABLE_SIZE] = {
    0x41,0x42,0x43,0x44,0x31,0x32,0x33,0x34,0x35,0x36 };
```

##### 查询并输出字符在密码表中的位置

使用下面的代码可以在`console`看到对应的数组，将它copy下来，例如我们这里传入'BCD456'后生成对应的int数组`len = 6 { 1, 2, 3, 7, 8, 9, }`，把这个数组记下来。
```c
//PasswordTable.c

/**
 * lookup magic numbers in terms of the given str based 
 * on magic table.
 */
void lookupMagics(const char * str, size_t str_len)
{
    printf("len = %li {", str_len);
    for (int j = 0; j < str_len; j++) {
        for (int i = 0; i < TABLE_SIZE; i++) {
            if (hex_table[i] == str[j]) {
                printf(" %i,", i);
                break;
            }
        }
    }
    printf(" }\n");
}
```

##### 从密码表生成原字符串

```c
//PasswordTable.h

#define TABLE_SIZE 10

/**
 Convert integer array to string based on built-in hex table.

 @param array       integer array.
 @param arr_len     length of the array.
 @param buffer      string buffer for receiving result, make sure it was alloced.
 @param buffer_len  length of the buffer.
 
 @return            0 success, -1 if buffer_len is less than arr_len.
 */
int convertMagics2String(int * array, size_t arr_len, char * buffer, size_t buffer_len);
```

```c
//PasswordTable.c

//source string: ABCD123456
static const uint8_t hex_table[TABLE_SIZE] = {
    0x41,0x42,0x43,0x44,0x31,0x32,0x33,0x34,0x35,0x36 };

int convertMagics2String(int * array, size_t arr_len, char * buffer, size_t buffer_len)
{
    if (buffer_len <= arr_len) {
        return -1;
    }
    memset((void *)buffer, 0, arr_len + 1);
    for (int i = 0; i < arr_len; i++) {
        buffer[i] = hex_table[array[i]];
    }
    return 0;
}

//main.c
#include "PasswordTable.h"

int main() {
    int magics[] = {1, 2, 3, 7, 8, 9};//之前由调用lookupMagics生成的整形数组
    char str[7];
    convertMagics2String(magics, 6, str, 7);
    printf("the source string is %s\n", str); //将会打印: the source string is BCD456
    return 0;
}
```

在上面代码中main函数的例子就是你在实际代码中的样子。这样做了以后，反编译工具就很难快速容易地通过字符串获得关键信息了，除非对方对你的代码抱有极大的兴趣，愿意花时间分析。

最后再多说一点，为了进一步加强安全性，建议对以上c函数进行混淆，将进一步加大反编译分析的难度。
