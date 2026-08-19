---
title: string类的用法
published: 2026-05-11
# description: 
# image: ./cover.jpg
tags: [C++]
category: C++
draft: false
# slug: how-to-use-firefly
---
## 引言
### 什么是字符串？
字符串是由一系列字符组成的序列，用于表示文本信息。它在编程中被广泛应用于用户交互、文件处理、数据解析等场景。

### C 风格字符串 vs `std::string`
在 C++ 中，有两种主要的字符串类型：

- C 风格字符串（C-strings）：基于字符数组，以空字符 ('\0') 结尾。
- C++ `std::string` 类：更高级、功能更丰富的字符串类，封装了字符串操作的复杂性。
**C 风格字符串示例：**
```c++
char cstr[] = "Hello, World!";
```

**std::string 示例：**
```c++
#include <string>
std::string str = "Hello, World!";
```
## `std::string` 基础
**定义与初始化**
`std::string` 是 `C++` 标准库中的一个类，位于 `<string>` 头文件中。它封装了字符序列，并提供了丰富的成员函数用于操作字符串。

初始化有很多中方式：
| 代码                    | 说明                                      |
| --------------------- | --------------------------------------- |
| `string s1`           | 默认初始化，s1 是一个空串                          |
| `string s2(s1)`       | s2 是 s1 的副本                             |
| `string s2 = s1`      | 等价于 `s2(s1)`，s2 是 s1 的副本                |
| `string s3("value")`  | s3 是字面值 `"value"` 的副本，除了字面值最后的那个空字符外    |
| `string s3 = "value"` | 等价于 `s3("value")`，s3 是字面值 `"value"` 的副本 |
| `string s4(n, 'c')`   | 把 s4 初始化为由连续 n 个字符 c 组成的串               |
**包含头文件：**
```c++
#include <string>
```
**初始化示例：**
```c++
#include <iostream>
#include <string>

int main() {
    // 默认构造函数
    std::string str1;

    // 使用字符串字面值初始化
    std::string str2 = "Hello";

    // 使用拷贝构造函数
    std::string str3(str2);

    // 使用部分初始化
    std::string str4(str2, 0, 3); // "Hel"

    // 使用重复字符初始化
    std::string str5(5, 'A'); // "AAAAA"

    std::cout << "str1: " << str1 << std::endl;
    std::cout << "str2: " << str2 << std::endl;
    std::cout << "str3: " << str3 << std::endl;
    std::cout << "str4: " << str4 << std::endl;
    std::cout << "str5: " << str5 << std::endl;

    return 0;
}
```
**输出：**
```
str1:
str2: Hello
str3: Hello
str4: Hel
str5: AAAAA
```
### 字符串输入与输出
**输出字符串：**
```c++
#include <iostream>
#include <string>

int main() {
    std::string greeting = "Hello, C++ Strings!";
    std::cout << greeting << std::endl;
    return 0;
}
```
**从用户输入字符串：**
```c++
#include <iostream>
#include <string>

int main() {
    std::string input;
    std::cout << "请输入一个字符串：";
    std::cin >> input; // 读取直到第一个空白字符
    std::cout << "您输入的字符串是：" << input << std::endl;
    return 0;
}
```
**读取包含空格的整行字符串：**
```c++
#include <iostream>
#include <string>

int main() {
    std::string line;
    std::cout << "请输入一行文本：";
    std::getline(std::cin, line);
    std::cout << "您输入的文本是：" << line << std::endl;
    return 0;
}
```
## 字符串操作
常用的字符串操作如下：
| 操作               | 说明                                                         |
| ---------------- | ---------------------------------------------------------- |
| `os<<s`          | 将 `s` 写到输出流 `os` 当中，返回 `os`                                |
| `is>>s`          | 从 `is` 中读取字符串赋给 `s`，字符串以空白分隔，返回 `is`                       |
| `getline(is, s)` | 从 `is` 中读取一行赋给 `s`，返回 `is`                                 |
| `s.empty()`      | `s` 为空返回 `true`，否则返回 `false`                               |
| `s.size()`       | 返回 `s` 中字符的个数                                              |
| `s[n]`           | 返回 `s` 中第 `n` 个字符的引用，位置 `n` 从 0 计起                         |
| `s1+s2`          | 返回 `s1` 和 `s2` 连接后的结果                                      |
| `s1=s2`          | 用 `s2` 的副本代替 `s1` 中原来的字符                                   |
| `s1==s2`         | 如果 `s1` 和 `s2` 中所含的字符完全一样，则它们相等；`string` 对象的相等性判断对字母的大小写敏感 |
| `s1!=s2`         | （同上）                                                       |
| `<, <=, >, >=`   | 利用字符在字典中的顺序进行比较，且对字母的大小写敏感                                 |

