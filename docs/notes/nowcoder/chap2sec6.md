---
layout: post
title: 第二章 第六节 面向对象与泛式编程
date: 16/12 2021
permalink: /notes/nowcoder/chap2sec6/
---

## [C++ 右值引用](https://zhuanlan.zhihu.com/p/335994370)

### 左值 vs 右值

- 左值 有地址的变量
- 右值 没地址的字面量、临时值
  const & 不会修改指向值，因此可以引用常量

- 左值引用 int &
- 右值引用 int &&
  - 使用std::move()实现左值转换成右值，可以用于移动构造函数和移动复制重载函数
  - std::forward\<T>(u)

