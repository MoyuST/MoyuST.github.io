---
layout: post
title: 第二章 第五节 类和数据抽象
date: 16/12 2021
permalink: /notes/nowcoder/chap2sec5/
---

## C++ 成员访问符

- public
  members are accessible from outside the class

- private
  members cannot be accessed (or viewed) from outside the class

- protected
  members cannot be accessed from outside the class, however, they can be accessed in **inherited** classes. 也就是说能被 其它成员函数，**子类成员函数、友元函数访问**

## 请你来说一下C++中struct和class的区别

在C++中，可以用struct和class定义类，都可以继承。区别在于：structural的默认继承权限和默认访问权限是public，而class的默认继承权限和默认访问权限是private。

## 请你回答一下C++类内可以定义引用数据成员吗？

可以，必须通过类的构造函数初始化列表初始化。

