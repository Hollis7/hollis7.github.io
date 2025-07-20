---
title: c++ primer ch7
categories:
- c++
tags:
- c++
- vscode
- cmake
---
<meta name="referrer" content="no-referrer"/>

### 内容导航

本节主要包括c++的类知识

<!--more-->

### 优化Sales_data

~~~c++
struct Sales_data
{
    // 新成员：关于Sales_data对象的操作
    std::string isbn() { return bookNo; }
    Sales_data &combine(const Sales_data &);
    double avg_price() const;
    std::string bookNo;
    unsigned units_sold = 0;
    double revenue = 0.0;
};

Sales_data add(const Sales_data&,const Sales_data&);
std::ostream &print(std::ostream&, const Sales_data&);
std::istream &read(std::istream&, Sales_data&);
~~~

其中`double avg_price() const;`中的`const`是一个 **成员函数的常量限定符**，它表示这个成员函数是一个 **常量成员函数**。保证函数不会修改对象的数据成员。

在 `print` 和 `read` 函数声明中，`std::ostream &` 和 `std::istream &` 前面的 `&` 表示这两个函数 **返回的是引用（reference）**，而不是值拷贝。

- `std::ostream` 和 `std::istream` 是流对象，它们通常较大（可能包含缓冲区、状态标志等），直接返回它们的值会导致拷贝开销。
- 返回引用（`&`）效率更高，因为它只是传递一个别名（指针），而不是整个对象。
- C++ 标准库的 `<<` 和 `>>` 运算符也返回流引用，与标准库风格一致
- 支持链式调用

### 在类的外部定义成员函数

~~~c++
double Sales_data::avg_price() const{
    if (units_sold) {
        return revenue / units_sold;
    } else {
        return 0.0; // 避免除以零
    }
}
~~~

像其他函数一样，当我们在类的外部定义成员函数时，成员函数的定义必须与它的声
明匹配。也就是说，返回类型、参数列表和函数名都得与类内部的声明保持一致。如果成
员被声明成常量成员函数，那么**它的定义也必须在参数列表后明确指定const属性**。同
时，**类外部定义的成员的名字必须包含它所属的类名**：

### 定义一个返回this对象的函数

~~~c++
Sales_data& Sales_data::combine(const Sales_data &rhs){
    units_sold += rhs.units_sold;
    revenue += rhs.revenue;
    return *this;//返回调用该函数的对象
}
~~~

### 定义非成员函数

~~~c++
istream &read(istream &is, Sales_data &item){
double price = 0.0;
    is >> item.bookNo >> item.units_sold >> price;
    if (is) {
        item.revenue = item.units_sold * price; //计算总收入
    } else {
        item = Sales_data(); //重置为默认状态
    }
    return is;
}
ostream &print(ostream &os, const Sales_data &item){
    os << item.isbn() << " " << item.units_sold << " " << item.revenue << " " << item.avg_price();
    return os;
}
Sales_data add(const Sales_data &lhs, const Sales_data &rhs) {
    Sales_data sum = lhs; // 创建一个新的Sales_data对象
    sum.combine(rhs); // 使用combine方法合并数据
    return sum; // 返回合并后的对象
}
~~~

~~~bash
103-xx-45 4 4.56
103-xx-45 2 4.56
103-xy-45 4 4.56
~~~

### 使用这些函数

~~~c++
int main(){
Sales_data total;
	if (read(std::cin, total))
	{
		Sales_data trans;
		while (read(std::cin, trans))
		{
			if (total.isbn() == trans.isbn())
				total.combine(trans);
			else
			{
				print(std::cout, total) << std::endl;
				total = trans;
			}
		}
		print(std::cout, total) << std::endl;
	}
	else
	{
		std::cerr << "No data?!" << std::endl;
		return -1;
	}

    
    return 0;
}
~~~

### 构造函数

编译器创建的构造函数又被称为合成的默认构造函数（synthesized default constructor)。对于大多数类来说，这个合成的默认构造函数将按照如下规则初始化类的数据成员：

- 如果存在类内的初始值，用它来初始化成员。
- 否则，默认初始化该成员。

~~~c++
//新增的构造函数
Sales_data() = default; // 默认构造函数
Sales_data(const std::string &s) : bookNo(s) {} // 带参数的构造函数
Sales_data(const std::string &s, unsigned n, double p) 
    : bookNo(s), units_sold(n), revenue(n * p) {}
Sales_data(std::istream &is); // 从输入流构造
~~~

#### 测试代码

~~~c++
int main()
{
	Sales_data item1;
	print(std::cout, item1) << std::endl;

	Sales_data item2("0-201-78345-X");
	print(std::cout, item2) << std::endl;

	Sales_data item3("0-201-78345-X", 3, 20.00);
	print(std::cout, item3) << std::endl;

	Sales_data item4(std::cin);
	print(std::cout, item4) << std::endl;
	return 0;
}
~~~

