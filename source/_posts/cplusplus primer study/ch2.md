---
title: c++ primer ch2
categories:
- c++
tags:
- c++
- vscode
- cmake
---
<meta name="referrer" content="no-referrer"/>

### 内容导航

本节主要包括c++的一些基础知识

<!--more-->

### 基本内置类型

<img src="C:\data\mysoftware\Typora\typoraPicture\image-20250710170042239.png" alt="image-20250710170042239" style="zoom:45%;" />

> 类型int、short、long和long long都是带符号的，通过在这些类型名前添加unsigned就可以得到无符号类型，例如unsigned long。类型unsigned int可以缩写为unsigned。

> 与其他整型不同，字符型被分为了三种：char、signed char和unsigned char。特别需要注意的是：类型char和类型signed char并不一样。尽管字符型有三种，但是字符的表现形式却只有两种：带符号的和无符号的。类型char实际上会表现为上述两种形式中的一种，具体是哪种由编译器决定。
>
> 无符号类型中所有比特都用来存储值，例如，8比特的unsigned char可以表示0至255区间内的值。
>
> C++标准并没有规定带符号类型应如何表示，但是约定了在表示范围内正值和负值的量应该平衡。因此，8比特的signed char理论上应该可以表示-127至127区间内的值，大多数现代计算机将实际的表示范围定为-128至127。

当从无符号数中减去一个值时，不管这个值是不是无符号数，我们都必须确保结果不能是一个负值：

~~~c++
unsigned u = 10, u2 = 42;
std::cout << u2 - u << std::endl;   // 32
std::cout << u - u2 << std::endl;   // 4294967264，取模2^32
~~~

### 转义序列

<img src="C:\data\mysoftware\Typora\typoraPicture\image-20250711102633758.png" alt="image-20250711102633758" style="zoom: 50%;" />

### 变量声明和定义的关系

为了支持分离式编译，C++语言将声明和定义区分开来。

- 声明(declaration)使得名字为程序所知，一个文件如果想使用别处定义的名字则必须包含对那个名字的声明。
- 而定义(definition)负责创建与名字关联的实体。

如果想声明一个变量而非定义它，就在变量名前添加关键字extern,而且不要显式地初始化变量：

~~~c++
extern int i;//声明i而非定义i
int j;//声明并定义j
~~~

extern语句如果包含初始值就不再是声明，而变成定义了：

~~~c++
extern double pi = 3.14159;
~~~

在函数体内部，如果试图初始化一个由extern关键字标记的变量，将引发错误。

> 变量能且只能被定义一次，但是可以被多次声明。

### 变量命名规范

- 标识符要能体现实际含义。
- 变量名一般用小写字母，如index,不要使用Index或INDEX。
- 用户自定义的类名一般以大写字母开头，如`Sales_item`。
- 如果标识符由多个单词组成，则单词间应有明显区分，如`student_loan`或`studentLoan`,不要使用`studentloan`。

### 引用

引用并非对象，相反的，它只是为一个已经存在的对象所起的另外一个名字

为引用赋值，实际上是把值赋给了与引用绑定的对象。获取引用的值，实际上是获取了与引用绑定的对象的值。同理，以引用作为初始值，实际上是以与引用绑定的对象作为初始值：

:warning: 因为引用本身不是一个对象，所以不能定义引用的引用。

### 指针

指针(pointer)是“指向(point to)”另外一种类型的复合类型。与引用类似，指针也实现了对其他对象的间接访问。然而指针与引用相比又有很多不同点。

其一，指针本身就是一个对象，允许对指针赋值和拷贝，而且在指针的生命周期内它可以先后指向几个不同的对象。
其二，指针无须在定义时赋初值。和其他内置类型一样，在块作用域内定义的指针如果没有被初始化，也将拥有一个不确定的值。

#### 获取对象的地址

指针存放某个对象的地址，要想获取该地址，需要使用取地址符(操作符&)：

~~~c++
int a =10;
int *b = &a;
int *c= b;
cout<<b<<endl; // prints the address of a
cout<<*c<<endl; // prints the value of a
return 0;

0x7fffffffd814
10
~~~

~~~c++
int main()
{
   int i=42;
   int &r  =i;
   int *p;
   p = &i; // p points to i
   *p = i;
   cout<<*p << endl; // prints 42
   int &r2 = *p; // r2 is a reference to the value pointed to by p
   r2 =6;
   cout<<r2 << endl; // prints 6
   cout<<i << endl; // prints 6
}
~~~

### void*指针

`void*`是一种特殊的指针类型，可用于存放任意对象的地址。一个`void*`指针存放着一个地址，这一点和其他指针类似。不同的是，我们对该地址中到底是个什么类型的对象并不了解：

利用`void*`指针能做的事儿比较有限：**拿它和别的指针比较、作为函数的输入或输出，或者赋给另外一个`void*`指针**。

不能直接操作`void*`指针所指的对象，因为我们并不知道这个对象到底是什么类型，也就无法确定能在这个对象上做哪些操作。

~~~c++
double obj = 3.14, *pd = &obj; // 正确：void*能存放任意类型对象的地址
void *pv = &obj;               // obj可以是任意类型的对象
pv = pd;                       // pv可以存放任意类型的指针
~~~

