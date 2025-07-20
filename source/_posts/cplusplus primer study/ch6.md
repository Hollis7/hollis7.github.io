---
title: c++ primer ch6
categories:
- c++
tags:
- c++
- vscode
- cmake
---
<meta name="referrer" content="no-referrer"/>

### 内容导航

本节主要包括c++的函数知识

<!--more-->

### 局部静态对象

某些时候，有必要令局部变量的生命周期贯穿函数调用及之后的时间。可以将局部变量定义成static类型从而获得这样的对象。**局部静态对象（local static object）在程序的执行路径第一次经过对象定义语句时初始化，并且直到程序终止才被销毁，在此期间即使对象所在的函数结束执行也不会对它有影响。**

~~~c++
// 例子
int count_add(int n) // n是形参
{
    static int ctr = 0; // ctr 是局部静态变量
    ctr += n;
    return ctr;
}

int main()
{
    for (int i = 0; i != 10; ++i) // i 是局部变量
        cout << count_add(i)<<" ";

    return 0;
}
~~~

~~~c++
0 1 3 6 10 15 21 28 36 45
~~~

### 编译和链接多个源文件

#### Chapter6.h

~~~c++
int fact(int val);
int func();

template <typename T>
T abs(T i)
{
	return i >= 0 ? i : -i;
}
~~~

#### fact.cpp

~~~c++
#include "Chapter6.h"
#include <iostream>

int fact(int val)
{
	if (val == 0 || val == 1) return 1;
	else return val * fact(val - 1);
}

int func()
{
	int n, ret = 1;
	std::cout << "input a number: ";
	std::cin >> n;
	while (n > 1) ret *= n--;
	return ret;
}
~~~

#### factMain.cpp

~~~c++
#include "Chapter6.h"
#include <iostream>

int main()
{
	std::cout << "5! is " << fact(5) << std::endl;
	std::cout << func() << std::endl;
	std::cout << abs(-9.78) << std::endl;
}
~~~

#### cmaklists.txt

~~~cmake
# 修改1: 为 factMain 添加额外的源文件，其它保持不变
add_executable(factMain src/ch6/factMain.cpp src/ch6/fact.cpp)
target_include_directories(factMain PRIVATE ${MYLIB_INCLUDE_DIR})
target_compile_options(factMain PRIVATE -g -O0)
~~~

### 指针形参

指针的行为和其他非引用类型一样。当执行指针拷贝操作时，拷贝的是指针的值。拷贝之后，两个指针是不同的指针。因为指针使我们可以间接地访问它所指的对象，所以通过指针可以修改它所指对象的值：

```
void swap(int* lhs, int* rhs)
{
	int tmp;
	tmp = *lhs;
	*lhs = *rhs;
	*rhs = tmp;
}

int main()
{
	for (int lft, rht; std::cout << "Please Enter:\n", std::cin >> lft >> rht;)
	{
		swap(&lft, &rht);
		std::cout << lft << " " << rht << std::endl;
	}

	return 0;
}
```

### 使用引用避免拷贝

拷贝大的类类型对象或者容器对象比较低效，甚至有的类类型(包括IO类型在内)根本就不支持拷贝操作。当某种类型不支持拷贝操作时，函数只能通过引用形参访问该类型的对象。

~~~c++
bool isshorter(const string &s1, const string &s2)
{
    return s1.size() < s2.size();
}

int main()
{
    string s1 = "hello";
    string s2 = "world";
    if (!isshorter(s1, s2))
    {
        cout << s1 << endl;
    }
    else
    {
        cout << s2 << endl;
    }

    return 0;
}
~~~

#### 常量指针的区别

~~~c++
int i= 42;
int * const p1 = &i;
*p1 = 3;//yes
~~~

p1这是一个**常量指针**（const pointer），表示指针本身是常量，即指针的指向不可改变，但可以通过指针修改所指向的值。

~~~c++
int j = 34;
const int *p2 =&j;
*p2 = 4;//no
p2 = &i;//yes
~~~

p2这是一个**指向常量的指针**（pointer to const），表示指针指向的值是常量，不能通过该指针修改值，但指针本身可以指向其他变量。

#### 尽量使用常量引用

使用引用而非常量引用也会极大地限制函数所能接受的实参类型。就像刚刚看到的，我们不能把const对象、字面值或者需要类型转换的对象传递给普通的引用形参。

```
string::size_type find_char(const string &s,char c,string::size_type &occurs);
```

调用成功：

~~~c++
find_char("hello ", 'h', ctr);
~~~

调用失败：

~~~c++
string::size_type find_char(string &s,char c,string::size_type &occurs);
find_char("hello ", 'h', ctr);
~~~

#### 再举例

> 下面的这个函数虽然合法，但是不算特别有用。指出它的局限性并设法改善。

```cpp
bool is_empty(string& s) { return s.empty(); }
```

局限性在于**常量字符串**和**字符串字面值**无法作为该函数的实参，如果下面这样调用是非法的：

```cpp
const string str;
bool flag = is_empty(str); //非法
bool flag = is_empty("hello"); //非法
```

所以要将这个函数的形参定义为常量引用：

```cpp
bool is_empty(const string& s) { return s.empty(); }
```

### initializer_ list形参

有时我们无法提前预知应该向函数传递几个实参。例如，我们想要编写代码输出程序产生的错误信息，此时最好用同一个函数实现该项功能，以便对所有错误的处理能够整齐划一。然而，错误信息的种类不同，所以调用错误输出函数时传递的实参也各不相同。