### 访问控制与封装

在C++语言中，我们使用访问说明符(accessspecifiers)加强类的封装性：

- 定义在public说明符之后的成员在整个程序内可被访问，public成员定义类的接口。
- 定义在private说明符之后的成员可以被类的成员函数访问，但是不能被使用该类的代码访问，private部分封装了（即隐藏了）类的实现细节。

### 使用class或struct关键字

类可以在它的第一个访问说明符之前定义成员，对这种成员的访问权限依赖于类定义的方式。如果我们使用struct关键字，则定义在第一个访问说明符之前的成员是public的：相反，如果我们使用class关键字，则这些成员是private的。

出于统一编程风格的考虑，当我们希望定义的类的所有成员是public的时，使用struct；反之，如果希望成员是private的，使用class。

### 友元

类可以允许其他类或者函数访问它的非公有成员，方法是令其他类或者函数成为它的友元(friend)。如果类想把一个函数作为它的友元，只需要增加一条以friend关键字开始的函数声明语句即可：

~~~c++
class Sales_data
{
    friend Sales_data add(const Sales_data &, const Sales_data &);
    friend std::istream &read(std::istream &, Sales_data &);
    friend std::ostream &print(std::ostream &, const Sales_data &);
...
}
~~~

友元的声明仅仅指定了访问的权限，而非一个通常意义上的函数声明。**如果我们希望类的用户能够调用某个友元函数，那么我们就必须在友元声明之外再专门对函数进行一次声明**。

### 类成员再探

#### 定义一个类型成员

~~~c++
#ifndef SCREEN_H
#define SCREEN_H

#include <iostream>
#include <string>
#include <vector>
class Screen
{
public:
    typedef std::string::size_type pos;
    Screen() = default; // 默认构造函数
    Screen(pos ht, pos wd, char c) : height(ht), width(wd), contents(ht * wd, c) {}
    char get() const { return contents[cursor]; } // 获取当前光标位置的字符
    inline char get(pos ht, pos wd) const;        // 获取指定位置的字符
    Screen &move(pos r, pos c);                   // 移动光标到指定位置
    void some_member() const;
    Screen &set(char c);
    Screen &set(pos r, pos c, char ch);

private:
    pos cursor = 0;
    pos height = 0, width = 0;
    std::string contents;
    mutable size_t access_ctr = 0; // 可变成员变量，用于跟踪访问次数
};
inline char Screen::get(pos r, pos c) const
{
    return contents[r * width + c]; // 返回指定位置的字符
}

inline Screen &Screen::move(pos r, pos c)
{
    cursor = r * width + c; // 计算光标的新位置
    return *this;           // 返回当前对象的引用
}

void Screen::some_member() const
{
    ++access_ctr; // 增加访问计数
}

inline Screen &Screen::set(char c)
{
    contents[cursor] = c; // 设置光标位置的字符
    return *this;         // 返回当前对象的引用
}
inline Screen &Screen::set(pos r, pos c, char ch)
{
    contents[r * width + c] = ch; // 设置指定位置的字符
    return *this;                 // 返回当前对象的引用
}
class Window_mgr
{
private:
    std::vector<Screen> screens{Screen(24, 80, ' ')}; // 存储屏幕对象的向量
};
#endif
~~~

### 可变数据成员

一个可变数据成员(mutable data member)永远不会是const，即使它是const对象的成员。因此，一个const成员函数可以改变一个可变成员的值。举个例子，我们将给Screen添加一个名为access_ctr的可变成员，通过它我们可以追踪每个Screen的成员函数被调用了多少次。

对于 `inline` 函数，通常应该在头文件中直接定义它（因为 `inline` 函数的定义需要在每个使用它的编译单元中可见），而不是在 `.cpp` 文件中定义

### 返回*this的成员函数

~~~c++
 Screen myScreen(5, 5, 'X'); // 创建一个5x5的屏幕，初始字符为'X'
 myScreen.move(4,0).set('#'); // 将光标移动到(4,0)并设置字符为'#'
~~~

和move操作一样，我们的set成员的返回值是调用set的对象的引用。返回引用的函数是左值的，意味着这些函数返回的是对象本身而非对象的副本。如果我们把一系列这样的操作连接在一条表达式中的话：

### 基于const的重载

通过区分成员函数是否是const的，我们可以对其进行重载

~~~c++
Screen &display(std::ostream &os);
const Screen &display(std::ostream &os) const;
~~~

因为非常量版本的函数对于常量对象是不可用的，所以我们只能在一个常量对象上调用const成员函数。另一方面，虽然可以在非常量对象上调用常量版本或非常量版本，但显然此时非常量版本是一个更好的匹配。

### 类之间的友元关系

