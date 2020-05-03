<!--
 * @Description: 
 * @Output: 
 * @Autor: GengchenXu
 * @Date: 2020-05-02 20:02:21
 * @LastEditTime: 2020-05-03 17:17:42
 -->


我们知道，有时会让一个基类[指针](http://c.biancheng.net/c/80/)指向用 new 运算符动态生成的派生类对象；同时，用 new 运算符动态生成的对象都是通过 delete 指向它的指针来释放的。如果一个基类指针指向用 new 运算符动态生成的派生类对象，而释放该对象时是通过释放该基类指针来完成的，就可能导致程序不正确。

例如下面的程序：

```cpp
#include <iostream>
using namespace std;
class CShape  //基类
{
public:
    ~CShape() { cout << "CShape::destrutor" << endl; }
};
class CRectangle : public CShape  //派生类
{
public:
    int w, h;  //宽度和高度
    ~CRectangle() { cout << "CRectangle::destrutor" << endl; }
};
int main()
{
    CShape* p = new CRectangle;
    delete p;
    return 0;
}
```

程序的输出结果如下：  
CShape::destrutor

输出结果说明，`delete p;`只引发了 CShape 类的析构函数被调用，没有引发 CRectangle 类的析构函数被调用。这是因为该语句是静态联编的，编译器编译到此时，不可能知道此时 p 到底指向哪个类型的对象，它只根据 p 的类型是 CShape * 来决定应该调用 CShape 类的析构函数。

按理说，`delete p;`会导致一个 CRectangle 类的对象消亡，应该调用 CRectangle 类的析构函数才符合逻辑，否则有可能引发程序的问题。

例如，假设程序需要对 CRetangle 类的对象进行计数，如果此处不调用 CRetangle 类的析构函数，就会导致计数不正确。

再如，假设 CRectangle 类的对象在存续期间进行了动态内存分配，而释放内存的操作都是在析构函数中进行的，如果此处不调用 CRetangle 类的析构函数，就会导致被释放的对象中动态分配的内存以后再也没有机会回收。

综上所述，人们希望`delete p;`这样的语句能够聪明地根据 p 所指向的对象执行相应的析构函数。实际上，这也是多态。为了在这种情况下实现多态，[C++](http://c.biancheng.net/cplus/) 规定，需要将基类的析构函数声明为虚函数，即虚析构函数。

改写上面程序中的 CShape 类，在析构函数前加 virtual 关键字，将其声明为虚函数：

```cpp
class CShape{
public:
    virtual ~CShape() { cout << "CShape::destrutor" << endl; }
};
```

则程序的输出变为：  
CRectangle::destrutor  
CShape::destrutor

说明 CRetangle 类的析构函数被调用了。实际上，派生类的析构函数会自动调用基类的析构函数。

只要基类的析构函数是虚函数，那么派生类的析构函数不论是否用 virtual 关键字声明，都自动成为虚析构函数。

一般来说，一个类如果定义了虚函数，则最好将析构函数也定义成虚函数。

析构函数可以是虚函数，但是构造函数不能是虚函数。