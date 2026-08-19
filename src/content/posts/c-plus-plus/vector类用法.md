---
title: vector类的用法
published: 2026-05-11
# description: 
# image: ./cover.jpg
tags: [C++]
category: C++
draft: false
# slug: how-to-use-firefly
---

### 1. 引言
#### 什么是向量（Vector）？
向量（**Vector**）是 C++ 标准模板库（STL）中的一种序列容器，能够动态地管理可变大小的数组。与传统的固定大小的数组不同，向量可以根据需要随时调整其大小，提供更高的灵活性和便利性。



#### 向量与数组的比较
| 特性 | 数组（Array） | 向量（Vector） |
| --- | --- | --- |
| 大小 | 固定大小（编译时或运行时） | 动态可变大小 |
| 内存管理 | 手动管理（需要预留足够空间） | 自动管理（自动扩展或收缩） |
| 支持的操作 | 限制较多 | 丰富的成员函数和操作 |
| 安全性 | 较低（易发生缓冲区溢出） | 较高（通过成员函数进行边界检查） |
| 与 STL 算法的兼容性 | 低 | 高 |


---

### 2. `std::vector` 基础
#### 2.1 包含头文件
使用 `std::vector` 需要包含 `<vector>` 头文件：



```cpp
#include <vector>
```



#### 2.2 定义与初始化
**定义一个整数向量：**



```cpp
std::vector<int> numbers;
```



**定义一个字符串向量：**



```cpp
#include <vector>
#include <string>

std::vector<std::string> words;
```



**初始化向量：**



+ **默认初始化：**

```cpp
std::vector<int> vec1; // 空向量
```

+ **指定大小和默认值：**

```cpp
std::vector<int> vec2(5, 10); // 5个元素，值均为10
```

+ **使用初始化列表：**

```cpp
std::vector<int> vec3 = {1, 2, 3, 4, 5};
```

+ **拷贝构造：**

```cpp
std::vector<int> vec4(vec3); // 复制vec3
```

+ **移动构造：**

```cpp
std::vector<int> vec5(std::move(vec4)); // 移动vec4到vec5
```



**示例代码：**



```cpp
#include <iostream>
#include <vector>

int main() {
    // 默认初始化
    std::vector<int> vec1;
    
    // 指定大小和默认值
    std::vector<int> vec2(5, 10);
    
    // 使用初始化列表
    std::vector<int> vec3 = {1, 2, 3, 4, 5};
    
    // 拷贝构造
    std::vector<int> vec4(vec3);
    
    // 移动构造
    std::vector<int> vec5(std::move(vec4));
    
    // 输出vec2
    std::cout << "vec2: ";
    for(auto num : vec2) {
        std::cout << num << " ";
    }
    std::cout << std::endl;
    
    // 输出vec3
    std::cout << "vec3: ";
    for(auto num : vec3) {
        std::cout << num << " ";
    }
    std::cout << std::endl;
    
    // 输出vec5
    std::cout << "vec5: ";
    for(auto num : vec5) {
        std::cout << num << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```



**输出：**



```makefile
vec2: 10 10 10 10 10 
vec3: 1 2 3 4 5 
vec5: 1 2 3 4 5 
```



#### 2.3 向量的大小与容量
+ `size()`：返回向量中元素的数量。
+ `capacity()`：返回向量目前为止分配的存储容量。
+ `empty()`：检查向量是否为空。



**示例代码：**



```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec = {1, 2, 3};
    
    std::cout << "Size: " << vec.size() << std::endl;       // 输出: 3
    std::cout << "Capacity: " << vec.capacity() << std::endl; // 输出: 3（或更大，取决于实现）
    
    std::cout << "Is empty? " << (vec.empty() ? "Yes" : "No") << std::endl; // 输出: No
    
    vec.reserve(10); // 预留容量
    std::cout << "After reserve(10), Capacity: " << vec.capacity() << std::endl; // 输出: 10
    
    vec.shrink_to_fit(); // 收缩到适合大小
    std::cout << "After shrink_to_fit(), Capacity: " << vec.capacity() << std::endl; // 输出: 3
    
    return 0;
}
```



