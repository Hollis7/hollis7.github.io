---
title: c++ primer ch3
categories:
- c++
tags:
- c++
- vscode
- cmake
---
<meta name="referrer" content="no-referrer"/>

### 内容导航

本节主要包括c++的字符串、向量、数组

<!--more-->

### 命名空间的using声明

有了`using`声明就无须专门的前缀（形如命名空间：：）也能使用所需的名字了。`using`声明具有如下的形式：

~~~c++
using namespace::name；
~~~

**头文件不应包含using声明**

### string初始化

~~~c++
string s5 = "hiya"; // 拷贝初始化
string s6("hiya");  // 直接初始化
string s7(10, 'c'); // 直接初始化，s7的内容是cccccccccc
~~~

### 读写string对象

~~~c++
int main()
{
    string s;
    cin>>s;
    cout<<s<<endl;
    
    return 0;
}
~~~

~~~c++
in:hello world
out:hello
~~~

读取操作时，string对象会自动忽略开头的空白(即空格符、换行符、制表符等)并从第一个真正的字符开始读起，**直到遇见下一处空白为止**。

#### getline

~~~c++
string line;
while (getline(cin,line))
{
    cout << line << endl;
}
~~~

触发`getline`函数返回的那个换行符实际上被丢弃掉了，得到的string对象中并不包含该换行符。

#### size_type

string类及其他大多数标准库类型都定义了几种配套的类型。这些配套类型体现了标准库类型与机器无关的特性，类型`size_type`即是其中的一种。在具体使用的时候，通过作用域操作符来表明名字`size_type`是在类string中定义的。

如果一条表达式中已经有了`string::size()`函数就不要再使用int了，这样可以避免混用int和unsigned可能带来的问题。

#### string +

当把string对象和字符字面值及字符串字面值混在一条语句中使用时，必须确保每个加去运算符(+)的两侧的运算对象至少有一个是string:

### cctype头文件

<img src="C:\data\mysoftware\Typora\typoraPicture\image-20250712112437457.png" alt="image-20250712112437457" style="zoom:50%;" />

~~~c++
string line = "hello,WOrld!";
int cnt = 0;
for (auto c : line)
{
    if (ispunct(c))
    {
        ++cnt;
    }
}
cout << "cnt: " << cnt << endl;
for (char &c : line)
{
    c = toupper(c);
}
cout << line << endl;
~~~

~~~bash
cnt: 2
HELLO,WORLD!
~~~

### vector

#### 初始化

~~~c++
vector<int> v1;         // size:0,  no values.
vector<int> v2(10);     // size:10, value:0
vector<int> v3(10, 42); // size:10, value:42
vector<int> v4{ 10 };     // size:1,  value:10
vector<int> v5{ 10, 42 }; // size:2,  value:10, 42
vector<string> v6{ 10 };  // size:10, value:""
vector<string> v7{ 10, "hi" };  // size:10, value:"hi"
~~~

#### 添加元素

~~~c++
vector<int> a;
for(int i =0;i<100;i++){
    a.push_back(i);
}
cout<<a.size()<<endl;
~~~

如果循环体内部包含有向vec七or对象添加元素的语句， 则不能使用范围for循环，

#### 取值

~~~c++
vector<unsigned> scores(11, 0); // 11 个分数段， 全都初始化为0
unsigned int grade;
while (cin >> grade)
{
    if (grade <= 100)
        scores[grade / 10]++;
}
for (auto c : scores)
{
    cout << c << " ";
}

~~~

### iter

```
＊iter 返回迭代器iter所指元素的引用
iter->mem 解引用iter井获取该元素的名为mem的成员， 等价千(*iter).mem
++iter 令iter指示容器中的下一个元素
--iter 令iter指示容器中的上一个元素
iterl == iter2 判断两个迭代器是否相等（不相等）， 如果两个迭代器指示的是同一个元
iterl != iter2 素或者它们是同一个容器的尾后迭代器， 则相等；反之，不相等
```

~~~c++
int main()
{
    string s("some string");
    if (s.begin() != s.end())
    {
        auto it = s.begin();
        *it = toupper(*it);
    }
    cout << s << endl;

    return 0;
}
//Some string
~~~