### 拼接与连接
**使用 + 运算符：**
```c++
#include <iostream>
#include <string>

int main() {
    std::string first = "Hello, ";
    std::string second = "World!";
    std::string combined = first + second;
    std::cout << combined << std::endl; // 输出: Hello, World!
    return 0;
}
```
**使用 `append()` 函数：**
```c++
#include <iostream>
#include <string>

int main() {
    std::string str = "Hello";
    str.append(", World!");
    std::cout << str << std::endl; // 输出: Hello, World!
    return 0;
}
```
**使用 `+=` 运算符：**
```c++
#include <iostream>
#include <string>

int main() {
    std::string str = "Data";
    str += " Structures";
    std::cout << str << std::endl; // 输出: Data Structures
    return 0;
}
```
### 比较字符串
关于字符串的比较，其实是逐个位置按照字符比较，计算机中字符存储的方式是ASCII码表，每个字符对应一个ASCII码值，比较字符就是比较ASCII码值的大小
|    二进制    | 十进制 | 十六进制 |    图形   |    二进制    | 十进制 | 十六进制 |  图形 |    二进制    | 十进制 | 十六进制 |  图形 |   |
| :-------: | :-: | :--: | :-----: | :-------: | :-: | :--: | :-: | :-------: | :-: | :--: | :-: | - |
| 0010 0000 |  32 |  20  | (space) | 0100 0000 |  64 |  40  |  @  | 0110 0000 |  96 |  60  |  \` |   |
| 0010 0001 |  33 |  21  |    !    | 0100 0001 |  65 |  41  |  A  | 0110 0001 |  97 |  61  |  a  |   |
| 0010 0010 |  34 |  22  |    "    | 0100 0010 |  66 |  42  |  B  | 0110 0010 |  98 |  62  |  b  |   |
| 0010 0011 |  35 |  23  |    #    | 0100 0011 |  67 |  43  |  C  | 0110 0011 |  99 |  63  |  c  |   |
| 0010 0100 |  36 |  24  |    \$   | 0100 0100 |  68 |  44  |  D  | 0110 0100 | 100 |  64  |  d  |   |
| 0010 0101 |  37 |  25  |    %    | 0100 0101 |  69 |  45  |  E  | 0110 0101 | 101 |  65  |  e  |   |
| 0010 0110 |  38 |  26  |    &    | 0100 0110 |  70 |  46  |  F  | 0110 0110 | 102 |  66  |  f  |   |
| 0010 0111 |  39 |  27  |    '    | 0100 0111 |  71 |  47  |  G  | 0110 0111 | 103 |  67  |  g  |   |
| 0010 1000 |  40 |  28  |    (    | 0100 1000 |  72 |  48  |  H  | 0110 1000 | 104 |  68  |  h  |   |
| 0010 1001 |  41 |  29  |    )    | 0100 1001 |  73 |  49  |  I  | 0110 1001 | 105 |  69  |  i  |   |
| 0010 1010 |  42 |  2A  |    \*   | 0100 1010 |  74 |  4A  |  J  | 0110 1010 | 106 |  6A  |  j  |   |
| 0010 1011 |  43 |  2B  |    +    | 0100 1011 |  75 |  4B  |  K  | 0110 1011 | 107 |  6B  |  k  |   |
| 0010 1100 |  44 |  2C  |    ,    | 0100 1100 |  76 |  4C  |  L  | 0110 1100 | 108 |  6C  |  l  |   |
| 0010 1101 |  45 |  2D  |    -    | 0100 1101 |  77 |  4D  |  M  | 0110 1101 | 109 |  6D  |  m  |   |
| 0010 1110 |  46 |  2E  |    .    | 0100 1110 |  78 |  4E  |  N  | 0110 1110 | 110 |  6E  |  n  |   |
| 0010 1111 |  47 |  2F  |    /    | 0100 1111 |  79 |  4F  |  O  | 0110 1111 | 111 |  6F  |  o  |   |
| 0011 0000 |  48 |  30  |    0    | 0101 0000 |  80 |  50  |  P  | 0111 0000 | 112 |  70  |  p  |   |
| 0011 0001 |  49 |  31  |    1    | 0101 0001 |  81 |  51  |  Q  | 0111 0001 | 113 |  71  |  q  |   |
| 0011 0010 |  50 |  32  |    2    | 0101 0010 |  82 |  52  |  R  | 0111 0010 | 114 |  72  |  r  |   |
| 0011 0011 |  51 |  33  |    3    | 0101 0011 |  83 |  53  |  S  | 0111 0011 | 115 |  73  |  s  |   |
| 0011 0100 |  52 |  34  |    4    | 0101 0100 |  84 |  54  |  T  | 0111 0100 | 116 |  74  |  t  |   |
| 0011 0101 |  53 |  35  |    5    | 0101 0101 |  85 |  55  |  U  | 0111 0101 | 117 |  75  |  u  |   |
| 0011 0110 |  54 |  36  |    6    | 0101 0110 |  86 |  56  |  V  | 0111 0110 | 118 |  76  |  v  |   |
| 0011 0111 |  55 |  37  |    7    | 0101 0111 |  87 |  57  |  W  | 0111 0111 | 119 |  77  |  w  |   |
| 0011 1000 |  56 |  38  |    8    | 0101 1000 |  88 |  58  |  X  | 0111 1000 | 120 |  78  |  x  |   |
| 0011 1001 |  57 |  39  |    9    | 0101 1001 |  89 |  59  |  Y  | 0111 1001 | 121 |  79  |  y  |   |
| 0011 1010 |  58 |  3A  |    :    | 0101 1010 |  90 |  5A  |  Z  | 0111 1010 | 122 |  7A  |  z  |   |
| 0011 1011 |  59 |  3B  |    ;    | 0101 1011 |  91 |  5B  |  \[ | 0111 1011 | 123 |  7B  |  {  |   |
| 0011 1100 |  60 |  3C  |    <    | 0101 1100 |  92 |  5C  |  \\ | 0111 1100 | 124 |  7C  |  \| |   |
| 0011 1101 |  61 |  3D  |    =    | 0101 1101 |  93 |  5D  |  ]  | 0111 1101 | 125 |  7D  |  }  |   |
| 0011 1110 |  62 |  3E  |    >    | 0101 1110 |  94 |  5E  |  ^  | 0111 1110 | 126 |  7E  |  ~  |   |
| 0011 1111 |  63 |  3F  |    ?    | 0101 1111 |  95 |  5F  |  \_ |           |     |      |     |   |

一些控制字符也是通过ASCII码存储的
|    二进制    | 十进制 | 十六进制 |  缩写 | Unicode 表示法 | 脱出字符表示法 | 名称 / 意义              |
| :-------: | :-: | :--: | :-: | :---------: | :-----: | :------------------- |
| 0000 0000 |  0  |  00  | NUL |     NUL     |    ^@   | 空字符（Null）            |
| 0000 0001 |  1  |  01  | SOH |     SOH     |    ^A   | 标题开始                 |
| 0000 0010 |  2  |  02  | STX |     STX     |    ^B   | 本文开始                 |
| 0000 0011 |  3  |  03  | ETX |     ETX     |    ^C   | 本文结束                 |
| 0000 0100 |  4  |  04  | EOT |     EOT     |    ^D   | 传输结束                 |
| 0000 0101 |  5  |  05  | ENQ |     ENQ     |    ^E   | 请求                   |
| 0000 0110 |  6  |  06  | ACK |     ACK     |    ^F   | 确认回应                 |
| 0000 0111 |  7  |  07  | BEL |     BEL     |    ^G   | 响铃                   |
| 0000 1000 |  8  |  08  |  BS |      BS     |    ^H   | 退格                   |
| 0000 1001 |  9  |  09  |  HT |      HT     |    ^I   | 水平定位符号               |
| 0000 1010 |  10 |  0A  |  LF |      LF     |    ^J   | 换行键                  |
| 0000 1011 |  11 |  0B  |  VT |      VT     |    ^K   | 垂直定位符号               |
| 0000 1100 |  12 |  0C  |  FF |      FF     |    ^L   | 换页键                  |
| 0000 1101 |  13 |  0D  |  CR |      CR     |    ^M   | CR（字符）               |
| 0000 1110 |  14 |  0E  |  SO |      SO     |    ^N   | 取消变换（Shift out）      |
| 0000 1111 |  15 |  0F  |  SI |      SI     |    ^O   | 启用变换（Shift in）       |
| 0001 0000 |  16 |  10  | DLE |     DLE     |    ^P   | 跳出数据通讯               |
| 0001 0001 |  17 |  11  | DC1 |     DC1     |    ^Q   | 设备控制一（XON 激活软件速度控制）  |
| 0001 0010 |  18 |  12  | DC2 |     DC2     |    ^R   | 设备控制二                |
| 0001 0011 |  19 |  13  | DC3 |     DC3     |    ^S   | 设备控制三（XOFF 停用软件速度控制） |
| 0001 0100 |  20 |  14  | DC4 |     DC4     |    ^T   | 设备控制四                |
| 0001 0101 |  21 |  15  | NAK |     NAK     |    ^U   | 确认失败回应               |
| 0001 0110 |  22 |  16  | SYN |     SYN     |    ^V   | 同步用暂停                |
| 0001 0111 |  23 |  17  | ETB |     ETB     |    ^W   | 区块传输结束               |
| 0001 1000 |  24 |  18  | CAN |     CAN     |    ^X   | 取消                   |
| 0001 1001 |  25 |  19  |  EM |      EM     |    ^Y   | 连线介质中断               |
| 0001 1010 |  26 |  1A  | SUB |     SUB     |    ^Z   | 替换                   |
| 0001 1011 |  27 |  1B  | ESC |     ESC     |   ^\[   | 退出键                  |
| 0001 1100 |  28 |  1C  |  FS |      FS     |   ^\\   | 文件分割符                |
| 0001 1101 |  29 |  1D  |  GS |      GS     |    ^]   | 组群分隔符                |
| 0001 1110 |  30 |  1E  |  RS |      RS     |    ^^   | 记录分隔符                |
| 0001 1111 |  31 |  1F  |  US |      US     |   ^\_   | 单元分隔符                |
| 0111 1111 | 127 |  7F  | DEL |     DEL     |    ^?   | Delete 字符            |

使用 `==`, `!=`, `<`, `>`, `<=`, `>=` 运算符：
```c++
#include <iostream>
#include <string>

