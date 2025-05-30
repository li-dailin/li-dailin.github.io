---
title: 'C++ Primer (5th Edition) 读书笔记（更新中）'
date: 2025-05-13
permalink: /blogs/2025/C++Primer(5thEdition)读书笔记
excerpt: 阅读 C++ Primer，更加深入地了解 C++ 语言。
tags:
  - C/C++
  - 笔记
---

# 第一章 开始

1. 在命令行中，执行完一个程序后，可以通过 `echo` 命令获得 `main` 函数的返回值：
    - UNIX：`echo $?`
    - Windows：`echo %ERRORLEVEL%`

2. 输入运算符 `>>` 从左侧的 `istream` 读入数据，并存入右侧的对象中。输入运算符的计算结果是左侧的 `istream` 对象。当遇到文件结束符 `EOF`，或遇到一个无效输入时，`istream` 对象的状态会变成无效。

3. 输出运算符 `<<` 将右侧的值写到左侧的 `ostream` 对象中。输出运算符的计算结果是左侧的 `ostream` 对象。

4. 操纵符 `std::endl` 不仅输出一个换行符，还会**强制刷新输出缓冲区**（将与设备关联的缓冲区中的内容刷入设备中）。因此，在高性能场景中建议使用 `\n`，减少刷新开销。

# 第二章 变量和基本类型

1. 通过添加前缀和后缀，可以改变整型、浮点型和字符型字面值的默认类型。
     -  字符和字符串字面值的前缀
        
         | 前缀 | 含义                          | 类型       |
         | ---- | ----------------------------- | ---------- |
         | `u`  | Unicode 16 字符               | `char16_t` |
         | `U`  | Unicode 32 字符               | `char32_t` |
         | `L`  | 宽字符                        | `wchar_t`  |
         | `u8` | UTF-8（仅用于字符串字面常量） | `char`     |
         
     - 整型字面值的后缀
       
         | 后缀         | 最小匹配类型 |
         | ------------ | ------------ |
         | `u` 或 `U`   | `unsigned`   |
         | `l` 或 `L`   | `long`       |
         | `ll` 或 `LL` | `long long`  |
         
     - 浮点型字面值的后缀
       
         | 后缀       | 最小匹配类型  |
         | ---------- | ------------- |
         | `f` 或 `F` | `float`       |
         | `l` 或 `L` | `long double` |

2. C++11 允许用花括号来初始化变量，称为**列表初始化**。如果使用列表初始化时，初始值存在丢失信息的风险，编译器将报错。例如，下面四条语句都能完成初始化：

   ```c++
   int a = 0;
   int a = {0};
   int a{0};
   int a(0);
   ```

3. 定义于函数体内的内置类型的对象如果没有初始化，则其值未定义。类的对象如果没有显式地初始化，则其值由类确定。

4. 为了支持**分离式编译**，C++ 将声明和定义区分开来。**声明**使得名字为程序所知，一个文件如果想使用别处定义的名字则必须包含对那个名字的声明。如果想声明一个变量而非定义它，就在变量名前添加关键字 **`extern`**，而且不要显式地初始化变量：`extern int i;`。**任何包含了显式初始化的声明即成为定义。** 

5. **引用必须被初始化。**定义了一个引用之后，对其进行的所有操作都是在与之绑定的对象上进行的。**引用本身不是一个对象**，所以不能定义引用的引用。引用的类型要和与之绑定的对象严格匹配（对 const 的引用和基类的引用除外）。引用只能绑定在对象上，而不能与字面值或某个表达式的计算结果绑定在一起。

6. **指针本身就是一个对象，且无须在定义时赋初值。**指针的值（即地址）应属下列 4 种状态之一： 
   - 指向一个对象。
   - 指向紧邻对象所占空间的下一个位置（如 `int *p = &ival + 1;`，这种指针禁止解引用，可用于标识容器的结束位置）。
   - 空指针，意味着指针没有指向任何对象。
   - 无效指针，也就是上述情况之外的其他值。

7. 生成空指针的方法：
     ```c++
     int *p1 = nullptr;
     int *p2 = 0;
     ```