**输出示例：**



```yaml
Size: 3
Capacity: 3
Is empty? No
After reserve(10), Capacity: 10
After shrink_to_fit(), Capacity: 3
```



**注意：** `capacity()` 并不一定精确匹配 `size()`，它表示在需要重新分配内存之前，向量可以容纳的元素数量。



---

### 3. 向量的基本操作
#### 3.1 添加与删除元素
+ `push_back()`：在向量末尾添加一个元素。
+ `pop_back()`：移除向量末尾的元素。
+ `insert()`：在指定位置插入元素。
+ `erase()`：移除指定位置的元素或范围内的元素。
+ `clear()`：移除所有元素。



**示例代码：**



```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec;
    
    // 使用push_back添加元素
    vec.push_back(10);
    vec.push_back(20);
    vec.push_back(30);
    
    std::cout << "After push_back: ";
    for(auto num : vec) {
        std::cout << num << " ";
    }
    std::cout << std::endl; // 输出: 10 20 30 
    
    // 使用pop_back移除最后一个元素
    vec.pop_back();
    
    std::cout << "After pop_back: ";
    for(auto num : vec) {
        std::cout << num << " ";
    }
    std::cout << std::endl; // 输出: 10 20 
    
    // 在第二个位置插入25
    vec.insert(vec.begin() + 1, 25);
    
    std::cout << "After insert: ";
    for(auto num : vec) {
        std::cout << num << " ";
    }
    std::cout << std::endl; // 输出: 10 25 20 
    
    // 删除第二个元素（25）
    vec.erase(vec.begin() + 1);
    
    std::cout << "After erase: ";
    for(auto num : vec) {
        std::cout << num << " ";
    }
    std::cout << std::endl; // 输出: 10 20 
    
    // 清空向量
    vec.clear();
    std::cout << "After clear, size: " << vec.size() << std::endl; // 输出: 0
    
    return 0;
}
```



**输出：**



```yaml
After push_back: 10 20 30 
After pop_back: 10 20 
After insert: 10 25 20 
After erase: 10 20 
After clear, size: 0
```



#### 3.2 访问元素
+ `operator[]`：通过索引访问元素。
+ `at()`：通过索引访问元素，带边界检查。
+ `front()`：访问第一个元素。
+ `back()`：访问最后一个元素。



**示例代码：**



```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<std::string> fruits = {"Apple", "Banana", "Cherry"};
    
    // 使用operator[]访问元素
    std::cout << "First fruit: " << fruits[0] << std::endl; // 输出: Apple
    
    // 使用at()访问元素
    try {
        std::cout << "Second fruit: " << fruits.at(1) << std::endl; // 输出: Banana
        std::cout << "Invalid fruit: " << fruits.at(5) << std::endl; // 抛出异常
    }
    catch(const std::out_of_range& e) {
        std::cerr << "Exception: " << e.what() << std::endl;
    }
    
    // 使用front()和back()
    std::cout << "Front: " << fruits.front() << std::endl; // 输出: Apple
    std::cout << "Back: " << fruits.back() << std::endl;   // 输出: Cherry
    
    return 0;
}
```



**输出：**



```vbnet
First fruit: Apple
Second fruit: Banana
Exception: vector::_M_range_check: __n (which is 5) >= this->size() (which is 3)
Front: Apple
Back: Cherry
```



#### 3.3 遍历向量
+ **使用范围 **`for`** 循环**
+ **使用传统 **`for`** 循环**
+ **使用迭代器**



**示例代码：**



```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> numbers = {1, 2, 3, 4, 5};
    
    // 使用范围 for 循环
    std::cout << "Using range-based for loop: ";
    for(auto num : numbers) {
        std::cout << num << " ";
    }
    std::cout << std::endl;
    
    // 使用传统 for 循环
    std::cout << "Using traditional for loop: ";
    for(size_t i = 0; i < numbers.size(); ++i) {
        std::cout << numbers[i] << " ";
    }
    std::cout << std::endl;
    
    // 使用迭代器
    std::cout << "Using iterators: ";
    for(auto it = numbers.begin(); it != numbers.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```



