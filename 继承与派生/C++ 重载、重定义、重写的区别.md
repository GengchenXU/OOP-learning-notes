>a.成员函数被重载的特征：  
（1）相同的范围（在同一个类中）  
（2）函数名字相同  
（3）参数不同  
（4）virtual 关键字可有可无   
重载的目的：
重载又称为静态多态，静态绑定，静态决议等。因为要实现重载，所以C++和C的命名方式有所不同。重载主要是为了减轻程序员对函数名的记忆负担，让所有功能相
似的函数使用同一名字。  
b.覆盖(重写）是指派生类函数覆盖基类函数，特征是：  
（1）不同的范围（分别位于派生类与基类）  
（2）函数名字相同  
（3）参数相同  
（4）基类函数必须有virtual 关键字    
覆盖(重写)的目的：
覆盖是指父类中的虚函数在子类中被重新定义,所以在子类对象的虚函数列表中将由子类中重新定义的函数地址覆盖掉原来父类中虚函数的地址.  
c.“隐藏”是指派生类的函数屏蔽了与其同名的基类函数，规则如下：  
（1）如果派生类的函数与基类的函数同名，但是参数不同。此时，不论有无virtual关键字，基类的函数将被隐藏（注意别与重载混淆）  
（2）如果派生类的函数与基类的函数同名，并且参数也相同，但是基类函数没有virtual 关键字。此时，基类的函数被隐藏（注意别与覆盖混淆）  
隐藏的目的：
隐藏是因为在子类中定义了与基类同名的函数，而将基类的函数隐藏掉，要想访问基类的函数，则必须加作用域限定符。
## 一.重载
1.在同一个作用域下，函数名相同，函数的参数不同（参数不同指参数的类型或参数的个数不相同）
2.不能根据返回值判断两个函数是否构成重载。
3.当函数构成重载后，调用该函数时，编译器会根据函数的参数选择合适的函数进行调用。
4.构成重载的例子：
```cpp
#include<iostream>
using namespace std;
int Add(int a, int b)
{
      return a + b;
}
double Add(double a, double b)
{
      return a + b;
}
int main()
{
      cout << Add(3, 4) << endl;   
      cout << Add(3.1, 4.1) << endl;
      system("pause");
      return 0;
}
```
5.运行结果：
![](https://img-blog.csdn.net/20180428111022858?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h1MTEwNTc3NTQ0OA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 二.重定义（隐藏）
1.在不同的作用域下（这里不同的作用域指一个在子类，一个在父类 ），函数名相同的两个函数构成重定义。
2.当两个函数构成重定义时，父类的同名函数会被隐藏，当用子类的对象调用同名的函数时，如果不指定类作用符，就只会调用子类的同名函数。
3.如果想要调用父类的同名函数，就必须指定父类的域作用符。
注意：当父类和子类的成员变量名相同时，也会构成隐藏。
4.隐藏的例子：
（1）定义一个B的对象b，当访问同名的函数fun1时，不指定域，会默认访问子类的fun1。
![](https://img-blog.csdn.net/20180428111035839?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h1MTEwNTc3NTQ0OA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
（2）定义一个B的对象b，当访问同名的函数fun1时，指定域为A，则会访问父类的fun1。
![](https://img-blog.csdn.net/20180428111046280?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h1MTEwNTc3NTQ0OA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
（3）将父类与子类的同名函数fun1进行如下改写，使这两个函数的参数不同，
```cpp
class A
{
public:
      void fun1(char c)
      {
           cout << "A::fun1()" << endl;
      }
      int _a;
};
class B : public A
{
public:
      void fun1(int a,int b)
      {
           cout << "B::fun1()" << endl;
      }
      int _b;
};
int main()
{
      B b;
      b.fun1('a');
      system("pause");
      return 0;
}
```
当定义B的对象b，调用与父类参数相同的同名函数时会出现错误，因为子类隐藏了父类的同名成员函数，不指定域，只会调用子类的同名成员函数，而传的参数与子类同名成员函数的参数不同，所以会编译失败
![](https://img-blog.csdn.net/20180428111102301?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h1MTEwNTc3NTQ0OA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
当指定作用域时，就会调用父类的同名函数
![](https://img-blog.csdn.net/20180428111111861?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h1MTEwNTc3NTQ0OA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

三.重写（覆盖）
1.在不同的作用域下（一个在父类，一个在子类），函数的函数名、参数、返回值完全相同，父类必须含有virtual关键字（协变除外）。
2.什么是协变？
（1）函数的函数名相同，参数也相同，但是函数的返回值可以不同（但必须只能是一个返回父类的指针（或引用）一个返回子类的指针（或引用），父类必须含有virtual关键字。
（2）构成协变的一种方式，返回指针；
```cpp
class A
{
public:
      virtual A* func1()
      {
           cout << "A::func1()" << endl;
           return this;
      }
private:
      int _a;
};
class B :public A
{
public:
      virtual A* func1()
      {
           cout << "B::func1()" << endl;
           return this;
      }
private:
      int _b;
};
int main()
{
      A a;
      B b;
      A* p = &a;   //多态的场景
      p->func1();
      p = &b;
      p->func1();
      system("pause");
      return 0;
}
```
则运行结果如下：（因为是父类对象的指针，由包含协变，所以构成多态，指向父类调父类指向子类调子类）
![](https://img-blog.csdn.net/20180428111131688?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h1MTEwNTc3NTQ0OA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
（3）构成协变的另一种方式，返回引用；
```cpp
class A
{
public:
      virtual A& func1()
      {
           cout << "A::func1()" << endl;
           return *this;
      }
private:
      int _a;
};
class B :public A
{
public:
      virtual A& func1()
      {
           cout << "B::func1()" << endl;
           return *this;
      }
private:
      int _b;
};
int main()
{
      A a;
      B b;
      A* p = &a;
      p->func1();
      p = &b;
      p->func1();
      system("pause");
      return 0;
}
```
运行结果如下，仍然构成多态
![](https://img-blog.csdn.net/20180428111141991?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h1MTEwNTc3NTQ0OA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
3.构成重写的代码：
```cpp
class A
{
public:
      virtual void func1()
      {
           cout << "A::func1()" << endl;
      }
private:
      int _a;
};
class B :public A
{
public:
      virtual void func1()
      {
           cout << "B::func1()" << endl;
      }
private:
      int _b;
};
int main()
{
      A a;
      B b;
      A* p = &a;
      p->func1();
      p = &b;
      p->func1();
      system("pause");
      return 0;
}
```
## 示例程序
```cpp
#include <iostream>
#include <stdlib.h>
#include <string.h>

using namespace std;

class Base 
{
	private:
	virtual void display() {cout << "Base display()" << endl;}
	void say() {cout << "Base say()" << endl;}
	
	public:
	void exec(){ display(); say(); }
	void f1(string a) { cout<<"Base f1(string)"<<endl; }
	void f1(int a) { cout<<"Base f1(int)"<<endl; } //overload，两个f1函数在Base类的内部被重载
	void f2(char a) { cout<<"Base f2(char)"<<endl; }
};

class DeriveA: public Base 
{
	public:
	void display() { cout<<"DeriveA display()"<<endl; } //override，基类中display为虚函数，故此处为重写
	void f1(int a,int b) { cout<<"DeriveA f1(int,int)"<<endl; } //redefining，f1函数在Base类中不为虚函数，故此处为重定义
	void say() { cout<<"DeriveA say()"<<endl; } //redefining，同上
};

class DeriveB: public Base
{
	public:
	void f1(int a) { cout<<"DeriveB f1(int)"<<endl; } //redefining，重定义
	void f2(int a) { cout<<"Base f2(int)"<<endl; }
};



int main(void){
	DeriveA a;
	Base *b=&a;
	b->exec(); //display():version of DeriveA call(polymorphism) //say():version of Base called(allways )
	cout << "***" << endl;
	a.exec(); //same result as last statement   
	cout << "***" << endl;
	a.say();
	cout << "***" << endl;
	DeriveB c;
	c.f1(1); //version of DeriveB called
	cout << "***" << endl;
	c.f2(6);
	return 0;
}
```

运行结果

>DeriveA display()
Base say()
*******
DeriveA display()
Base say()
*******
DeriveA say()
*******
DeriveB f1(int)
*******
Base f2(int)