8. **`void*`** 是一种特殊的指针类型，可用于存放任意对象的地址。**不能直接操作 `void*` 指针所指的对象**，因为我们并不知道这个对象到底是什么类型。

9. 对指针的引用：`int *&r = p;`

10. const 对象一旦创建后其值就不能再改变，所以 **const 对象必须初始化**。当以编译时初始化的方式定义一个 const 对象时，编译器将在编译过程中把用到该变量的地方都替换成对应的值。

11. 默认情况下，const 对象被设定为仅在文件内有效。建议只在一个文件中定义 const，而在其他多个文件中声明并使用它。如果想在多个文件之间共享 const 对象，不管是声明还是**定义**都要添加 `extern` 关键字：

      ```c++
      extern const int bufSize = 1000;  // 定义
      extern const int bufSize;         // 声明
      ```

12. 对 const 的引用（简称常量引用）不能被用作修改它所绑定的对象，并且可能引用一个并非 const 的对象。例如，`const int &r = i;` 表示**不允许通过 `r` 修改 `i` 的值**。尽管如此，**`i` 的值仍然允许通过其他途径修改**。

13. 在初始化常量引用时允许用任意表达式作为初始值，只要该表达式的结果能转换成引用的类型即可。在这种情况下，引用绑定了一个**临时量**对象。  

14. 注意区分指向常量的指针和常量指针：
      - **指向常量的指针**（pointer to const）：`const ElemType *p;`
        要想存放常量对象的地址，只能使用指向常量的指针。注意，允许令一个指向常量的指针指向一个非常量对象，所谓指向常量的指针仅仅要求**不能通过该指针改变对象的值**。
      - **常量指针**（const pointer）：`ElemType *const p;`
        常量指针必须初始化，而且一旦初始化完成，则**指针的值**（也就是存放在指针中的那个地址）**不能再改变**。

15. 用**顶层 const** 表示某个对象本身是个常量，用**底层 const** 表示指针所指的对象是一个常量。指针类型既可以是顶层 const 也可以是底层 const。

      ```c++
      const int *const p = i;	// 靠右的 const 是顶层 const, 靠左的 const 是底层 const
      ```

16. 当执行对象的拷贝操作时，拷入和拷出的对象必须具有相同的底层 const 资格，或者两个对象的数据类型必须能够转换。一般来说，非常量可以转换成常量，反之则不行。

17. 常量表达式是指值不会改变并且在**编译**过程就能得到计算结果的表达式。用常量表达式初始化的 const 对象也是常量表达式。C++11 规定，声明为 `constexpr` 的变量一定是一个常量表达式，而且必须用常量表达式初始化。也可以用 `constexpr` 函数去初始化 `constexpr` 变量，这种函数应该足够简单以使得编译时就可以计算其结果。声明 `constexpr` 时用到的类型称为**字面值类型**。算术类型、引用、指针和枚举都属于字面值类型。

18. `constexpr` 指针的初始值必须是 `nullptr` 或者 0，或者是存储于某个固定地址中的对象。限定符 `constexpr` 仅对指针有效：

      ```c++
      constexpr int *q = nullptr; 	// q 是一个指向整数的**常量指针**
      constexpr const int *p = &i; 	// p 是**常量指针**，指向整型常量 i
      ```

19. 除了 `typedef` 外，C++11 还可以使用**别名声明**来定义类型的别名，别名声明由关键字 `using` 开始：

      ```c++
      using ElemType = int;
      ```

20. 尝试把类型别名替换成它的本名以理解该语句的含义是错误的行为。例如：

      ```c++
      typedef char *pstring;
      const pstring p = 0;	// p 是指向 char 的**常量指针**
      ```

21. C++11 引入了 `auto` 类型说明符，让编译器通过初始值来推算变量的类型。使用 `auto` 也能在一条语句中声明多个变量，但是该语句中**所有变量的初始基本数据类型都必须一样**。

22. `auto` 一般会忽略顶层 const 而保留底层 const。如果希望推断出的 `auto` 类型是一个顶层 const，需要明确指出（即使用 `const auto`）。