~~~c++
class Screen
{
    friend class Window_mgr; // 声明Window_mgr为友元类
    ...
}
~~~

如果一个类指定了友元类，则友元类的成员函数可以访问此类**包括非公有成员在内的所有成员**。通过上面的声明，Window_mgr被指定为Screen的友元.

~~~c++
class Window_mgr
{
    public:
    using ScreenIndex = std::vector<Screen>::size_type; // 定义屏幕索引类型
    void clear(ScreenIndex); // 清除指定屏幕的内容
private:
    std::vector<Screen> screens{Screen(24, 80, ' ')}; // 存储屏幕对象的向量
};
void Window_mgr::clear(ScreenIndex i)
{
    Screen &s = screens[i]; // 获取指定索引的屏幕
    s.contents = std::string(s.height * s.width, ' '); // 清空屏幕内容
    s.cursor = 0; // 重置光标位置
}
~~~

必须要注意的一点是，**友元关系不存在传递性**。也就是说，如果window_mgr有它自已的友元，则这些友元并不能理所当然地具有访问Screen的特权。

#### 组织架构

声明Window_mgr为Screen的友元类

~~~c++
// 前向声明
class Window_mgr;

class Screen
{
    // 声明Window_mgr为Screen的友元类
    friend class Window_mgr;

public:
    typedef std::string::size_type pos;
    Screen() = default; // 默认构造函数
    Screen(pos ht, pos wd, char c) : height(ht), width(wd), contents(ht * wd, c) {}
    char get() const { return contents[cursor]; } // 获取当前光标位置的字符
    inline char get(pos ht, pos wd) const;        // 获取指定位置的字符
    Screen &move(pos r, pos c);                   // 移动光标到指定位置
    void some_member() const;
    Screen &set(char c);
    Screen &set(pos r, pos c, char ch);
    Screen &display(std::ostream &os);
    const Screen &display(std::ostream &os) const;

private:
    pos cursor = 0;
    pos height = 0, width = 0;
    std::string contents;
    mutable size_t access_ctr = 0; // 可变成员变量，用于跟踪访问次数
    void do_display(std::ostream &os) const
    {
        os << contents; // 显示屏幕内容
    }
};

class Window_mgr
{
public:
    using ScreenIndex = std::vector<Screen>::size_type; // 定义屏幕索引类型
    void clear(ScreenIndex);                            // 清除指定屏幕的内容

private:
    std::vector<Screen> screens{Screen(10, 5, '#')}; // 存储屏幕对象的向量
};

// 现在可以声明Window_mgr::clear为友元函数
inline void Window_mgr::clear(ScreenIndex i)
{
    Screen &s = screens[i];                            // 获取指定索引的屏幕
    s.contents = std::string(s.height * s.width, ' '); // 清空屏幕内容
    s.cursor = 0;                                      // 重置光标位置
}

inline char Screen::get(pos r, pos c) const
{
    return contents[r * width + c]; // 返回指定位置的字符
}

inline Screen &Screen::move(pos r, pos c)
{
    cursor = r * width + c; // 计算光标的新位置
    return *this;           // 返回当前对象的引用
}

void Screen::some_member() const
{
    ++access_ctr; // 增加访问计数
}

inline Screen &Screen::set(char c)
{
    contents[cursor] = c; // 设置光标位置的字符
    return *this;         // 返回当前对象的引用
}

inline Screen &Screen::set(pos r, pos c, char ch)
{
    contents[r * width + c] = ch; // 设置指定位置的字符
    return *this;                 // 返回当前对象的引用
}

Screen &Screen::display(std::ostream &os)
{
    do_display(os); // 调用私有成员函数显示内容
    return *this;   // 返回当前对象的引用
}

const Screen &Screen::display(std::ostream &os) const
{
    do_display(os); // 调用私有成员函数显示内容
    return *this;   // 返回当前对象的引用
}
~~~

#### 组织架构2

修改了`Window_mgr::clear`为友元函数，需要修改`std::vector<Screen> screens{Screen(10, 5, '#')};`，不能初始化。

### 类的作用域

一个类就是一个作用域的事实能够很好地解释为什么当我们在类的外部定义成员函数时必须同时提供类名和函数名。在类的外部，成员的名字被隐藏起来了。

~~~c++
class Window_mgr
{
public:
    ScreenIndex addScreen(const Screen &s); // 添加新屏幕并返回索引
...
};
~~~

~~~c++
Window_mgr::ScreenIndex Window_mgr::addScreen(const Screen &s)
{
    screens.push_back(s); // 将新屏幕添加到向量中
    return screens.size() - 1; // 返回新屏幕的索引
}
~~~

### 构造函数初始值列表

有时我们可以忽略数据成员初始化和赋值之间的差异，但并非总能这样。如果成员是**const或者是引用**的话，**必须将其初始化**。类似的，当成员属于某种**类类型且该类没有定义默认构造函数**时，也**必须将这个成员初始化**。

