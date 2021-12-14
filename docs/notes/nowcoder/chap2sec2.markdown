---
layout: post
title: 第二章 第二节 基本语言（二）
date: 14/12 2021
permalink: /notes/nowcoder/chap2sec2/
---

---

## 智能指针

为什么要使用智能指针：

智能指针的作用是管理一个指针，因为存在以下这种情况：申请的空间在函数结束时忘记释放，造成内存泄漏。使用智能指针可以很大程度上的避免这个问题，因为智能指针就是**一个类**，当超出了类的作用域是，类会自动调用**析构函数**，析构函数会自动释放资源。所以智能指针的作用原理就是在函数结束时自动释放内存空间，不需要手动释放内存空间。

### 4种类型的指针

- auto_ptr
    采用所有权形式（一个智能独自占有一个变量），当p2剥夺p1的所有权时，p1失去了所有权，但是不会报错

    ```cpp
    auto_ptr< string> p1 (new string ("I reigned lonely as a cloud."));
    auto_ptr<string> p2;
    p2 = p1; //auto_ptr不会报错
    ```

- unique_ptr
    实现独占式拥有或严格拥有概念，保证同一时间内只有一个智能指针可以指向该对象。

    ```cpp
    unique_ptr<string> p3 (new string ("auto"));           //#4
    unique_ptr<string> p4；                           //#5
    p4 = p3;    //此时编译器会报错！！
    ```

    与此同时，unique_ptr还支持临时右值的赋值

    ```cpp
    unique_ptr<string> pu1(new string ("hello world"));
    unique_ptr<string> pu2;
    pu2 = pu1;     // #1 not allowed
    unique_ptr<string> pu3;
    pu3 = unique_ptr<string>(new string ("You"));           // #2 allowed
    ```

    如果要实现重用智能指针，可以用std::move给指针重新赋值

    ```cpp
    unique_ptr<string> ps1, ps2;
    ps1 = demo("hello");
    ps2 = move(ps1);
    ps1 = demo("alexia");
    cout << *ps2 << *ps1 << endl;
    ```

- shared_ptr
    实现共享式拥有概念。多个智能指针可以指向相同对象，该对象和其相关资源会在“最后一个引用被销毁”时候释放。从名字share就可以看出了资源可以被多个指针共享，它使用**计数机制**来表明资源被几个指针共享。可以通过成员函数**use_count()**来查看资源的所有者个数。除了可以通过new来构造，还可以通过**传入auto_ptr,unique_ptr,weak_ptr**来构造。当我们调用**release()**时，当前指针会释放资源所有权，计数减一。当计数等于0时，资源会被释放。

    ```cpp
    use_count 返回引用计数的个数
    unique 返回是否是独占所有权( use_count 为 1)
    swap 交换两个 shared_ptr 对象(即交换所拥有的对象)
    reset 放弃内部对象的所有权或拥有对象的变更, 会引起原有对象的引用计数的减少
    get 返回内部对象(指针), 由于已经重载了()方法, 因此和直接使用对象是一样的。如 shared_ptr<int> sp(new int(1)); sp 与 sp.get()是等价的
    ```

- weak_ptr
    weak_ptr 是一种不控制对象生命周期的智能指针, 它指向一个 shared_ptr 管理的对象. 进行该对象的内存管理的是那个强引用的 shared_ptr. weak_ptr只是提供了对管理对象的一个访问手段。weak_ptr设计的目的是为配合 shared_ptr 而引入的一种智能指针来协助 shared_ptr 工作, 它只可以从一个 shared_ptr 或另一个 weak_ptr 对象构造, **它的构造和析构不会引起引用记数的增加或减少**。weak_ptr是用来解决shared_ptr相互引用时的死锁问题,如果说两个shared_ptr相互引用,那么这两个指针的引用计数永远不可能下降为0,资源永远不会释放。它是对对象的一种弱引用，不会增加对象的引用计数，和shared_ptr之间可以相互转化，shared_ptr可以直接赋值给它，它可以通过调用lock函数来获得shared_ptr。

    ```cpp
    class B;
    class A
    {
    public:
        shared_ptr<B> pb_; // change to weak_ptr will solve this dead lock
        ~A()
        {
            cout<<"A delete\n";
        }
    };
    class B
    {
    public:
        shared_ptr<A> pa_;
        ~B()
        {
            cout<<"B delete\n";
        }
    };
    void fun()
    {
        shared_ptr<B> pb(new B());
        shared_ptr<A> pa(new A());
        pb->pa_ = pa;
        pa->pb_ = pb;
        // pa and pb will have two ptr referenced
        cout<<pb.use_count()<<endl;
        cout<<pa.use_count()<<endl;
    }
    int main()
    {
        fun();
        return 0;
    }
    ```

