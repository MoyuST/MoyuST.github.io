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

- 容器
  用来存放数据的数据结构

- 迭代器
  用来遍历数据

- 仿函数
  重载了operator()的class，模拟函数运算

- 算法
  通过迭代器提取容器内的数据（如排序等）

- 分配器
  为STL分配内存

- 配接器
  组合仿函数实现操作

## map vs multimap

两种map底层均由红黑树实现，所有元素都是pair(key, value)，会按照元素键值自动排序。  
**不同点在于multipmap允许键值重复**

## vector vs list

- vector
  - 连续存储的容器，动态数组，在堆上分配空间
  - 底层由数组实现
  - 两倍容量增长

- list
  - 动态链表，在堆上分配空间，每插入一个元数都会分配空间，每删除一个元素都会释放空间
  - 底层由双向链表实现
  
## 迭代器 vs 指针

迭代器不是指针，是类模板，表现的像指针。他只是模拟了指针的一些功能，通过**重载了指针的一些操作符，->、*、++、--等**。迭代器封装了指针，是一个“可遍历STL（Standard Template Library）容器内全部或部分元素”的对象， 本质是封装了原生指针，是指针概念的一种提升（lift），提供了比指针更高级的行为，相当于一种智能指针，他可以根据不同类型的数据结构来实现不同的++，--等操作。

## epoll原理

- epoll_create(int size)
  创建epoll对象
- epoll_ctl(int field, int op, int fd, struct epoll_event *event)
  对epoll对象进行操作，把需要监控的描述添加进去，这些描述如将会以epoll_event结构体的形式组成**一颗红黑树**
- int epoll_wait(int epfd, struct epoll_event *events,int maxevents, int timeout)
  用其进行阻塞，进入大循环，当某个fd上有事件发生时，内核将会把其对应的结构体放入到一个链表中，返回有事件发生的链表。

## n个整数的无序数组，找到每个元素后面比它大的第一个数，要求时间复杂度为O(N)

```cpp
vector<int> findMax(vector<int>num){
  if(num.size()==0)
    return num;
  vector<int>res(num.size());
  int i=0;
  stack<int>s;
  while(i<num.size()){
    if(s.empty()||num[s.top()]>=num[i]){
      s.push(i++);
    }
    else{
      res[s.top()]=num[i];
      s.pop();
    }
  }
  while(!s.empty()){
    res[s.top()]=INT_MAX;
    s.pop();
  }

  for(int i=0; i<res.size(); i++)
    cout<<res[i]<<endl;

  return res;
}
```

## STL里的 resize vs reserve

- resize
  改变当前容器的元素数量，若原数组的size小于resize的新size，会在数组中新增元素
- reserve
  改变容器的最大容量
