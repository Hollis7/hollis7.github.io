---
title: c++ primer ch5
categories:
- c++
tags:
- c++
- vscode
- cmake
---
<meta name="referrer" content="no-referrer"/>

### 内容导航

本节主要包括c++的语句知识

<!--more-->

### switch内部的控制流

理解程序在case标签之间的执行流程非常重要。如果某个case标签匹配成功，将从该标签开始往后顺序执行所有case分支，除非程序显式地中断了这一过程，否则直到switch的结尾处才会停下来。要想避免执行后续case分支的代码，我们必须显式地告诉编译器终止执行过程。大多数情况下，在下一个case标签之前应该有一条break语句。

~~~c++
int main()
{
    char ch;
    unsigned vowelCnt = 0;
    unsigned otherCnt = 0;

    while (cin >> ch)
    {
        switch (ch)
        { // 出现了a、e、i、o或u中的任意一个都会将vowelCnt的值加1
        case 'a':
        case 'e':
        case 'i':
        case 'o':
        case 'u':
            ++vowelCnt;
            break;
        default:
            ++otherCnt;
            break;
        }
    }

    cout << "vowelCnt: " << vowelCnt << endl;
    return 0;
}
~~~

~~~c++
a e o i u o i
vowelCnt: 7
~~~

或者这种形式：

~~~c++
switch (ch)
    { // 出现了a、e、i、o或u中的任意一个都会将vowelCnt的值加1
    //另一种合法的书写形式
        case'a': case 'e': case 'i': case 'o': case 'u':
        ++vowelCnt;
        break;
    }
~~~

### 跳转语句

#### break

break语句(breakstatement)负责终止离它最近的while、do-while、for或switch语句，并从这些语句之后的第一条语句开始继续执行。

#### continue

continue语句(continue statement)终止最近的循环中的当前迭代并立即开始下一次迭代。continue语句只能出现在for、while和do-while循环的内部，或者嵌套在此类循环里的语句或块的内部。和break语句类似的是，出现在嵌套循环中的continue语句也仅作用于离它最近的循环。和break语句不同的是，只有当switch语句嵌套在迭代语句内部时，才能在switch里使用continue。

#### goto语句

goto语句(goto statement)的作用是从goto语句无条件跳转到同一函数内的另一条语句。

> 不要在程序中使用goto语句，因为它使得程序既难理解又难修改

### try

#### throw表达式

~~~c++
char character1;
char character2;
cin >> character1 >> character2;
if (character1 != character2)
{
    throw runtime_error("character must be the same!!!");
}
else
{
    cout << "character: " << character1 << endl;
}

return 0;
~~~

类型`runtime_error`是标准库异常类型的一种，定义在`stdexcept`头文件中。

#### try语句块

~~~c++
try {program-statements}
catch (exception-declaration){handler-statements}
catch (exception-declaration) {handler-statements}
//...
~~~

### 标准异常

- exception头文件定义了最通用的异常类`exception`。它只报告异常的发生，不提供任何额外信息。
- stdexcept头文件定义了几种常用的异常类
- new头文件定义了`bad_alloc`异常类型，
- type_info头文件定义了`bad_cast`异常类型

<img src="C:\data\mysoftware\Typora\typoraPicture\image-20250714111927211.png" alt="image-20250714111927211" style="zoom: 50%;" />

### 举例

~~~c++
int main()
{
    int i, j;
    cout << "please input tow numbers: " << endl;
    while (cin >> i >> j)
    {
        try
        {
            if (j == 0)
                throw runtime_error("divisor is 0");
            cout << i / j << endl;
        }
        catch (runtime_error err)
        {
            cout << err.what() << "\nTry Again? Enter y or n" << endl;
            char c;
            cin >> c;
            if (c != 'y')
                break;
        }
        cout << "please input tow numbers: " << endl;
    }

    return 0;
}
~~~