int main() {
    std::string a = "apple";
    std::string b = "banana";

    if (a == b) {
        std::cout << "a 和 b 相等" << std::endl;
    } else {
        std::cout << "a 和 b 不相等" << std::endl;
    }

    if (a < b) {
        std::cout << "a 在字典序中小于 b" << std::endl;
    } else {
        std::cout << "a 在字典序中不小于 b" << std::endl;
    }

    return 0;
}
```
**输出：**
```
a 和 b 不相等
a 在字典序中小于 b
```
### 查找与替换
使用 `find()` 查找子字符串：
```c++
#include <iostream>
#include <string>

int main() {
    std::string text = "The quick brown fox jumps over the lazy dog.";
    std::string word = "fox";

    size_t pos = text.find(word);
    if (pos != std::string::npos) {
        std::cout << "找到 '" << word << "' 在位置: " << pos << std::endl;
    } else {
        std::cout << "'" << word << "' 未找到。" << std::endl;
    }

    return 0;
}
```
替换子字符串：
```c++
#include <iostream>
#include <string>

int main() {
    std::string text = "I like cats.";
    std::string from = "cats";
    std::string to = "dogs";

    size_t pos = text.find(from);
    if (pos != std::string::npos) {
        text.replace(pos, from.length(), to);
        std::cout << "替换后: " << text << std::endl; // 输出: I like dogs.
    } else {
        std::cout << "'" << from << "' 未找到。" << std::endl;
    }

    return 0;
}
```
### 子字符串与切片
使用 `substr()` 获取子字符串：
```c++
#include <iostream>
#include <string>

