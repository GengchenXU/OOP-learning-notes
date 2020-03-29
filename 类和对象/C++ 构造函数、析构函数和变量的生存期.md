构造函数在对象生成时会被调用，析构函数在对象消亡时会被调用。对象何时生成和消亡是由对象的生存期决定的。下面通过一个例子来加深对构造函数、析构函数和变量的生存期的理解。

```cpp
#include <iostream >
using namespace std;
class Demo {
    int id;
public:
    Demo(int i)
    {
        id = i;
        cout << "id=" << id << "constructed" << endl;
    }
    ~Demo()
    {
        cout << "id=" << id << "destructed" << endl;
    }
};
Demo d1(1);
void Func()
{
    static Demo d2(2);
    Demo d3(3);
    cout << "func" << endl;
}
int main()
{
    Demo d4(4);
    d4 = 6;
    cout << "main" << endl;
    {
        Demo d5(5);
    }
    Func();
    cout << "main ends" << endl;
    return 0;
}
```

运行结果（行号只是为了便于查看，它不是输出的一部分）：  
>01) id=1constructed  
02) id=4constructed  
03) id=6constructed  
04) id=6destructed  
05) main  
06) id=5constructed  
07) id=5destructed  
08) id=2constructed  
09) id=3constructed  
10) func  
11) id=3destructed  
12) main ends  
13) id=6destructed  
14) id=2destructed  
15) id=1destructed

要分析程序的输出，首先要看有没有全局对象。因为全局对象是进入 main 函数以前就形成的，所以全局对象在 main 函数开始执行前就会被初始化。

本程序第 16 行定义了全局对象 d1，因此 d1 初始化引发的构造函数调用，导致了第 1) 行的输出结果。

main 函数开始执行后，局部对象 d4 初始化，导致第 2) 行输出。

第 26 行，`d4=6;`，6 先被自动转换成一个临时对象。这个临时对象的初始化导致第 3) 行输出。临时对象的值被赋给 d4 后，这条语句执行完毕，临时对象消亡，因此引发析构函数调用，导致第 4) 行输出。

第 29 行的 d5 初始化导致第 6) 行输出。d5 的作用域和生存期都只到离它最近的，且将其包含在内的那一对`{}`中的`}`为止，即第 30 行的`}`，因此程序执行到第 30 行时 d5 消亡，引发析构函数调用，输出第 7) 行。

第 8) 行的输出是由于进入 Func 函数后，执行第 19 行的静态局部对象 d2 初始化导致的。

静态局部对象在函数第一次被调用并执行到定义它的语句时初始化，生存期一直持续到整个程序结束，所以即便 Func 函数调用结束，d2 也不会消亡。

Func 函数中的 d3 初始化导致了第 9) 行输出。

第 31 行，Func 函数调用结朿后，d3 消亡导致第 11) 行输出。

main 函数结束时，其局部变量 d4 消亡，导致第 13) 行输出。

整个程序结束时，全局对象 d1 和静态局部对象 d2 消亡，导致最后两行输出。