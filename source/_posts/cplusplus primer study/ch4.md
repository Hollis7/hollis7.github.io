---
title: c++ primer ch4
categories:
- c++
tags:
- c++
- vscode
- cmake
---
<meta name="referrer" content="no-referrer"/>

### 内容导航

本节主要包括c++的表达式知识

<!--more-->

### 混用解引用和递增运算符

~~~c++
int main()
{
    vector<int> vec = {-1,-5,-4,-5,-4,0,2,3,5};
    auto p = vec.begin();
    while (p!=vec.end() && *p<=0)
    {
        cout<<*p++<<" ";//等价*(p++)
    }
    return 0;
}
~~~

后置递增运算符的优先级高于解引用运算符，因此`*p++`等价于`*(p++)`.`p++`把`p`的值加1,然后返回`p`的初始值的副本作为其求值结果，此时解引用运算符的运算对象是`p`未增加之前的值。最终，这条语句输出`p`开始时指向的那个元素，并将指针向前移动一个位置。

### 成员访问运算符

点运算符和箭头运算符都可用于访问成员，其中，点运算符获取类对象的一个成员：箭头运算符与点运算符有关，表达式`ptr->mem`等价于`(*ptr).mem`:

**因为解引用运算符的优先级低于点运算符**，所以执行解引用运算的子表达式两端必须加上括号。如果没加括号，代码的含义就大不相同了。

### 条件运算符

`?:`

~~~c++
cond?exprl:expr2;
~~~

举例如下：

~~~c++
int main()
{
    int grade = 89;
    string finalGrade = (grade < 60) ? "fail" : "pass";
    cout << "finalGrade: " << finalGrade << endl;
    return 0;
}
~~~

#### 嵌套条件运算符

允许在条件运算符的内部嵌套另外一个条件运算符。

~~~c++
int grade;
cin>>grade;
string finalGradeLevel = (grade>90) ? "excellent" : (grade<60)?"fail":"pass";
cout << "finalGradeLevel: " << finalGradeLevel << endl;
~~~

### 小节

没什么重点，扫一遍书就行