**输出：**



```vbnet
Using range-based for loop: 1 2 3 4 5 
Using traditional for loop: 1 2 3 4 5 
Using iterators: 1 2 3 4 5 
```



#### 3.4 修改元素
+ **通过索引或迭代器修改**
+ **使用 **`assign()`** 重新赋值**
+ **替换整个向量内容**



**示例代码：**



```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec = {10, 20, 30, 40, 50};
    
    // 通过索引修改元素
    vec[2] = 35;
    
    // 使用 at() 修改元素
    vec.at(4) = 55;
    
    // 使用迭代器修改元素
    for(auto it = vec.begin(); it != vec.end(); ++it) {
        if(*it == 20) {
            *it = 25;
        }
    }
    
    // 输出修改后的向量
    std::cout << "Modified vector: ";
    for(auto num : vec) {
        std::cout << num << " ";
    }
    std::cout << std::endl; // 输出: 10 25 35 40 55 
    
    return 0;
}
```



**输出：**



```cpp
Modified vector: 10 25 35 40 55 
```



---

### 4. 向量的高级用法
#### 4.1 嵌套向量（二维向量）
向量可以包含其他向量，形成多维数组结构。



**示例代码：二维向量**



```cpp
#include <iostream>
#include <vector>

int main() {
    // 定义一个3x4的二维向量，初始化为0
    std::vector<std::vector<int>> matrix(3, std::vector<int>(4, 0));
    
    // 填充矩阵
    for(int i = 0; i < 3; ++i) {
        for(int j = 0; j < 4; ++j) {
            matrix[i][j] = i * 4 + j + 1;
        }
    }
    
    // 输出矩阵
    std::cout << "Matrix:" << std::endl;
    for(auto row : matrix) {
        for(auto elem : row) {
            std::cout << elem << "\t";
        }
        std::cout << std::endl;
    }
    
    return 0;
}
```



**输出：**



```makefile
Matrix:
1	2	3	4	
5	6	7	8	
9	10	11	12	
```



#### 4.2 向量与其他数据结构结合
向量可以与结构体、类等其他数据结构结合使用，增强数据组织能力。



**示例代码：向量与结构体结合**



```cpp
#include <iostream>
#include <vector>
#include <string>

// 定义学生结构体
struct Student {
    int id;
    std::string name;
    float grade;
};

int main() {
    // 定义一个学生向量
    std::vector<Student> students;
    
    // 添加学生
    students.push_back({1001, "Alice", 89.5});
    students.push_back({1002, "Bob", 92.0});
    students.push_back({1003, "Charlie", 85.0});
    
    // 遍历并输出学生信息
    for(const auto& student : students) {
        std::cout << "ID: " << student.id 
                  << ", Name: " << student.name 
                  << ", Grade: " << student.grade << std::endl;
    }
    
    return 0;
}
```



**输出：**



```yaml
ID: 1001, Name: Alice, Grade: 89.5
ID: 1002, Name: Bob, Grade: 92
ID: 1003, Name: Charlie, Grade: 85
```



#### 4.3 使用迭代器操作向量
迭代器是一种指针类型，用于遍历和操作容器中的元素。



**示例代码：使用迭代器**



```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec = {10, 20, 30, 40, 50};
    
    // 使用迭代器遍历并修改元素
    for(auto it = vec.begin(); it != vec.end(); ++it) {
        *it += 5;
    }
    
    // 输出修改后的向量
    std::cout << "After modifying: ";
    for(auto it = vec.begin(); it != vec.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl; // 输出: 15 25 35 45 55 
    
    return 0;
}
```



**输出：**



```yaml
After modifying: 15 25 35 45 55 
```



---

### 5. 常用算法与向量
#### 5.1 排序
可以使用 `<algorithm>` 头文件中的 `sort()` 函数对向量进行排序。



**示例代码：对整数向量排序**



