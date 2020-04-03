同类对象之间可以通过赋值运算符`=`互相赋值。如果没有经过重载，`=`的作用就是把左边的对象的每个成员变量都变得和右边的对象相等，即执行逐个字节拷贝的工作，这种拷贝叫作 “浅拷贝”。

有的时候，两个对象相等，从实际应用的含义上来讲，指的并不应该是两个对象的每个字节都相同，而是有其他解释，这时就需要对`=`进行重载。

上节我们定义了 String 类，并重载了`=`运算符，使得 char * 类型的字符串可以赋值给 String 类的对象。完整代码如下：

```cpp
#include <iostream>
#include <cstring>
using namespace std;
class String {
private:
    char * str;
public:
    String() :str(NULL) { }
    const char * c_str() const { return str; };
    String & operator = (const char * s);
    ~String();
};
String & String::operator = (const char * s)
//重载"="以使得 obj = "hello"能够成立
{
    if (str)
        delete[] str;
    if (s) {  //s不为NULL才会执行拷贝
        str = new char[strlen(s) + 1];
        strcpy(str, s);
    }
    else
        str = NULL;
    return *this;
}
String::~String()
{
    if (str)
        delete[] str;
};
int main()
{
    String s;
    s = "Good Luck,"; //等价于 s.operator=("Good Luck,");
    cout << s.c_str() << endl;
    // String s2 = "hello!";   //这条语句要是不注释掉就会出错
    s = "Shenzhou 8!"; //等价于 s.operator=("Shenzhou 8!");
    cout << s.c_str() << endl;
    return 0;
}
```

对于上面的代码，如果让两个 String 对象相等（把一个对象赋值给另一个对象），其意义到底应该是什么呢？是两个对象的 str 成员变量都指向同一个地方，还是两个对象的 str 成员变量指向的内存空间中存放的内容相同？如果把 String 对象理解为存放字符串的对象，那应该是后者比较合理和符合习惯，而前者不但不符合习惯，还会导致程序漏洞。

按照上面代码中 String 类的写法，下面的程序片段会引发问题：

```
String s1, s2;
s1 = "this";
s2 = "that";
s2 = s1;
```

执行完上面的第 3 行后，s1 和 s2 的状态如图 1 (a) 所示，它们的 str 成员变量指向不同的存储空间。

![](http://c.biancheng.net/uploads/allimg/180829/1-1PR9152935560.jpg)  
图 1：浅拷贝导致的错误

`s2=s1;`执行的是浅拷贝。执行完`s2=s1;`后，s2.str 和 s1.str 指向同一个地方， 如图 1 (b) 所示。这导致 s2.str 原来指向的那片动态分配的存储空间再也不会被释放，变成内存垃圾。

此外，s1 和 s2 消亡时都会执行`delete[] str;`，这就使得同一片存储空间被释放两次，会导致严重的内存错误，可能引发程序意外中止。

而且，如果执行完`s1=s2;`后 又执行`s1 = "some";`，则会导致 s2.str 也被释放。

为解决上述问题，需要对做`=`再次重载。重载后的的逻辑，应该是使得执行`s2=s1;`后，s2.str 和 s1.str 依然指向不同的地方，但是这两处地方所存储的字符串是一样的。再次重载`=`的写法如下：

```cpp
String & String::operator = (const String & s)
{
    if(str == s.str)
        return * this;
    if(str)
        delete[] str;
    if(s.str){  //s. str不为NULL才执行复制操作
    str = new char[ strlen(s.str) + 1 ];
        strcpy(str, s.str);
    }
    else
        str = NULL;
    return * this;
}
```

经过重载，赋值号`=`的功能不再是浅拷贝，而是将一个对象中[指针](http://c.biancheng.net/c/80/)成员变量指向的内容复制到另一个对象中指针成员变量指向的地方。这样的拷贝就叫 “深拷贝”。

程序第 3 行要判断 str==s.str，是因为要应付如下的语句：

s1 = s1;

这条语句本该不改变 s1 的值才对。`s1=s1;`等价于`s.operator=(s1);`，如果没有第 3 行和第 4 行，就会导致函数执行中的 str 和 s.str 完全是同一个指针（因为形参 s 引用了实参 s1，因此可以说 s 就是 s1）。第 8 行为 str 新分配一片存储空间，第 9 行从自己复制到自己，那么 str 指向的内容就不知道变成什么了。

当然，程序员可能不会写`s1=s1;`这样莫名奇妙的语句，但是可能会写`rs1=rs2;`，如果 rs1 和 rs2 都是 String 类的引用，而且它们正好引用了同一个 String 对象，那么就等于发生了`s1=s1;`这样的情况。

思考题：上面的两个 operator= 函数有什么可以改进以提高执行效率的地方？

重载了两次`=`的 String 类依然可能导致问题。因为没有编写复制构造函数，所以一旦出现使用复制构造函数初始化的 String 对象（例如，String 对象作为函数形参，或 String 对象作为函数返回值），就可能导致问题。最简单的可能出现问题的情况如下：

```
String s2;
s2 = "Transformers";
String s1(s2);
```

s1 是以 s2 作为实参，调用默认复制构造函数来初始化的。默认复制构造函数使得 s1.str 和 s2.str 指向同一个地方，即执行的是浅拷贝，这就导致了前面提到的没有对`=`进行第二次重载时产生的问题。因此还应该为 String 类编写如下复制构造函数，以完成深拷贝：
```cpp
String::String(String & s)
{
    if(s.str){
        str = new char[ strlen(s.str) + 1 ];
        strcpy(str, s.str);
    }
    else
        str = NULL;
}
```

最后，给出 String 类的完整代码：

```cpp
class String {
private:
    char * str;
public:
    String() :str(NULL) { }
    String(String & s);
    const char * c_str() const { return str; };
    String & operator = (const char * s);
    String & operator = (const String & s);
    ~String();
};
String::String(String & s)
{
    if (s.str) {
        str = new char[strlen(s.str) + 1];
        strcpy(str, s.str);
    }
    else
        str = NULL;
}
String & String::operator = (const String & s)
{
    if (str == s.str)
        return *this;
    if (str)
        delete[] str;
    if (s.str) {  //s. str不为NULL才执行复制操作
        str = new char[strlen(s.str) + 1];
        strcpy(str, s.str);
    }
    else
        str = NULL;
    return *this;
}
String & String::operator = (const char * s)
//重载"="以使得 obj = "hello"能够成立
{
    if (str)
        delete[] str;
    if (s) {  //s不为NULL才会执行拷贝
        str = new char[strlen(s) + 1];
        strcpy(str, s);
    }
    else
        str = NULL;
    return *this;
}
String::~String()
{
    if (str)
        delete[] str;
};
```