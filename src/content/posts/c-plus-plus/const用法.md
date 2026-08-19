---
title: const用法
published: 2026-05-07
# description: 
# image: ./cover.jpg
tags: [C++]
category: C++
draft: false
# slug: how-to-use-firefly
---
## `const`限定符
### 1 `const` 的定义与作用
const 是 C++ 关键字，用于指示变量的值不可修改。通过使用 const，可以提高代码的安全性与可读性，防止无意中修改变量的值。
### 2 `const` 在变量声明中的位置
`const` 关键字通常放在变量类型之前，例如：
```c++
const int a = 10;
```
也可以放在类型之后，但这种用法较少见：
```c++
int const a = 10;
```
可以用一个变量初始化常量， 也可以将一个常量赋值给一个变量
```c++
//可以用一个变量初始化常量
int i1 = 10;
const int i2 = i1;
//也可以将一个常量赋值给一个变量
int i3 = i2;
```
const变量必须初始化
```c++
//错误用法，const变量必须初始化
//const int i4;
```
### 3 编译器如何处理 const 修饰的变量
## 指针和const
### 指向常量的指针(pointer to const)
可以令指针指向常量或非常量。类似于常量引用，指向常量的指针（`pointer to const`）不能用于改变其所指对象的值。
要想存放常量对象的地址，只能使用指向常量的指针：
```c++
//PI 是一个常量,它的值不能改变
const double PI = 3.14;
//错误，ptr是一个普通指针
//double * ptr = &PI;
//正确,cptr可以指向一个双精度常量
const double *cptr = &PI;
//错误，不能给*ptr赋值
//*cptr = 3.14;
```
指针的类型必须与其所指对象的类型一致，但是允许令一个指向常量的指针指向一个非常量对象
```c++
//可以用指向常量的指针指向一个非常量
int i10 = 2048;
//ptr指向i10
int *cptr2 = &i10;
```
### **const**指针
指针是对象而引用不是，因此就像其他对象类型一样，允许把指针本身定为常量。
常量指针（`const pointer`）必须初始化，而且一旦初始化完成，则它的值（也就是存放在指针中的那个地址）就不能再改变了。
把＊放在const关键字之前用以说明指针是一个常量，这样的书写形式隐含着一层意味，即不变的是指针本身的值而非指向的那个值：
```c++
int errNumb = 0;
//curErr是一个常量指针，指向errNumb
int * const curErr = &errNumb;
const double pi2 = 3.14;
//pip 是一个指向常量对象的常量指针
const double *const pip = &pi2;
```
指针本身是一个常量并不意味着不能通过指针修改其所指对象的值，能否这样做完全依赖于所指对象的类型
```c++
//错误，pip是一个指向常量的指针
//*pip = 2.72;
//可以修改常量指针指向的内容
*curErr = 1024;
//可以修改常量指针指向的地址
//curErr = &i10;
```
## 顶层const
指针本身是一个对象，它又可以指向另外一个对象。因此，指针本身是不是常量以及指针所指的是不是一个常量就是两个相互独立的问题

用名词顶层`const`（`top-level const`）表示指针本身是个常量，而用名词底层`const`（`low-level const`）表示指针所指的对象是一个常量。
顶层`const`可以表示任意的对象是常量，这一点对任何数据类型都适用，如算术类型、类、指针等。
底层`const`则与指针和引用等复合类型的基本类型部分有关。比较特殊的是，指针类型既可以是顶层const也可以是底层const，这一点和其他类型相比区别明显：
```c++
int i = 0;
//不能改变p1的值，这是一个顶层const
int * const pi = &i;
//不能改变ci的值，这是一个顶层const
const int ci  = 42;
//允许改变p2的值，这是一个底层const
const int *  p2 = &ci;
//靠右边的const是顶层const，靠左边的const是底层const
const int * const p3 = p2;
//用于声明引用的const都是底层const
const int &r = ci;
```
底层`const`的限制却不能忽视。当执行对象的拷贝操作时，拷入和拷出的对象必须具有相同的底层const资格，或者两个对象的数据类型必须能够转换
```c++
//指针赋值要注意关注底层const
//p2拥有底层const,p4无底层const，所以无法赋值
//int * p4 = p2;
```
## constexpr和常量表达式
常量表达式（`const expression`）是指值不会改变并且在编译过程就能得到计算结果的表达式。显然，字面值属于常量表达式，用常量表达式初始化的const对象也是常量表达式。后面将会提到，C++语言中有几种情况下是要用到常量表达式的。