int main() {
    std::string str = "Hello, World!";
    std::string sub = str.substr(7, 5); // 从位置7开始，长度5
    std::cout << sub << std::endl; // 输出: World
    return 0;
}
注意： 如果省略第二个参数，substr() 会返回从起始位置到字符串末尾的所有字符。
1
2
std::string sub = str.substr(7); // 从位置7开始直到结束
std::cout << sub << std::endl; // 输出: World!
```
**注意：** 如果省略第二个参数，substr() 会返回从起始位置到字符串末尾的所有字符。
```c++
std::string sub = str.substr(7); // 从位置7开始直到结束
std::cout << sub << std::endl; // 输出: World!
```
## 字符串的常用成员函数
### 长度与容量
获取字符串长度：
```c++
#include <iostream>
#include <string>

int main() {
    std::string str = "C++ Programming";
    std::cout << "字符串长度: " << str.length() << std::endl; // 输出: 14
    // 或者使用 size()
    std::cout << "字符串大小: " << str.size() << std::endl; // 输出: 14
    return 0;
}
```
获取字符串容量：

每个 `std::string` 对象都有一个容量`（capacity）`，表示它当前能够持有的最大字符数，而不需要重新分配内存。
```c++
#include <iostream>
#include <string>

int main() {
    std::string str = "Hello";
    std::cout << "初始容量: " << str.capacity() << std::endl;

    str += ", World!";
    std::cout << "追加后的容量: " << str.capacity() << std::endl;

    return 0;
}
```
输出示例：
```
初始容量: 15
追加后的容量: 15
```
注意： 容量可能因实现而异，并不保证它等于长度。
### 访问字符
对字符串中的字符操作，有如下方法, 切记需包含头文件
| 函数            | 说明                                                  |
| ------------- | --------------------------------------------------- |
| `isalnum(c)`  | 当 `c` 是字母或数字时为真                                     |
| `isalpha(c)`  | 当 `c` 是字母时为真                                        |
| `iscntrl(c)`  | 当 `c` 是控制字符时为真                                      |
| `isdigit(c)`  | 当 `c` 是数字时为真                                        |
| `isgraph(c)`  | 当 `c` 不是空格但可打印时为真                                   |
| `islower(c)`  | 当 `c` 是小写字母时为真                                      |
| `isprint(c)`  | 当 `c` 是可打印字符时为真（即 `c` 是空格或 `c` 具有可视形式）              |
| `ispunct(c)`  | 当 `c` 是标点符号时为真（即 `c` 不是控制字符、数字、字母、可打印空白中的一种）        |
| `isspace(c)`  | 当 `c` 是空白时为真（即 `c` 是空格、横向制表符、纵向制表符、回车符、换行符、进纸符中的一种） |
| `isupper(c)`  | 当 `c` 是大写字母时为真                                      |
| `isxdigit(c)` | 当 `c` 是十六进制数字时为真                                    |
| `tolower(c)`  | 如果 `c` 是大写字母，输出对应的小写字母；否则原样输出 `c`                   |
| `toupper(c)`  | 如果 `c` 是小写字母，输出对应的大写字母；否则原样输出 `c`                   |

使用索引访问单个字符：
```c++
#include <iostream>
#include <string>

