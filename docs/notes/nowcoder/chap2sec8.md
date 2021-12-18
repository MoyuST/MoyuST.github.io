---
layout: post
title: 第二章 第八节 C++11
date: 18/12 2021
permalink: /notes/nowcoder/chap2sec8/
---

## 类的初始化

credit to [zdd](http://www.cnblogs.com/graphics/)

### 初始化列表

e.g.

```cpp
struct foo
{
    string name ;
    int id ;
    foo(string s, int i):name(s), id(i){} ; // 初始化列表
};
```

构造函数的冒号后即位初始化列表.

### 构造函数的两个执行阶段

1. 初始化阶段
    所有类类型的成员都会在初始化阶段初始化，即使该成员**没有出现在初始化列表里**
    若使用初始化列表赋值，则可以直接跳过类成员初始化而直接用赋值

2. 计算阶段
   一般指构造函数体内给成员变量赋值

### 初始化列表适合放什么变量

- 常量成员
  因为**常量**只能初始化不能赋值，所以必须放在初始化列表里面
- 引用类型
  引用必须在定义的时候初始化，所以也要写在初始化列表里面
- 没有默认构造函数的类类型
  因为使用初始化列表可以不必调用默认构造函数来初始化，而是直接调用**拷贝构造函数**初始化

## C++11 新特性

- auto
  编译器可以根据初始值自动推导出类型。但是不能用于**函数传参**以及**数组类型**的推导
- nullptr
  nullptr是一种特殊类型的字面值，它可以被转换成任意其它的指针类型；而NULL一般被宏定义为0，在遇到重载时可能会出现问题
- 智能指针
  包括shared_ptr,weak_ptr等
- 初始化列表
  [见上文](#初始化列表)
- 右值引用
  [实现语义转换(std::move)和完美转发(std::forward)](chap2sec6.md)
- atomic
  多线程加锁操作
- STL新增 array和tuple

## C++11 可变参数模板

e.g.

```cpp
Template<class ... T>
void func(T ... args)
{
    cout << "num is " << sizeof ... (args) << endl;
}
```

其中

- T叫做模板参数包
  省略号在左边意味着声明多个变量
- args叫做函数参数包
  省略号在右边意味着展开参数
  如
  
  ```cpp
  func(T ... args) # 相当于将T展开指args
  ```

## Lambda 表达式

形式：\[capture] (params) mutatble -> return-type {statement}