## 请你回答一下为什么析构函数必须是虚函数？为什么C++默认的析构函数不是虚函数  考点:虚函数 析构函数

将可能会被继承的父类的析构函数设置为虚函数，可以保证当我们new一个子类，然后使用**基类指针**指向该子类对象，释放基类指针时可以释放掉子类的空间，防止内存泄漏。

C++默认的析构函数不是虚函数是因为虚函数需要额外的**虚函数表和虚表指针**，占用额外的内存。而对于不会被继承的类来说，其析构函数如果是虚函数，就会浪费内存。因此C++默认的析构函数不是虚函数，而是只有当需要当作父类时，设置为虚函数。

## 虚函数与纯虚函数的区别

- [tutorial](https://www.runoob.com/w3cnote/cpp-virtual-functions.html)
- 在子类中继承了父类的虚函数，不加virtual仍是虚函数

## 友元函数

[tutorial](https://www.runoob.com/cplusplus/cpp-friend-functions.html)

## 函数指针

- C在编译时，每一个函数都有一个入口地址，该入口地址就是函数指针所指向的地址。有了指向函数的指针变量后，可用该指针变量调用函数，就如同用指针变量可引用其他类型变量一样，在这些概念上是大体一致的。

- 用途：
  
    调用函数和做函数的参数，比如回调函数（应用编程和系统编程）。

- 示例：

    ```cpp
    char * fun(char * p)  {…}       // 函数fun
    char * (*pf)(char * p);             // 函数指针pf
    pf = fun;                        // 函数指针pf指向函数fun
    pf(p);                        // 通过函数指针pf调用函数fun
    ```

## fork 函数

- 成功调用fork( )会创建一个新的进程，它几乎与调用fork( )的进程一模一样，这两个进程都会继续运行。在**子进程**中，成功的fork( )调用会返回**0**。在父进程中fork( )返回子进程的**pid**。如果出现错误，fork( )返回一个**负值**。
- 最常见的fork( )用法是创建一个新的进程，然后使用exec( )载入二进制映像，替换当前进程的映像。这种情况下，派生（fork）了新的进程，而这个子进程会执行一个新的二进制可执行文件的映像。这种“派生加执行”的方式是很常见的。
- 在早期的Unix系统中，创建进程比较原始。当调用fork时，内核会把所有的内部数据结构复制一份，复制进程的页表项，然后把父进程的地址空间中的内容逐页的复制到子进程的地址空间中。但从内核角度来说，逐页的复制方式是十分耗时的。现代的Unix系统采取了更多的优化，例如Linux，采用了**写时复制(copy-on-write)**的方法，而不是对父进程空间进程整体复制。

## C++ 析构函数

- 析构函数名也应与类名相同，只是在函数名前面加一个位取反符~，例如~stud( )，以区别于构造函数。它**不能带任何参数，也没有返回值（包括void类型）**。只能有一个析构函数，**不能重载**。
- 如果用户没有编写析构函数，编译系统会自动生成一个缺省的析构函数（即使自定义了析构函数，编译器也总是会为我们合成一个析构函数，并且如果自定义了析构函数，编译器在执行时会先调用自定义的析构函数再调用合成的析构函数），它也不进行任何操作。所以许多简单的类中没有用显式的析构函数。
- 类析构顺序：1）派生类本身的析构函数；2）对象成员析构函数；3）基类析构函数。

## 静态函数 vs 虚函数

静态函数在**编译的时候就已经确定运行时机**，虚函数在**运行的时候动态绑定**。虚函数因为用了**虚函数表机制**，调用的时候会增加一次内存开销

## 重载 vs 重写

- 重载：两个函数名相同，但是参数列表不同（个数，类型），返回值类型没有要求，在同一作用域中
- 重写：子类继承了父类，父类中的函数是虚函数，在子类中重新定义了这个虚函数，这种情况是重写

## 多态

- 静态多态
  - 重载，在编译时确定

- 动态多态
  - 利用虚函数机制，在运行期间动态绑定

## 虚函数机制

在有虚函数的类中，类的**最开始部分是一个虚函数表的指针**，这个指针指向一个虚**函数表**，表中放了**虚函数的地址**，实际的虚函数在代码段(.text)中。当子类继承了父类的时候也会**继承其虚函数表**，当子类**重写**父类中虚函数时候，会将其继承到的虚函数表中的地址替换为**重新写的函数地址**。使用了虚函数，会增加访问内存开销，降低效率。

## C++ 储存类型

- overview
  [进程内存](Assets/imgs/进程结构.png)
  [储存类型](Assets/imgs/进程储存.png)

- 程序分区
  - 代码区 (.text)
    > 用来存放代码（包括static函数）

  - 数据区
    - 静态储存区
      - data区(.data)
        > 用来存放常量、初始化的全局变量、初始化的静态变量
        - 初始化的全局变量
        - 初始化的静态变量
      - bbs区(.bbs)
        > 存放未初始化的全局变量，静态变量  
        > **注意**：当前文件用extern全局变量之后，不会在为其分配内存空间
    - 常量区
      > 字符串常量，经初始化后不能修改
    - 堆区
    - 栈区
  
- 储存类型
  - auto
    > 栈区
  - extern
  - register
  - static
    - const static
      - 预编译声明(static声明时直接赋值)
        > 既不存于动态也不存于静态，相当于C的宏
      - 静态常量属性(int ClassName::value = 1)
        > 位于常量区

## 以下四行代码的区别是什么？`const char *arr = "123"; char* brr = "123"; const char crr[] = "123"; char drr[] = "123";`

- const char *arr = "123"
  > arr位于栈上,"123"位于常量区
- char* brr = "123"
  > brr位于栈上,"123"位于常量区
- const char crr[] = "123"
  > 均位于栈上,但是编译器可能会将"123"优化至常量区
- char drr[] = "123"
  > 均位于栈区

## 如果同时定义了两个函数，一个带const，一个不带，会有问题吗？

> 不会,相当于函数重载

## 隐式转换

- 对于内置类型,低精度变量给高精度变量赋值会发生隐式转换
- 对于只存在单个参数的构造函数的对象构造来说，函数调用可以直接使用该参数传入，编译器会自动调用其构造函数生成临时对象(下面有例子)

```cpp
// Example program
#include <iostream>
#include <string>


class Test{
    int test1;
    int test2;
    
public:
    friend void anything(Test t);
    
    Test(int test1, int test2){
        this->test1 = test1;
        this->test2 = test2;
    }
};

void anything(Test t){
    std::cout << "test inside: " << t.test1 << " " << t.test2 << std::endl;
}   

int main()
{
    anything({1, 2});
}
```

## 类型转换 (credit to [ydar95](https://blog.csdn.net/ydar95/article/details/69822540))

- C中的强制转换
  先会把运算符两边的类型转换成相同的(整数提升),再进行运算
- const_cast
  - 常量指针被转化成非常量的**指针**，并且仍然指向原来的对象
  - 常量引用被转换成非常量的**引用**，并且仍然指向原来的对象
- static_cast
  - static_cast 作用和C语言风格强制转换的效果基本一样，由于**没有运行时类型检查来保证转换的安全性**，所以这类型的强制转换和C语言风格的强制转换都有安全隐患
  - 用于类层次结构中基类（父类）和派生类（子类）之间指针或引用的转换。注意：进行**上行转换（把派生类的指针或引用转换成基类表示）是安全的**；进行**下行转换**（把基类指针或引用转换成派生类表示）时，由于没有动态类型检查，所以**是不安全的**
  - 用于基本数据类型之间的转换，如把int转换成char，把int转换成enum。这种转换的安全性需要开发者来维护
  - static_cast不能转换掉原有类型的const、volatile、或者 __unaligned属性。(前两种可以使用const_cast 来去除)
- dynamic_cast
  用于继承(子类与父类指针和引用的)转换,转换失败返回nullptr
- reinterpret_cast
  - 用于处理无关类型转换,通常为操作数的位模式提供较低层次的重新解释.仅仅解释了比特模型,没有进行二进制转换(下面有例子)

    ```cpp
    // expre_reinterpret_cast_Operator.cpp  
    // compile with: /EHsc  
    #include <iostream>  

    // Returns a hash code based on an address  
    unsigned short Hash(void *p) {
        unsigned int val = reinterpret_cast<unsigned int>(p);
        return (unsigned short)(val ^ (val >> 16));
    }
    using namespace std;
    int main() {
        int a[20];
        for (int i = 0; i < 20; i++)
            cout << Hash(a + i) << endl;
    }
    ```

  - A pointer to any integral type large enough to hold it （指针转向足够大的整数类型）
  - A value of integral or enumeration type to a pointer （从整形或者enum枚举类型转换为指针）
  - A pointer to a function to a pointer to a function of a different type （从指向函数的指针转向另一个不同类型的指向函数的指针）
  - A pointer to an object to a pointer to an object of a different type （从一个指向对象的指针转向另一个不同类型的指向对象的指针）
  - A pointer to a member to a pointer to a member of a different class or type, if the types of the members are both function types or object types （从一个指向成员的指针转向另一个指向类成员的指针！或者是类型，如果类型的成员和函数都是函数类型或者对象类型）

## extern "C"

在C中,由于C不支持函数重载(overloading), 所以编译器会给每个函数唯一的名字(即函数名),但由于C++支持多态,所以其编译器会为重载的函数生成不同的名字,这样的话会让C编译器看不懂(在把cpp与c的程序链接起来的时候出问题).  
extern "C"{} 会保证{}内的函数符号不会mangle(破坏)

## new/delete vs malloc/free

- new/delete是C++关键字,不需要明确内存大小,会调用构造和析构函数,返回值需要强制转换
- malloc/free是C关键字,需要明确内存大小,不会调用构造和析构函数

## RTTI(Run-time type infomration) (retrieved from [here](https://www.geeksforgeeks.org/g-fact-33/))

In C++, RTTI (Run-time type information) is a mechanism that exposes information about an object’s data type at runtime and is available only for the classes which have at least one virtual function. It allows the type of an object to be determined during program execution

For example, dynamic_cast uses RTTI and following program fails with error “cannot dynamic_cast 'b' (of type 'class B*') to type `class D*' (source type is not polymorphic) ” because there is no virtual function in the base class B.

```cpp
// CPP program to illustrate 
// Run Time Type Identification 
#include<iostream>
using namespace std;
class B { };
class D: public B {};
  
int main()
{
    B *b = new D;
    D *d = dynamic_cast<D*>(b);
    if(d != NULL)
        cout<<"works";
    else
        cout<<"cannot cast B* to D*";
    getchar();
    return 0;
}
```

Adding a virtual function to the base class B makes it working.

```cpp
// CPP program to illustrate 
// Run Time Type Identification 
#include<iostream>
using namespace std;
class B { virtual void fun() {} };
class D: public B { };
  
int main()
{
    B *b = new D;
    D *d = dynamic_cast<D*>(b);
    if(d != NULL)
        cout << "works";
    else
        cout << "cannot cast B* to D*";
    getchar();
    return 0;
}
```

运行时类型检查，在C++层面主要体现在**dynamic_cast**和**typeid**,VS中虚函数表的-1位置存放了指向type_info的指针。对于存在虚函数的类型，typeid和dynamic_cast都会去查询type_info

## C++如何处理返回值

生成一个临时变量，把它的引用作为函数参数传入函数内。

### C++中拷贝构造函数能否进行值传递

不能.碰到函数调用需要以值传递的时候,会一直调用拷贝构造函数直至栈溢出

### 重载 vs. 重写

- 重载(overload)
  多态
- 重写(override)
  子类重写父类虚函数

## 仿函数

变量类并重载operator()操作符来使类具有函数的性质

## inline

- 有时调用函数时,进程可能会花费更多的时间switch content of the caller,然而callee只需要执行一段很短时间的程序,这个时候我们可以使用inline来节省I/O,compiler会展开inline定义的函数(跟C macro类似).
- C macro不会检查调用的参数类型,error-prone

## explict

it cannot be used for implicit conversions and copy-initialization.

---