~~~c++
int main()
{
    string s("some string");
    // 依次处理s的字符直至我们处理完全部字符或者遇到空白
    for (auto it = s.begin(); it != s.end() && !isspace(*it); ++it)
    {
        *it = toupper(*it); // 将当前字符改成大写形式
    }
    cout << s << endl;

    return 0;
}
~~~

只有string和vector等一些标准库类型有下标运算符，而并非全都如此。与之类似，所有标准库容器的迭代器都定义了`==`和`!=`，但是它们中的大多数都没有定义`＜`运算符。因此，只要我们养成使用迭代器和`!=`的习惯，就不用大在意用的到底是哪种容器类型。

#### 迭代器类型

```
vector<int>::iterator it1;//读写
string::iterator it2;//读写

vector<int>::const_iterator it3;//只读
string::const_iterator it4;//只读
 auto it5 = s.cbegin();//返回string::const_iterator
```

#### 结合解引用和成员访问操作

箭头运算符把解引用和成员访问两个操作结合在一起，也就是说，it->mem和（＊it）.mem表达的意思相同

~~~c++
int main()
{
   vector<string> text;
	text.push_back("aaaaaaaaaa aaaaaaaaa aaaaaa");
	text.push_back("");
	text.push_back("bbbbbbbbbbbbbb bbbbbbbbbbb bbbbbbbbbbbbb");

	for (auto it = text.begin(); it != text.end() && !it->empty(); ++it)
	{
		for (auto &c : *it)
		{
			if (isalpha(c)) c = toupper(c);
		}
	}

	for (auto it : text)
	{
		cout << it << endl;
	}
	return 0;

}
~~~

### 字符数组的特殊性

字符数组有一种额外的初始化形式，我们可以用字符串字面值对此类数组初始化。当使用这种方式时，一定要注总字符串字面值的结尾处还有一个**空字符**，这个空字符也会像字符串的其他字符一样被拷贝到字符数组中去：

~~~c++
char a1[] = {'C', '+', '+'};       // 列表初始化， 没有空字符,3
char a2[] = {'C', '+', '+', '\0'}; // 列表初始化， 含有显式的空宇符,4
char a3[] = "C++";                 // 自动添加表示宇符串结束的空字符,4
const char a4[6] = "Daniel";       // 错误： 没有空间可存放空字符！
~~~

### 理解复杂的数组声明

~~~c++
int arr[10];
int *ptrs[10]; // ptrs是含有10个整型指针的数组
// int&refs[10]=/*?*/;//错误：不存在引用的数组
int (*Parray)[10] = &arr; // Parray指向一个含有10个整数的数组
int (&arrRef)[10] = arr;  // arrRef引用一个含有10个整数的数组
int *(&arry)[10] = ptrs;  // arry是数组的引用， 该数组含有10个指针
~~~

默认情况下，**类型修饰符从右向左依次绑定**。对于ptrs来说：首先知道我们定义的是一个大小为10的数组，它的名字
是ptrs, 然后知道数组中存放的是指向int的指针。

但是对千Parray来说，从右向左理解就不太合理了。**因为数组的维度是紧跟着被声明的名字的**，所以就数组而言，**由内向外**阅读要比从右向左好多了。

由内向外的顺序可帮助我们更好地理解`Parray`的含义：首先是圆括号括起来的部分，`*Parray`意味着 Parray是个指针，接下来观察右边，可知道Parray是个指向大小为10的数组的指针，最后观察左边，知道数组中的元素是int。这样最终的含义就明白无误了，Parray是一个指针，它指向一个int数组，数组中包含10个元素。同理，(&arrRef)表示arrRef 是一个引用，它引用的对象是一个大小为10的数组，数组中元素的类型是int。
### 标准库函数begin 和end

尽管能计算得到尾后指针，但这种用法极易出错。为了让指针的使用更简单、更安全，C++11新标准引入了两个名为begin和end的函数。

~~~c++
int ia[] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9}; // ia是一个含有10个整数的数组
int *beg = begin(ia);                      // 指向ia首元素的指针
int *last = end(ia);                       // 指向arr尾元素的下一位置的指针
cout<<beg[5]<<endl;
cout<<*(beg+1)<<endl;
~~~

只要指针指向的是数组中的元素（或者数组中尾元素的下一位置），都可以执行下标运算

#### 使用数组初始化vector对象

~~~c++
int ia[] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9}; // ia是一个含有10个整数的数组
vector<int> vec1(begin(ia), end(ia));
~~~

