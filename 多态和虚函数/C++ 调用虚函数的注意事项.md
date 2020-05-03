当在类的内部调用虚函数时，需要注意如下几点。

在成员函数中调用虚函数
-----------

类的成员函数之间可以互相调用。在成员函数（静态成员函数、构造函数和析构函数除外）中调用其他虚成员函数的语句是多态的。例如下面的程序：

```
#include <iostream>
using namespace std;
class CBase
{
public:
    void func1()
    {
        func2();
    }
    virtual void func2()  {cout << "CBase::func2()" << endl;}
};
class CDerived:public CBase
{
public:
    virtual void func2() { cout << "CDerived:func2()" << endl; }
};
int main()
{
    CDerived d;
    d.func1();
    return 0;
}
```

程序的输出结果如下：  
CDerived:func2()

第 20 行调用 func1 成员函数。进入 func1 成员函数，执行到第 8 行，调用 func2 函数。看起来调用的应该是 CBase 类的 func2 成员函数，但输出结果证明实际上调用的是 CDerived 类的 func2 成员函数。

这是因为，在 func1 函数中，`func2();`等价于`this -> func2();`，而 this [指针](http://c.biancheng.net/c/80/)显然是 CBase* 类型的，即是一个基类指针，那么`this -> func2();`就是在通过基类指针调用虚函数，因此这条函数调用语句就是多态的。

当本程序执行到第 8 行时，this 指针指向的是一个 CDerived 类的对象，即 d，因此被调用的就是 CDerived 类的 func2 成员函数。

在构造函数和析构函数中调用虚函数
----------------

在构造函数和析构函数中调用虚函数不是多态，因为编译时即可确定调用的是哪个函数。如果本类有该函数，调用的就是本类的函数；如果本类没有，调用的就是直接基类的函数；如果直接基类没有，调用的就是间接基类的函数，以此类推。

请看下面的程序：

```
#include <iostream>
using namespace std;
class A
{
public:
    virtual void hello() { cout << "A::hello" << endl; }
    virtual void bye() { cout << "A::bye" << endl; }
};
class B : public A
{
public:
    virtual void hello() { cout << "B::hello" << endl; }
    B() { hello(); }
    ~B() { bye(); }
};
class C : public B
{
public:
    virtual void hello() { cout << "C::hello" << endl; }
};
int main()
{
    C obj;
    return 0;
}
```

程序的输出结果如下：  
B::hello  
A::bye

类 A 派生出类 B，类 B 派生出类 C。

第 23 行，obj 对象生成时会调用类 B 的构造函数，在类 B 的构造函数中调用 hello 成员函数。由于在构造函数中调用虚函数不是多态，所以此时不会调用类 C 的 hello 成员函数，而是调用类 B 自己的 hello 成员函数。

obj 对象消亡时，会引发类 B 析构函数的调用，在类 B 的析构函数中调用了 bye 函数。类 B 没有自己的 bye 函数，只有从基类 A 继承的 bye 函数，因此执行的就是类 A 的 bye 函数。

将在构造函数中调用虚函数实现为多态是不合适的。以上面的程序为例，obj 对象生成时，要先调用基类构造函数初始化其中的基类部分。在基类构造函数的执行过程中，派生类部分还未完成初始化。此时，在基类 B 的构造函数中调用派生类 C 的 hello 成员函数，很可能是不安全的。

在析构函数中调用虚函数不能是多态的原因也与此类似，因为执行基类的析构函数时，派生类的析构函数已经执行，派生类对象中的成员变量的值可能已经不正确了。

注意区分多态和非多态的情况
-------------

初学者往往弄不清楚一条函数调用语句是否是多态的。要注意，通过基类指针或引用调用成员函数的语句，只有当该成员函数是虚函数时才会是多态。如果该成员函数不是虚函数，那么这条函数调用语句就是静态联编的，编译时就能确定调用的是哪个类的成员函数。

另外，[C++](http://c.biancheng.net/cplus/) 规定，只要基类中的某个函数被声明为虚函数，则派生类中的同名、同参数表的成员函数即使前面不写 virtual 关键字，也自动成为虚函数。

例如下面的程序：

```
#include <iostream>
using namespace std;
class A
{
public:
    void func1() { cout<<"A::func1"<<endl; };
    virtual void func2() { cout<<"A::func2"<<endl; };
};
class B:public A
{
public:
    virtual void func1() { cout << "B::func1" << endl;  };
    void func2() { cout << "B::func2" << endl; }  //func2自动成为虚函数
};
class C:public B  // C以A为间接基类
{
public:
    void func1() { cout << "C::func1" << endl; }; //func1自动成为虚函数
    void func2() { cout << "C::func2" << endl; }; //func2自动成为虚函数
};
int main()
{
    C obj;
    A *pa = &obj;
    B *pb = &obj;
    pa->func2();  //多态
    pa->func1();  //不是多态
    pb->func1();  //多态
    return 0;
}
```

程序的输出结果如下：  
C::func2  
A::func1  
C::func1

基类 A 中的 func2 是虚函数，因此派生类 B、C 中的 func2 声明时虽然没有写 virtual 关键字，也都自动成为虚函数。所以第 26 行就是一个多态的函数调用语句，调用的是 C 类的 func2 成员函数。

基类 A 中的 func1 不是虚函数，因此第 27 行就不是多态的。编译时，根据 pa 的类型就可以确定 func1 就是类 A 的成员函数。

func1 在类 B 中成为虚函数，因此在类 B 的直接和间接派生类中，func1 都自动成为虚函数。因此，第 28 行，pb 是基类指针，func1 是基类 B 和派生类 C 中都有的同名、同参数表的虚函数，故这条函数调用语句就是多态的。