int main() {
    std::string str = "ABCDE";

    // 正向索引
    for (size_t i = 0; i < str.length(); ++i) {
        std::cout << "字符 " << i << ": " << str[i] << std::endl;
    }

    //反向遍历
    for(int i = str.length() - 1; i >= 0 ; i --){
        std::cout << "下标为 " << i << "的字符为" << str[i] << std::endl;
    }

    return 0;
}
```
使用 `at()` 函数（包含边界检查）：
```c++
#include <iostream>
#include <string>

int main() {
    std::string str = "ABCDE";
    try {
        char c = str.at(10); // 超出范围，会抛出异常
    } catch (const std::out_of_range& e) {
        std::cout << "异常捕获: " << e.what() << std::endl;
    }
    return 0;
}
```
输出：
```
异常捕获: basic_string::at: __n (which is 10) >= this->size() (which is 5)
```
### 转换大小写
C++ 标准库中的 `std::toupper` 和 `std::tolower` 可以用于转换字符的大小写。结合 `std::transform`，可以实现整个字符串的大小写转换。

转换为大写：
```c++
#include <iostream>
#include <string>
#include <algorithm>
#include <cctype>

int main() {
    std::string str = "Hello, World!";
    std::transform(str.begin(), str.end(), str.begin(), 
                   [](unsigned char c) { return std::toupper(c); });
    std::cout << str << std::endl; // 输出: HELLO, WORLD!
    return 0;
}
```
转换为小写：
```c++
#include <iostream>
#include <string>
#include <algorithm>
#include <cctype>