<img src="C:\data\mysoftware\Typora\typoraPicture\image-20250714164206724.png" alt="image-20250714164206724" style="zoom:50%;" />

~~~c++
int sum(std::initializer_list<int> const& il)
{
	int sum = 0;
	for (auto i : il) 
		sum += i;
	return sum;
}

int main(void)
{
	auto il = { 1, 2, 3, 4, 5, 6, 7, 8, 9 };
	std::cout << sum(il) << std::endl;

	return 0;
}
~~~

### 重载函数

#### 定义重载函数

对于重载的函数来说，它们应该在形参数量或形参类型上有所不同。

~~~c++
//每对声明的是同一个函数
Recordlookup(const Account &acct);
Recordlookup(const Account&);//省略了形参的名字
~~~

#### const_cast和重载

~~~c++
const string &shorterString(const string &sl, const string &s2)
{
    return (sl.size() < s2.size()) ? sl : s2;
}
string &shorterString(string &sl, string &s2)
{
    auto &r = shorterString(const_cast<const string&>(sl),
                            const_cast<const string&>(s2));
    return const_cast<string &>(r);
}
~~~

这个函数的参数和返回类型都是const string的引用。我们可以对两个非常量的string实参调用这个函数，但返回的结果仍然是const string的引用。因此我们需要一种新的shorterstring函数，当它的实参不是常量时，得到的结果是一个普通的引用，使用const_cast可以做到这一点：

### 默认实参

~~~c++
typedef string::size_type sz;
string screen(sz ht = 24, sz wid = 80, char backgrnd =' ');
~~~

其中我们为每一个形参都提供了默认实参，默认实参作为形参的初始值出现在形参列表中。我们可以为一个或多个形参定义默认值，**不过需要注意的是，一旦某个形参被赋予了默认值，它后面的所有形参都必须有默认值。**

~~~c++
string window;
window = screen();
window = screen(66);
window = screen(24, 80);
window = screen(24, 80, '*');
~~~

函数调用时实参按其位置解析，默认实参负责填补函数调用缺少的尾部实参（靠右侧位置）。例如，要想覆盖backgrnd的默认值，必须为ht和wid提供实参

当设计含有默认实参的函数时，其中一项任务是合理设置形参的顺序**，尽量让不怎么使用默认值的形参出现在前面，而让那些经常使用默认值的形参出现在后面**。

### 内联函数

#### 内联函数可避免函数调用的开销

将函数指定为内联函数(inline)，通常就是将它在每个调用点上“内联地”展开。

~~~c++
inline const string &shorterString(const string &sl, const string &s2)
{
    return (sl.size() < s2.size()) ? sl : s2;
}
~~~

内联说明只是向编译器发出的一个请求，编译器可以选择忽略这个请求。

~~~c++
cout << shorterstring(sl, s2) << endl;
cout <<(sl.size（）< s2.size（） ? sl : s2)<< endl;
~~~

### constexpr函数

constexpr函数(constexpr function)是指能用于常量表达式的函数。定义constexpr函数的方法与其他函数类似，不过要遵循几项约定：**函数的返回类型及所有形参的类型都得是字面值类型，而且函数体中必须有且只有一条return语句**：

~~~c++
constexpr int max(int a, int b)
{
    return (a > b) ? a : b;
}
constexpr int new_sz(){
    return 42;
}
~~~

### NDEBUG

除了用于`assert`外，也可以使用NDEBUG编写自已的条件调试代码。如果NDEBUG未定义，将执行#ifndef和#endif之间的代码；如果定义了NDEBUG，这些代码将被忽略掉：

~~~c++
void print(const int ia[],size_t size)
{
#ifndef NDEBUG
    cerr<<__func__<<":array size = " << size << endl;
#endif
}
~~~

~~~
__FILE__存放文件名的字符串字面值。
__LINE__存放当前行号的整型字面值。
__TIME__存放文件编译时间的字符串字面值。
__DATE__存放文件编译日期的字符串字面值。
~~~

### 函数指针

函数指针指向的是函数而非对象。

我们还能直接使用指向函数的指针调用该函数**，无须提前解引用指针**：

~~~c++
bool lengthCompare(const string &s1, const string &s2){
    return (s1.size() < s2.size())? true : false;
}
int main(void)
{
    bool (*pf)(const string &, const string &) = lengthCompare;
    bool b1= pf("hello", "world");
    bool b2 = (*pf)("hello", "world");
    bool b3 =lengthCompare("hello", "world");
    cout << boolalpha << b1 << " " << b2 << " " << b3 << endl;
    
    return 0;
}
~~~

### 函数指针形参

和数组类似，虽然不能定义函数类型的形参，但是形参可以是指向函数的指针。此时，形参看起来是函数类型，实际上却是当成指针使用：

~~~c++
//第三个形参是函数类型，它会自动地转换成指向函数的指针
void useBigger(const string &sl, const string &s2,
    bool pf(const string&,const string &));
//等价的声明：显式地将形参定义成指向函数的指针  5c1111g&17
void useBigger(const string &sl, const string &s2,
bool (*pf)(const string&,const string&));
~~~

我们可以直接把函数作为实参使用，此时它会自动转换成指针：

~~~c++
 useBigger("hello", "world", lengthCompare);
~~~