这样是正确的：

~~~c++
class ConstRef{
    public:
    ConstRef(int ii);
    private:
    int i;
    const int ci;
    int &ri;
};
ConstRef::ConstRef(int ii) : i(ii), ci(ii), ri(i) {
    // 初始化成员变量
}
~~~

和其他常量对象或者引用一样，成员ci和ri都必须被初始化。因此，如果我们没有为它们提供构造函数初始值的话将引发错误：

**这样是错误的：**

~~~c++
ConstRef::ConstRef(int ii) {
    i = ii; // 非const成员变量可以被修改
    ci = ii; // 错误，const成员变量只能在初始化列表中被赋值
    ri = i;  // 错误，ri没被初始化
}
~~~

> 如果成员是const、引用，或者属于某种未提供歇认构造函数的类类型，我价必须通过构造函数初始值列表为这些成员提供初值。

### 成员初始化的顺序

成员的初始化顺序与它们在类定义中的出现顺序一致

### 委托构造函数

和其他构造函数一样，一个委托构造函数也有一个成员初始值的列表和一个函数体。在委托构造函数内，成员初始值列表只有一个唯一的入口，就是类名本身。和其他成员初始值一样，类名后面紧跟圆括号括起来的参数列表，参数列表必须与类中另外一个构造函数匹配。

~~~c++
class Sales_data
{
public:
    //非委托构造函数使用对应的实参初始化成员
    Sales_data(const std::string &s, unsigned n, double p)
        : bookNo(s), units_sold(n), revenue(n * p) {}
    
    // 委托构造函数：从字符串初始化
    Sales_data():Sales_data("", 0, 0.0) {} // 默认构造函数，初始化为空状态
    Sales_data(const std::string &s) : Sales_data(s, 0, 0.0) {} // 从字符串构造
    // 委托构造函数：从输入流初始化
    Sales_data(std::istream &is):Sales_data(){read(is, *this);} 
    ...
}
~~~

### 抑制构造函数定义的隐式转换

在要求隐式转换的程序上下文中，我们可以通过将构造函数声明为`explicit`加以阻止

~~~c++
explicit Sales_data(const std::string &s) : Sales_data(s, 0, 0.0) {} 
explicit Sales_data(std::istream &is) : Sales_data() { read(is, *this); }
~~~

关键字`explicit`只对一个实参的构造函数有效。**需要多个实参的构造函数不能用于执行隐式转换**，所以无须将这些构造函数指定为explicit的。只能在类内声明构造函数时使用explicit关键字，**在类外部定义时不应重复**

### 聚合类

聚合类(aggregate class)使得用户可以直接访问其成员，并且具有特殊的初始化语法形式。当一个类满足如下条件时，我们说它是聚合的。

- 所有成员都是public的。
- 没有定义任何构造函数。
- 没有类内初始值
- 没有基类，也没有virtua1函数

~~~c++
struct Data {
int ival;
string s;
}
~~~

### 类的静态成员

我们通过在成员的声明之前加上关键字static使得其与类关联在一起。和其他成员一样，静态成员可以是public的或private的。静态数据成员的类型可以是常量、引用、指针、类类型等。

~~~c++
#ifndef ACCOUNT_H
#define ACCOUNT_H
#include <iostream>
class Account
{
private:
    std::string owner;          // 账户所有者
    double amount;              // 账户余额
    static double interestRate; // 静态成员变量，存储利率
    static double initRate();   // 静态成员函数，初始化利率
    static constexpr int period = 30; // 静态常量，表示利率周期
    double daily_tbl[period]; // 静态数组，存储每天的利率
public:
    Account(const std::string &own, double amt)
        : owner(own), amount(amt) 
    {
        // 初始化daily_tbl数组
        for (int i = 0; i < period; ++i) {
            daily_tbl[i] = 0.0;
        }
    }
    void calculate()
    {
        amount += amount * interestRate; // 计算利息并更新余额
    }
    static double rate()
    {
        return interestRate;
    }
    static void rate(double newRate)
    {
        interestRate = newRate; // 设置新的利率
    }
};
double Account::interestRate = Account::initRate(); // 定义并初始化静态成员变量
double Account::initRate()
{
    return 0.02; // 默认利率2%
}
#endif // ACCOUNT_H
~~~

**静态成员函数也不与任何对象绑定在一起，它们不包含this指针**。作为结果，静态成员函数不能声明成const的，而且我们也不能在static函数体内使用this指针。这一限制既适用于this的显式使用，也对调用非静态成员的隐式使用有效。

和其他的成员函数一样，我们既可以在类的内部也可以在类的外部定义静态成员函数。**当在类的外部定义静态成员时，不能重复static关键字**，该关键字只出现在类内部的声明语句：、s