我们先在global.h中声明一个全局函数返回固定大小
```c++
extern int GetSize();
```
在global.cpp中实现
```c++
int GetSize(){
    return 20;
}
```
然后我们用const定义一些常量表达式

一个对象（或表达式）是不是常量表达式由它的数据类型和初始值共同决定，例如
```c++
{
    //max_files是一个常量表达式
    const int max_files = 20;
    //limit是一个常量表达式
    const int limit = max_files + 10;
    //staff_size不是常量表达式,无const声明
    int staff_size = 20;
    //sz不是常量表达式,运行时计算才得知
    const int sz = GetSize();
}
```
尽管`staff_size`的初始值是个字面值常量，但由于它的数据类型只是一个普通`int`而非`const int`，所以它不属于常量表达式。

另一方面，尽管`sz`本身是一个常量，但它的具体值直到运行时才能获取到，所以也不是常量表达式。

在一个复杂系统中，很难（几乎肯定不能）分辨一个初始值到底是不是常量表达式。

当然可以定义一个`const`变量并把它的初始值设为我们认为的某个常量表达式，但在实际使用时，尽管要求如此却常常发现初始值并非常量表达式的情况。

**C++11新标准**

C++11新标准规定，允许将变量声明为`constexpr`类型以便由编译器来验证变量的值是否是一个常量表达式。声明为`constexpr`的变量一定是一个常量，而且必须用常量表达式初始化：
```c++
//20是一个常量表达式
constexpr int mf = 20;
//mf+1是一个常量表达式
constexpr int limit = mf + 10;
//错误，GetSize()不是一个常量表达式，需要运行才能返回
//constexpr int sz = GetSize();
```
尽管不能使用普通函数作为`constexpr`变量的初始值，新标准允许定义一种特殊的`constexpr`函数。
这种函数应该足够简单以使得编译时就可以计算其结果，这样就能用`constexpr`函数去初始化constexpr变量了。
我们在global.h中定义一个`constexpr`函数
```c++
inline constexpr int GetSizeConst() {
    return 1;
}
```
为了避免在多个源文件中包含同一个头文件而导致的多重定义错误，可以将 `constexpr` 函数声明为 `inline`。

`inline` 关键字允许在多个翻译单元中定义同一个函数，而不会引起链接错误。

接下来在定义一个`constexpr`变量就行了
```c++
constexpr int sz = GetSizeConst();
```
**指针和`constexpr`**
必须明确一点，在`constexpr`声明中如果定义了一个指针，限定符`constexpr`仅对指针有效，与指针所指的对象无关：
```c++
//p是一个指向整形常量的指针
const int * p = nullptr;
//q是一个指向整数的常量指针
constexpr int *q = nullptr;
```
一个`constexpr`指针的初始值必须是`nullptr`或者`0`，或者是存储于某个固定地址中的对象。

函数体内定义的变量一般来说并非存放在固定地址中，因此`constexpr`指针不能指向这样的变量。

定义于所有函数体之外的对象其地址固定不变，能用来初始化`constexpr`指针

global_i是一个全局变量
```c++
//constexpr指针只能绑定固定地址
//constexpr int *p = &mvalue;
constexpr int *p = nullptr;
//可以绑定全局变量，全局变量地址固定
constexpr  int *cp = &global_i;
```
可以修改constexpr指向的内容
```c++
constexpr int *p = &global_i;
//修改p指向的内容数据
*p = 1024;
```