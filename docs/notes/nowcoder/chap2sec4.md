---
layout: post
title: 第二章 第四节 容器和算法
date: 14/12 2021
permalink: /notes/nowcoder/chap2sec4/
---
## map vs set

map和set都是C++的关联容器，其底层实现都是**红黑树（RB-Tree）**。由于 map 和set所开放的各种操作接口，RB-tree 也都提供了，所以几乎所有的 map 和set的操作行为，都只是转调 RB-tree 的操作行为。

- **set的迭代器是const的**，不允许修改元素的值；map允许修改value，**但不允许修改key**。其原因是因为map和set是根据关键字排序来保证其有序性的，如果允许修改key的话，那么首先需要删除该键，然后调节平衡，再插入修改后的键值，调节平衡，如此一来，**严重破坏了map和set的结构**，导致iterator失效，不知道应该指向改变前的位置，还是指向改变后的位置。所以STL中将set的迭代器设置成const，不允许修改迭代器的值；而map的迭代器则不允许修改key值，允许修改value值。
- map支持下标操作，set不支持下标操作。map可以用key做下标，map的下标运算符[ ]将关键码作为下标去执行查找，**如果关键码不存在，则插入一个具有该关键码和mapped_type类型默认值的元素至map中**，因此下标运算符[ ]在map应用中需要慎用，const_map不能用，只希望确定某一个关键值是否存在而不希望插入元素时也不应该使用，mapped_type类型没有默认值也不应该使用。如果find能解决需要，尽可能用find。

## STL的allocator

- C++ 中的内存配置和释放
  - new
    1. 调用 ::operator new配置内存
    2. 调用对象构造函数构造对象内容
  - delete
    1. 调用析构函数
    2. 调用 ::operator delete释放内存
- STL allocator
  - alloc::allocate() 内存配置
  - alloc::deallocate() 内存释放
  - ::construct() 对象构造
  - ::destory() 对象析构
- SGI STL采用了两级配置器
  - 当分配的空间大小超过128B时，会使用第一级空间配置器;第一级空间配置器直接使用malloc()、realloc()、free()函数进行内存空间的分配和释放
  - 当分配的空间大小小于128B时，将使用第二级空间配置器;第二级空间配置器采用了内存池技术，通过空闲链表来管理内存

## STL 迭代器删除元素

- 序列容器(vector,deque)
  使用erase(iterator)后,后面的每个元素的迭代器都会失效,但是后面每个元素都会往前移动一个位置;同时erase会返回下一个有效的迭代值
- 关联容器(map,set)
  erase(iterator)之后,由于其内在实现是红黑树,删除当前元素不会影响到下一个元素,调用erase之前记录下一个元素迭代器即可
- list
  采用了不连续分配的内存,erase方法会返回下一个有效的iterator

## STL的map

- map为红黑树
- unordered map为hash

## STL的组成