int main() {
    std::string str = "Hello, World!";
    std::transform(str.begin(), str.end(), str.begin(), 
                   [](unsigned char c) { return std::tolower(c); });
    std::cout << str << std::endl; // 输出: hello, world!
    return 0;
}
```
### 其他有用的函数
`empty()`：检查字符串是否为空。
```c++
std::string str;
if (str.empty()) {
    std::cout << "字符串为空。" << std::endl;
}
```
`clear()`：清空字符串内容。
```c++
std::string str = "Clear me!";
str.clear();
std::cout << "str: " << str << std::endl; // 输出为空
```
`erase()：`删除字符串的部分内容。
```c++
std::string str = "Hello, World!";
str.erase(5, 7); // 从位置5开始，删除7个字符
std::cout << str << std::endl; // 输出: Hello!
```
`insert()：`在指定位置插入字符串或字符。
```c++
std::string str = "Hello World";
str.insert(5, ",");
std::cout << str << std::endl; // 输出: Hello, World
```
`replace()：`替换字符串的部分内容（前面已示例）。

`find_first_of()`, `find_last_of()`：查找字符集合中的任何一个字符。
```c++
std::string str = "apple, banana, cherry";
size_t pos = str.find_first_of(", ");
std::cout << "第一个逗号或空格的位置: " << pos << std::endl; // 输出: 5
```
## 高级用法
### 字符串流（stringstream）
`std::stringstream` 是 C++ 标准库中第 `<sstream>` 头文件提供的一个类，用于在内存中进行字符串的读写操作，类似于文件流。
基本用法示例：
```c++
#include <iostream>
#include <sstream>
#include <string>

int main() {
    std::stringstream ss;
    ss << "Value: " << 42 << ", " << 3.14;

    std::string result = ss.str();
    std::cout << result << std::endl; // 输出: Value: 42, 3.14

    return 0;
}
```
从字符串流中读取数据：
```c++
#include <iostream>
#include <sstream>
#include <string>

