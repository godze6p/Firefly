---
title: 标准IO
published: 2026-05-13
# description: 
# image: ./cover.jpg
tags: [Linux,IO]
category: Linux
draft: false
# slug: how-to-use-firefly
---
## 标准IO流的打开和关闭
### 流的打开和关闭
**打开流**

标准IO中提供了fopen函数，用于打开标准IO流，原型如下：

```cpp
FILE* fopen
(
    //被打开的文件名称
    char const* _FileName,
    //打开模式
    char const* _Mode
);

```

p打开成功，返回流指针（FILE结构体指针）；出错返回NULL

**打开模式**

mode参数支持模式如下

| 模式(mode) | 含义 | 文件是否存在的要求 |
| --- | --- | --- |
| “r”或“rb” | 只读打开 | 文件必须存在 |
| “r+”或“r+b” | 可读可写打开 | 文件必须存在，从文件起始位置写入 |
| “w”或“wb” | 仅写打开 | 不存在则创建；存在会清空原有内容 |
| “w+”或“w+b” | 可读可写打开 | 不存在则创建；存在会清空原有内容 |
| “a”或“ab” | 仅写打开 | 不存在则创建；如已存在且有内容，在文件末尾追加新内容 |
| “a+”或“a+b” | 可读可写打开 | 不存在则创建；如已存在且有内容，写入会在文件末尾追加新内容 |


p当给定”b”参数时，表示二进制方式打开（仅Windows有效，Linux忽略b参数不区分是否是b）

**流的关闭**

标准IO中提供了fclose函数，用于打开标准IO流，原型如下:

```cpp
int fclose(FILE* _Stream);
```

传入的参数为被关闭的流指针（FILE结构体指针）；

+ 关闭成功返回整数0；关闭失败返回EOF（整数-1），并设置errno
+ 关闭后自动刷新缓冲区数据并释放其空间
+ 程序正常停止后，所有打开的流都会被关闭，流关闭后不可再操作

**示例代码**

创建文件

```cpp
#include <stdio.h>

int main(void) {
    FILE *fp = fopen("test.txt", "w");
    fclose(fp);
    return 0;
}
```



以读的方式打开文件, 并且通过fgets每次获取一行

```cpp
#include <stdio.h>

int main(){
    FILE * fp = fopen("test.txt", "r");
    char line[1024] = "";
    while(fgets(line, 1024, fp) != NULL){
        printf("%s",line);
    }

    fclose(fp);
    return 0;
}
```



### 处理错误信息
在打开流的过程中，有可能出现错误，标准IO提供了3个可操作对象供处理错误信息。

+ extern int errno; 错误号
+ void perror(const char *s); 打印错误信息，输出用户提供字符串s和当前错误
+ char * strerror(int errno); 根据错误号，返回错误信息字符串



+ 使用error错误号，需要包含 error.h 头文件
+ 使用perror函数，需要包含 stdio.h 头文件
+ 使用strerror函数，需要包含 string.h 头文件



示例代码1，基于perror函数处理错误信息：

```cpp
#include <stdio.h>


int main(void)
{
FILE * fp= fopen("test.txt", "r");// 此文件不存在

if (fp == NULL)
{
perror("错误信息是：");
return -1;
}

fclose(fp);
return 0;
}

```



示例代码2，基于strerror函数处理错误信息：

```cpp
#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <errno.h>
#include <string.h>

int main(void)
{
    FILE *fp = fopen("test.txt", "r"); // 此文件不存在
    if (fp == NULL)
    {
        printf("错误号是：%d\n", errno);
        printf("错误信息是：%s\n", strerror(errno));
        return -1;
    }
    fclose(fp);
    return 0;
}

```

**strerror_s函数**

为了避免strerror的C4996警告，可以使用strerror_s函数，需包含 string.h 头文件

```cpp
int strerror_s(
    //存放错误信息字符串
    char*  _Buffer,
    //大小（bytes），和字符串数组长度一致即可
    unsigned long long _SizeInBytes,
    //错误号
    int    _ErrorNumber
);

```

注意strerror_s不是c标准库的内容，需要编译器支持c11标准, 所以我们在代码之前打开c11标准扩展, 需提前定义`#define __STDC_WANT_LIB_EXT1__ 1`,

在使用的时候通过宏`__STDC_LIB_EXT1__`判断，我们的编译器是否支持c11标准扩展

```cpp
#define __STDC_WANT_LIB_EXT1__ 1
#include <locale.h>
#include <stdio.h>
#include <errno.h>
#include <string.h>

int main(void)
{
    FILE *fp = fopen("test.txt", "r"); // 此文件不存在
    if (fp == NULL)
    {
        printf("错误号是：%d\n", errno);
        char err_msg[1024] = "";
#ifdef __STDC_LIB_EXT1__
        strerror_s(err_msg, 1024, errno);
        printf("错误信息是：%s", err_msg);
#endif
        return -1;
    }
    fclose(fp);
    return 0;
}

```

编译时使用`gcc -std=c11 ./main.cpp `
