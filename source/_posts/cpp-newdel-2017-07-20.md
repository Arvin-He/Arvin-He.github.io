---
title: C++的new/delete操作符
date: 2017-07-20 14:20:11
tags: C/C++
categories: 编程
---

### new/delete operator 和 operator new/delete 区别
new/delete/new[]/delete[] operator   
operator new/delete/new[]/delete[]
以new为例子
new operator 即new 操作符
分配内存就使用 operator new

### C++的new是怎么实现的
```cpp
Class *pc = new Class;
// ...
delete pc;
```

上面代码的第一行即为 `new operator` ，而第三行即为 `delete operator` ，代码很简单，但对编译器来说，它需要做额外的工作，将上述代码翻译为近似于下面的代码：

```cpp
void *p = operator new(sizeof(Class));
// 对p指向的内存调用Class的构造函数，此处无法用直观的代码展现
Class *pc = static_cast<Class*>(p);
// ...
pc->~Class();
operator delete(pc);
```

面代码中，第一行即为 operator new ，而最后一行即为 operator delete 

new operator 实际上做了两件事情：
1. 调用 operator new 分配内存
2. 在分配好的内存上初始化对象，并返回指向该对象的指针

它可能会调用malloc, 但是具体如何分配内存要取决于实现. 相比于malloc, new默认在申请失败的时候会抛出异常而不是直接返回0.和new对应的是delete, 它们必须成对出现. new[] 则要和 delete[] 同时出现.

而 delete operator 类似，调用析构函数，再调用 operator delete 释放内存

C++标准库的实现之一——Clang的libcxx是如何实现全局的 operator new/delete 的（去掉了一些控制编译选项的宏定义，只留下了核心代码）：
```cpp
void * operator new(std::size_t size) throw(std::bad_alloc) {
    if (size == 0)
        size = 1;
    void* p;
    while ((p = ::malloc(size)) == 0) {
        std::new_handler nh = std::get_new_handler();
        if (nh)
            nh();
        else
            throw std::bad_alloc();
    }
    return p;
}

void operator delete(void* ptr) {
    if (ptr)
        ::free(ptr);
}
```

神秘的 `operator new/delete` 在背后也不过是调用C函数库的 `malloc/free`. 当然，这跟实现有关，libcxx这样实现，不代表其它实现也是如此。

### new与malloc区别
new可以认为是一种封装，有一个全局函数叫`operator new(size_t)`就是一般意义上的new，你可以重写它，自己来实现分配内存并调用构造,一般认为new和malloc最大的区别就在于是否调用构造函数,除了会调用构造函数，new 还可以抛出bad_alloc异常.



### 参考
[深入探究C++的new/delete操作符](https://kelvinh.github.io/blog/2014/04/19/research-on-operator-new-and-delete/)