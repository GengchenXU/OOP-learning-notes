<!--
 * @Description: 
 * @Autor: GengchenXu
 * @Date: 2020-03-28 18:49:57
 * @LastEditTime: 2020-03-28 22:49:39
 -->
对象数组中的元素同样需要用构造函数初始化。具体哪些元素用哪些构造函数初始化，取决于定义数组时的写法。

```cpp
#include<iostream>
using namespace std;
class CSample{
public:
    CSample(){  //构造函数 1
        cout<<"Constructor 1 Called"<<endl;
    }
    CSample(int n){  //构造函数 2
        cout<<"Constructor 2 Called"<<endl;
    }
}
int main(){
    CSample arrayl[2];
    cout<<"stepl"<<endl;
    CSample array2[2] = {4, 5};
    cout<<"step2"<<endl;
    CSample array3[2] = {3};
    cout<<"step3"<<endl;
    CSample* array4 = new CSample[2];
    delete [] array4;
    return 0;
}
```

程序的输出结果是：  
>Constructor 1 Called  
Constructor 1 Called  
stepl  
Constructor 2 Called  
Constructor 2 Called  
step2  
Constructor 2 Called  
Constructor 1 Called  
step3  
Constructor 1 Called  
Constructor 1 Called

第 13 行的 array1 数组中的两个元素没有指明如何初始化，要用无参构造函数初始化，因此输出两行 Constructor 1 Called。

第 15 行的 array2 数组进行了初始化，初始化列表 {4, 5} 可以看作用来初始化两个数组元素的参数，所以 array2[0] 以 4 为参数，调用构造函数 2 进行初始化；array2[1] 以 5 为参数，调用构造函数 2 进行初始化。这导致输出两行 Constructor 2 Called。

第 17 行的 array3 只指出了 array3[0] 的初始化方式，没有指出 array3[1] 的初始化方式，因此它们分别用构造函数 2 和构造函数 1 进行初始化。

第 19 行动态分配了一个 CSample 数组，其中有两个元素，没有指出和参数有关的信息，因此这两个元素都用无参构造函数初始化。

在构造函数有多个参数时，数组的初始化列表中要显式包含对构造函数的调用。例如下面的程序：

```cpp
#include <iostream>
using namespace std;
class CTest{
public:
    CTest(int n){cout<<1<<endl; }  //构造函数(1)
    CTest(int n, int m){cout<<2<<endl; }  //构造函数(2)
    CTest(){cout<<3<<endl; }  //构造函数(3)
};
int main(){
    //三个元素分别用构造函数(1)、(2)、(3) 初始化
    CTest arrayl [3] = { 1, CTest (1, 2) };
    //三个元素分别用构造函数(2)、(2)、(1)初始化
    cout<<"step1"<<endl;
    CTest array2[3] = { CTest(2,3), CTest (1,2), 1};
    //两个元素指向的对象分别用构造函数(1)、(2)初始化
    cout<<"step2"<<endl;
    CTest* pArray[3] = { new CTest(4) , new CTest(1,2) };
    return 0;
}
```
输出结果：
>1  
2  
3  
step1  
2  
2  
1  
step2  
1  
2
上面程序中比较容易令初学者困惑的是第 13 行。pArray 数组是一个[指针](http://c.biancheng.net/c/80/)数组，其元素不是 CTest 类的对象，而是 CTest 类的指针。第 13 行对 pArray[0] 和 pArray[1] 进行了初始化，把它们初始化为指向动态分配的 CTest 对象的指针，而这两个动态分配出来的 CTest 对象又分别是用构造函数（1）和构造函数（2）初始化的。pArray[2] 没有初始化，其值是随机的，不知道指向哪里。第 13 行生成了两个 CTest 对象，而不是三个，所以也只调用了两次 CTest 类的构造函数。