### 双指针

~~~c++
int main()
{
    int ival = 1024;
    int *p = &ival;
    int **pp = &p;
    cout << "ival: " << ival << endl;
    cout << "&ival: " << &ival << endl;
    cout << "p: " << p << endl;
    cout << "*p: " << *p << endl;
    cout << "pp: " << pp << endl;
    cout << "*pp: " << *pp << endl;
    cout << "**pp: " << **pp << endl;

    return 0;
}
~~~

~~~c++
ival: 1024
&ival: 0x7fffffffd814
p: 0x7fffffffd814
*p: 1024
pp: 0x7fffffffd818
*pp: 0x7fffffffd814
**pp: 1024
~~~

<img src="C:\data\mysoftware\Typora\typoraPicture\image-20250711150720155.png" alt="image-20250711150720155" style="zoom:50%;" />

### 指针的引用

面对一条比较复杂的指针或引用的声明语句时，从右向左阅读有助于弄清楚它的真实含义。

~~~c++
int i =42;
int *p;
int *&r = p; // r is a reference to pointer p
r = &i; // r now points to i
*r = 0;
cout << "i: " << i << endl; // Output: i: 0
~~~

比如`int *&r = p`中r先是一个引用，然后是一个指针的引用，最后是一个整型指针的引用。

### const限定符

将变量修饰为一个不可修改的常量

~~~c++
const int bufsize = 512;
~~~

因为const对象一旦创建后其值就不能再改变，**所以const对象必须初始化**。

如果想在多个文件之间共享const对象，必须在变量的定义之前添加`extern`关键字。

~~~c++
// fi1e1.cc定义并初始化了一个常量，该常量能被其他文件访问
const int bufsize = fcn();
// fi1e1.h头文件
extern const int bufsize; // 与file1.cc中定义的bufSize是同一个
~~~

#### constexpr变量

C++11新标准规定，允许将变量声明为constexpr类型以便由编译器来验证变量的值是否是一个常量表达式。声明为constexpr的变量一定是一个常量，而且必须用常量表达式初始化：

新标准允许定义一种特殊的constexpr函数。这种函数应该足够简单以使得编译时就可以计算其结果，这样就能用constexpr函数去初始化constexpr变量了。

~~~c++
constexpr int sz = size（）;//只有当size是一个constexpr函数时，才是一条正确的声明语句
~~~

### 类型别名

类型别名(type alias)是一个名字，它是某种类型的同义词。使用类型别名有很多好处，它让复杂的类型名字变得简单明了、易于理解和使用，还有助于程序员清楚地知道使用该类型的真实目的。

~~~c++
typedef double base,*p;//base是double的同义词，p是double*的同义词
~~~

新标准规定了一种新的方法，使用别名声明(alias declaration)来定义类型的别名：

```c++
using SI = Sales_item;
//SI是Sales_item的同义词
```

### auto类型说明符

C++11新标准引入了auto类型说明符，用它就能让编译器替我们去分析表达式所属的类型。和原来那些只对应一种特定类型的说明符(比如double)不同，**auto让编译器通过初始值来推算变量的类型**。显然，auto定义的变量必须有初始值：

使用auto也能在一条语句中声明多个变量。**因为一条声明语句只能有一个基本数据类型**，所以该语句中所有变量的初始基本数据类型都必须一样：

~~~c++
auto i=0,*p =&i;
//正确：i是整数、p是整型指针
auto sz=0,p1=3.14;
//错误：sz和pi的类型不一致
~~~

### decltype类型指示符

有时会遇到这种情况：希望从表达式的类型推断出要定义的变量的类型，但是不想用该表达式的值初始化变量。为了满足这一要求，C++11新标准引入了第二种类型说明符`decltype`,它的作用是选择并返回操作数的数据类型。在此过程中，编译器分析表达式并得到它的类型，却不实际计算表达式的值：

~~~c++
const int ci = 0, &cj = ci;
decltype(ci) x = 1; // x is an const int
decltype(cj) y = x; // y is an const int&
cout << "x: " << x << ", y: " << y << endl;
~~~

~~~c++
// decltype的结果可以是引用类型
int i = 42, *p = &i, &r = i;
decltype(r + 0) b;// 正确：加法的结果是int,因此b是一个（未初始化的)int
decltype(*p) c;// 错误：c是int&,必须初始化
~~~

切记：decltype((variable))（注意是双层括号）的结果永远是引用，而decltype(variable)结果只有当variable本身就是一个引用时才是NG引用。

### 预处理器概述

`#define`指令把一个名字设定为预处理变量，另外两个指令则分别检查某个指定的预处理变量是否己经定义：`#ifdef`当且仅当变量已定义时为真，`#ifndef`当且仅当变量未定义时为真。一旦检查结果为真，则执行后续操作直至遇到`#endif`指令为止。

~~~c++
#ifndef SALES_DATA_H
#define SALES_DATA_H
#include <iostream>
struct Sales_data
{
    std::string bookNo;
    unsigned units_sold = 0;
    double revenue = 0.0;
};

#endif
~~~

