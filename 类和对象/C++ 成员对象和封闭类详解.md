

一个类的成员变量如果是另一个类的对象，就称之为 “成员对象”。包含成员对象的类叫封闭类（enclosed class）。

封闭类构造函数的初始化列表
-------------

当封闭类的对象生成并初始化时，它包含的成员对象也需要被初始化，这就会引发成员对象构造函数的调用。如何让编译器知道，成员对象到底是用哪个构造函数初始化的呢？这可以通过在定义封闭类的构造函数时，添加初始化列表的方式解决。

在构造函数中添加初始化列表的写法如下：

类名:: 构造函数名 (参数表): 成员变量 1(参数表), 成员变量 2(参数表), ...  
{  
    ...  
}

`:`和`{`之间的部分就是初始化列表。初始化列表中的成员变量既可以是成员对象，也可以是基本类型的成员变量。对于成员对象，初始化列表的 “参数表” 中存放的是构造函数的参数（它指明了该成员对象如何初始化）。对于基本类型成员变量，“参数表”中就是一个初始值。

“参数表” 中的参数可以是任何有定义的表达式，该表达式中可以包括变量甚至函数调用等，只要表达式中的标识符都是有定义的即可。例如：

```cpp
#include <iostream>
using namespace std;
class CTyre  //轮胎类
{
private:
    int radius;  //半径
    int width;  //宽度
public:
    CTyre(int r, int w) : radius(r), width(w) { }
};
class CEngine  //引擎类
{
};
class CCar {  //汽车类
private:
    int price;  //价格
    CTyre tyre;
    CEngine engine;
public:
    CCar(int p, int tr, int tw);
};
CCar::CCar(int p, int tr, int tw) : price(p), tyre(tr, tw)
{
};
int main()
{
    CCar car(20000, 17, 225);
    return 0;
}
```

第 9 行的构造函数添加了初始化列表，将 radius 初始化成 r，width 初始化成 w。这种写法比在函数体内用 r 和 w 对 radius 和 width 进行赋值的风格更好。建议对成员变量的初始化都使用这种写法。

CCar 是一个封闭类，有两个成员对象：tyre 和 engine。在编译第 27 行时，编译器需要知道 car 对象中的 tyre 和 engine 成员对象该如何初始化。

编评器已经知道这里的 car 对象是用上面的 CCar(int p, int tr, int tw) 构造函数初始化的，那么 tyre 和 engine 该如何初始化，就要看第 22 行 CCar(int p,int tr,int tw) 后面的初始化列表了。该初始化列表表明，tyre 应以 tr 和 tw 作为参数调用 CTyre(intr, hit w) 构造函数初始化，但是并没有说明 engine 该如何处理。在这种情况下，编译器就认为 engine 应该用 CEngine 类的无参构造函数初始化。而 CEngine 类确实有一个编译器自动生成的默认无参构造函数，因此，整个 car 对象的初始化问题就都解决了。

总之，生成封闭类对象的语句一定要让编译器能够弄明白其成员对象是如何初始化的，否则就会编译错误。

在上面的程序中，如果 CCar 类的构造函数没有初始化列表，那么第 27 行就会编译出错，因为编译器不知道该如何初始化 car.tyre 对象，因为 CTyre 类没有无参构造函数，而编译器又找不到用来初始化 car.tyre 对象的参数。

封闭类对象生成时，先执行所有成员对象的构造函数，然后才执行封闭类自己的构造函数。成员对象构造函数的执行次序和成员对象在类定义中的次序一致，与它们在构造函数初始化列表中出现的次序无关。

当封闭类对象消亡时，先执行封闭类的析构函数，然后再执行成员对象的析构函数，成员对象析构函数的执行次序和构造函数的执行次序相反，即先构造的后析构，这是 [C++](http://c.biancheng.net/cplus/) 处理此类次序问题的一般规律。

例如下面的程序：

```cpp
#include<iostream>
using namespace std;
class CTyre {
public:
    CTyre() { cout << "CTyre constructor" << endl; }
    ~CTyre() { cout << "CTyre destructor" << endl; }
};
class CEngine {
public:
    CEngine() { cout << "CEngine constructor" << endl; }
    ~CEngine() { cout << "CEngine destructor" << endl; }
};
class CCar {
private:
    CEngine engine;
    CTyre tyre;
public:
    CCar() { cout << "CCar constructor" << endl; }
    ~CCar() { cout << "CCar destructor" << endl; }
};
int main() {
    CCar car;
    return 0;
}
```

运行结果：  
>CEngine constructor  
CTyre constructor  
CCar constructor  
CCar destructor  
CTyre destructor  
CEngine destructor

封闭类的对象初始化时，要先执行成员对象的构造函数，是因为封闭类的构造函数中有可能用到成员对象。如果此时成员对象还没有初始化，那就不合理了。

思考题：为什么封闭类对象消亡时，要先执行封闭类的析构函数，然后才执行成员对象的析构函数？

封闭类的复制构造函数
----------

封闭类的对象，如果是用默认复制构造函数初始化的，那么它包含的成员对象也会用复制构造函数初始化。例如下而的程序：

```cpp
#include <iostream>
using namespace std;
class A
{
public:
    A() { cout << "default" << endl; }
    A(A &a) { cout << "copy" << endl; }
};
class B
{
    A a;
};
int main()
{
    B b1, b2(b1);
    return 0;
}
```

程序的输出结果是：  
>default  
copy

说明 b2.a 是用类 A 的复制构造函数初始化的，而且调用复制构造函数时的实参就是 b1.a。