<!--
 * @Description: 
 * @Autor: GengchenXu
 * @Date: 2020-04-01 00:17:03
 * @LastEditTime: 2020-04-01 00:21:58
 -->
赋值运算符`=`要求左右两个操作数的类型是匹配的，或至少是兼容的。有时希望`=`两边的操作数的类型即使不兼容也能够成立，这就需要对`=`进行重载。[C++](http://c.biancheng.net/cplus/) 规定，`=`只能重载为成员函数。来看下面的例子。

要编写一个长度可变的字符串类 String，该类有一个 char* 类型的成员变量，用以指向动态分配的存储空间，该存储空间用来存放以`\0`结尾的字符串。String 类可以如下编写：

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

程序的运行结果：  
>Good Luck,  
Shenzhou 8!

第 8 行的构造函数将 str 初始化为 NULL，仅当执行了 operator= 成员函数后，str 才会指向动态分配的存储空间，并且从此后其值不可能再为 NULL。在 String 对象的生存期内，有可能从未执行过 operator= 成员函数，所以在析构函数中，在执行 delete[] str 之前，要先判断 str 是否为 NULL。

第 9 行的函数返回了指向 String 对象内部动态分配的存储空间的[指针](http://c.biancheng.net/c/80/)，但是不希望外部得到这个指针后修改其指向的字符串的内容，因此将返回值设为 const char*。这样，假定 s 是 String 对象，那么下面两条语句编译时都会报错，s 对象内部的字符串就不会轻易地从外部被修改了 ：

```cpp
char* p = s.c_str ();
strcpy(s.c_str(), "Tiangong1");
```

第一条语句出错是因为`=`左边是 char* 类型，右边是 const char * 类型，两边类型不匹配；第二条语句出错是因为 strcpy 函数的第一个形参是 char* 类型，而这里实参给出的却是 const char * 类型，同样类型不匹配。

如果没有第 13 行对`=`的重载，第 34 行的`s = "Good Luck,"`肯定会因为类型不匹配而编译出错。经过重载后，第 34 行等价`于s.operator=("Good Luck,");`，就没有问题了。

在 operator= 函数中，要先判断 str 是否已经指向动态分配的存储空间，如果是，则要先释放那片空间，然后重新分配一片空间，再将参数 s 指向的内容复制过去。这样，对象中存放的字符串就和 s 指向的字符串一样了。分配空间时，要考虑到字符串结尾的`\0`，因此分配的字节数要比 strlen(s) 多 1。

需要注意一点，即使对 = 做了重载，第 36 行的`String s2 = "hello!";`还是会编译出错，因为这是一条初始化语句，要用到构造函数，而不是赋值运算符 =。String 类没有编写参数类型为 char * 的构造函数，因此编译不能通过。补上构造函数
```cpp
String(const char *s){
    str = new char[strlen(s)+1];
    strcpy(str,s);
}
```
就上面的程序而言，对 operator= 函数的返回值类型没有什么特别要求，void 也可以。但是在对运算符进行重载时，好的风格是应该尽量保留运算符原本的特性，这样其他人在使用这个运算符时才不容易产生困惑。赋值运算符是可以连用的，这个特性在重载后也应该保持。即下面的写法应该合法：

a = b = c;

假定 a、b、c 都是 String 对象，则上面的语句等价于下面的嵌套函数调用：

a.operator=(b.operator=(c) );

如果 operator= 函数的返回值类型为 void，显然上面这个嵌套函数调用就不能成立。将返回值类型改为 String 并且返回 *this 可以解决问题，但是还不够好。因为，假设 a、b、c 是基本类型的变量，则

(a =b) = c;

这条语句执行的效果会使得 a 的值和 c 相等，即`a = b`这个表达式的值其实是 a 的引用。为了保持 = 的这个特性，operator= 函数也应该返回其所作用的对象的引用。因此，返回值类型为 String & 才是风格最好的写法。在 a、b、c 都是 String 对象时，`(a=b)=c;`等价于

(a.operator=(b) ).operator=(c);

a.operator=(b) 返回对 a 的引用后，通过该引用继续调用 operator=(c)，就会改变 a 的值。