int main() {
    std::string data = "123 45.67 Hello";
    std::stringstream ss(data);

    int a;
    double b;
    std::string c;

    ss >> a >> b >> c;

    std::cout << "a: " << a << ", b: " << b << ", c: " << c << std::endl;
    // 输出: a: 123, b: 45.67, c: Hello

    return 0;
}
```
### 字符串与其他数据类型的转换
将其他类型转换为 `std::string`：
- 使用 `std::to_string()`：
```c++
#include <iostream>
#include <string>

int main() {
    int num = 100;
    double pi = 3.14159;

    std::string str1 = std::to_string(num);
    std::string str2 = std::to_string(pi);

    std::cout << "str1: " << str1 << ", str2: " << str2 << std::endl;
    // 输出: str1: 100, str2: 3.141590
    return 0;
}
```
将 `std::string` 转换为其他类型：
- 使用字符串流：
```c++
#include <iostream>
#include <sstream>
#include <string>

int main() {
    std::string numStr = "256";
    std::string piStr = "3.14";

    int num;
    double pi;

    std::stringstream ss1(numStr);
    ss1 >> num;

    std::stringstream ss2(piStr);
    ss2 >> pi;

    std::cout << "num: " << num << ", pi: " << pi << std::endl;
    // 输出: num: 256, pi: 3.14
    return 0;
}
```
- 使用 `std::stoi()`, `std::stod()` 等函数（C++11 及以上）：
```c++
#include <iostream>
#include <string>

int main() {
    std::string numStr = "256";
    std::string piStr = "3.14";

    int num = std::stoi(numStr);
    double pi = std::stod(piStr);

    std::cout << "num: " << num << ", pi: " << pi << std::endl;
    // 输出: num: 256, pi: 3.14
    return 0;
}
```
### 正则表达式与字符串匹配
C++ 标准库提供了 `<regex>` 头文件，用于支持正则表达式。

关于正则表达式的规则可以参考菜鸟教程文档https://www.runoob.com/regexp/regexp-syntax.html

基本用法示例：
```c++
#include <iostream>
#include <string>
#include <regex>

int main() {
    std::string text = "The quick brown fox jumps over the lazy dog.";
    std::regex pattern(R"(\b\w{5}\b)"); // 匹配所有5个字母的单词

    std::sregex_iterator it(text.begin(), text.end(), pattern);
    std::sregex_iterator end;

    std::cout << "5个字母的单词有:" << std::endl;
    while (it != end) {
        std::cout << (*it).str() << std::endl;
        ++it;
    }

    return 0;
}
```
输出：
```
5个字母的单词有:
quick
brown
jumps
leazy
```
说明：

- `\b` 匹配单词边界。
- `\w{5}` 匹配恰好5个字母的单词。
**注意：** 使用原始字符串字面值`（R"()"）`以简化正则表达式的编写。
字符串与 C 风格字符串的转换
## 从 C 风格字符串转换为 std::string
通过 `std::string` 的构造函数，可以轻松将 C 风格字符串转换为 `std::string`。
```c++
#include <iostream>
#include <string>

int main() {
    const char* cstr = "Hello, C-strings!";
    std::string str(cstr);
    std::cout << str << std::endl; // 输出: Hello, C-strings!
    return 0;
}
```
### 从 std::string 转换为 C 风格字符串
使用 `c_str()` 成员函数，可以获取 C 风格字符串指针。
```c++
#include <iostream>
#include <string>

int main() {
    std::string str = "Hello, std::string!";
    const char* cstr = str.c_str();
    std::cout << cstr << std::endl; // 输出: Hello, std::string!
    return 0;
}
```
**注意：** 返回的指针是只读的，且指向的内存由 `std::string` 管理，确保在 `std::string` 对象有效期间使用。