```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> numbers = {50, 20, 40, 10, 30};
    
    // 排序前
    std::cout << "Before sorting: ";
    for(auto num : numbers) {
        std::cout << num << " ";
    }
    std::cout << std::endl;
    
    // 使用sort()排序
    std::sort(numbers.begin(), numbers.end());
    
    // 排序后
    std::cout << "After sorting: ";
    for(auto num : numbers) {
        std::cout << num << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```



**输出：**



```yaml
Before sorting: 50 20 40 10 30 
After sorting: 10 20 30 40 50 
```



**自定义排序规则：降序**



```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> numbers = {50, 20, 40, 10, 30};
    
    // 使用sort()并传入lambda表达式进行降序排序
    std::sort(numbers.begin(), numbers.end(), [](int a, int b) {
        return a > b;
    });
    
    // 输出排序后的向量
    std::cout << "After sorting in descending order: ";
    for(auto num : numbers) {
        std::cout << num << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```



**输出：**



```csharp
After sorting in descending order: 50 40 30 20 10 
```



#### 5.2 反转
使用 `reverse()` 函数可以反转向量中的元素顺序。



**示例代码：**



```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<char> letters = {'A', 'B', 'C', 'D', 'E'};
    
    std::cout << "Before reversing: ";
    for(auto c : letters) {
        std::cout << c << " ";
    }
    std::cout << std::endl;
    
    // 反转向量
    std::reverse(letters.begin(), letters.end());
    
    std::cout << "After reversing: ";
    for(auto c : letters) {
        std::cout << c << " ";
    }
    std::cout << std::endl;
    
    return 0;
}
```



**输出：**



```less
Before reversing: A B C D E 
After reversing: E D C B A 
```



#### 5.3 查找
使用 `find()` 函数可以在向量中查找特定元素。



**示例代码：**



```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<std::string> fruits = {"Apple", "Banana", "Cherry", "Date"};
    std::string target = "Cherry";
    
    // 使用find()查找元素
    auto it = std::find(fruits.begin(), fruits.end(), target);
    
    if(it != fruits.end()) {
        std::cout << target << " found at position " << std::distance(fruits.begin(), it) << std::endl;
    }
    else {
        std::cout << target << " not found." << std::endl;
    }
    
    return 0;
}
```



**输出：**



```css
Cherry found at position 2
```



---

### 6. 向量的性能与优化
#### 6.1 内存管理
向量会动态地管理内存，自动调整其容量以适应新增或删除的元素。频繁的内存分配可能会影响性能。



#### 6.2 预留空间
使用 `reserve()` 可以提前为向量分配足够的内存，减少内存重新分配的次数，提高性能。



**示例代码：**



```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec;
    
    // 预留空间
    vec.reserve(1000);
    std::cout << "Capacity after reserve(1000): " << vec.capacity() << std::endl;
    
    // 添加元素
    for(int i = 0; i < 1000; ++i) {
        vec.push_back(i);
    }
    
    std::cout << "Size after adding elements: " << vec.size() << std::endl;
    std::cout << "Capacity after adding elements: " << vec.capacity() << std::endl;
    
    return 0;
}
```



**输出示例：**



```yaml
Capacity after reserve(1000): 1000
Size after adding elements: 1000
Capacity after adding elements: 1000
```



#### 6.3 收缩容量
使用 `shrink_to_fit()` 可以请求收缩向量的容量以匹配其大小，释放多余的内存。



**示例代码：**



```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec;
    
    // 预留较大的空间
    vec.reserve(1000);
    std::cout << "Capacity before adding: " << vec.capacity() << std::endl;
    
    // 添加少量元素
    vec.push_back(1);
    vec.push_back(2);
    vec.push_back(3);
    std::cout << "Size after adding: " << vec.size() << std::endl;
    std::cout << "Capacity after adding: " << vec.capacity() << std::endl;
    
    // 收缩容量
    vec.shrink_to_fit();
    std::cout << "Capacity after shrink_to_fit: " << vec.capacity() << std::endl;
    
    return 0;
}
```



**输出示例：**



```yaml
Capacity before adding: 1000
Size after adding: 3
Capacity after adding: 1000
Capacity after shrink_to_fit: 3
```