23. C++11 还引入了 `decltype` 类型说明符，它的作用是选择并返回操作数的数据类型。在此过程中，编译器分析表达式并得到它的类型，却不实际计算表达式的值。
      - 如果 `decltype` 使用的表达式是一个变量，则 `decltype` 返回该变量的类型（包括顶层 const 和引用在内）。
      - 如果 `decltype` 使用的表达式不是一个变量，则 `decltype` 返回表达式结果对应的类型。
        - 有些表达式将向 `decltype` 返回一个**引用**类型。一般来说当这种情况发生时，意味着该表达式的结果对象能作为**一条赋值语句的左值**。
        -  如果表达式的内容是**解引用**操作，则 `decltype` 将得到**引用**类型。
      - 如果给变量加上了一层或多层括号，编译器就会把它当成是一个表达式，`decltype` 会得到**引用**类型。

24. `decltype` 和 `auto` 的区别：
      - `auto` 会丢弃引用和顶层 const（除非显式声明为 `auto &` 或 `const auto`）。
      - `decltype` 严格保留表达式的类型信息。

25. C++11 规定，可以为数据成员提供一个**类内初始值**。创建对象时，类内初始值将用于初始化数据成员。没有初始值的成员将被默认初始化。类内初始值或者放在花括号里，或者放在等号右边，**不能使用圆括号**。

26. 确保头文件多次包含仍能安全工作的常用技术是**预处理器**。**头文件保护符**确保头文件的内容只被编译一次。
    - `#define` 指令把一个名字设定为预处理变量。
    - `#ifdef` 和 `#ifndef` 指令分别检查某个指定的预处理变量是否己经定义：
      - `#ifdef` 当且仅当变量已定义时为真；
      - `#ifndef` 当且仅当变量未定义时为真。
    - 一旦检查结果为真，则执行后续操作直至遇到 `#endif` 指令为止。

    头文件保护符的一般语法（3 行）：

      ```c++
      #ifndef HEADER_FILE_H
      #define HEADER_FILE_H
      // 头文件内容
      #endif
      ```

    上述语句也可以用更现代的 **`#pragma once`** 代替。

# 第三章 字符串、向量和数组

1. `using` 声明具有如下的形式：`using namespace::name;`，从而可以无须专门的前缀直接访问命名空间中的名字。每个 `using` 声明引入命名空间中的一个成员。注意，头文件不应包含 `using` 声明。

2. 如果使用等号（=）初始化一个变量，实际上执行的是**拷贝初始化**。如果不使用等号，则执行的是**直接初始化**。

      ```c++
      string s3("value");     // 直接初始化，s3 是字面值 "value" 的副本
      string s4(n, 'c');      // 直接初始化，把 s4 初始化为由连续 n 个字符 c 组成的串
      string s3 = "value";    // 拷贝初始化，等价于 s3("value")
      ```

3. `getline(cin, line)` 得到的 `string` 对象中并不包含最后的换行符。

4. 所有用于存放 `string` 类的 `size` 函数返回值的变量，都应该是 **`string::size_type`** 类型的。它是一个**无符号**类型的值（注意尽量避免混用 `int` 和 `unsigned`），而且能足够存放下任何 `string` 对象的大小。**下标**运算符 ([ ]) 接收的输入参数也是 `string::size_type` 类型的值。

5. 当把 `string` 对象和字符字面值及字符串字面值混在一条语句中使用时，必须确保每个加法运算符（+）的两侧的运算对象至少有一个是 `string`。注意，**不能把字面值直接相加**，因为 C++ 中的字符串字面值并不是标准库类型 `string` 的对象，而是 `const char[N]` 类型。

6. 如果想要改变 `string` 对象中字符的值，必须把循环变量定义成引用类型，如 `for (auto &c : s)`。

7. `vector` 是一个**类模板**。编译器根据模板创建类或函数的过程称为**实例化**。需要通过提供一些额外信息来指定模板到底实例化成什么样的类，提供信息的方式总是这样：即在模板名字后面跟一对**尖括号**，在括号内放上信息。