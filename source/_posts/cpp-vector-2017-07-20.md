---
title: C++的vertor实现
date: 2017-07-20 15:01:28
tags: C/C++
categories: 编程
---

### C++的vector实现
新增元素：Vector通过一个连续的数组存放元素，如果集合已满，在新增数据的时候，就要分配一块更大的内存，将原来的数据复制过来，释放之前的内存，在插入新增的元素。
插入元素: 插入新的数据分在最后插入push_back和通过迭代器在任何位置插入，
这里说一下通过迭代器插入，通过迭代器与第一个元素的距离知道要插入的位置，即int index=iter-begin()。这个元素后面的所有元素都向后移动一个位置，在空出来的位置上存入新增的元素。
```cpp
void insert(const_iterator iter,const T& t )
{  
    int index=iter-begin();
    if (index<size_)
    {
        if (size_==capacity_)
        {
            int capa=calculateCapacity();
            newCapacity(capa);
        }
        memmove(buf+index+1,buf+index,(size_-index)*sizeof(T)); 
        buf[index]=t;
        size_++;
    } 
}
```        

删除元素：删除和新增差不多，也分两种，删除最后一个元素pop_back和通过迭代器删除任意一个元素erase(iter)。通过迭代器删除还是先找到要删除元素的位置，即int index=iter-begin();这个位置后面的每个元素都想前移动一个元素的位置。同时我们知道erase不释放内存只初始化成默认值。

删除全部元素clear：只是循环调用了erase，所以删除全部元素的时候，不释放内存。内存是在析构函数中释放的。
```cpp
iterator erase(const_iterator iter)
{
    int index=iter-begin(); 
    if (index<size_ && size_>0)
    {
        memmove(buf+index ,buf+index+1,(size_-index)*sizeof(T)); 
        buf[--size_]=T();
    } 
    return iterator(iter); 
}
```

### 迭代器
迭代器iteraotr是STL的一个重要组成部分,通过iterator可以很方便的存储集合中的元素.STL为每个集合都写了一个迭代器, 迭代器其实是对一个指针的包装,实现一些常用的方法,如`++`,`--`,`!=`,`==`,`*`,`->`等, 通过这些方法可以找到当前元素或是别的元素. 

vector是STL集合中比较特殊的一个,因为vector中的每个元素都是连续的,所以在自己实现vector的时候可以用指针代替,如`typedef T* iterator;typedef const T* const_iterator`，如果STL中的函数能方便的操作自己写的集合，实现的迭代器最好继承`std::iterator<std::forward_iterator_tag,T>`。
`std::iterator<std::forward_iterator_tag,T>`的源码如下：
```cpp
template<class _Category, 
         class _Ty, 
         class _Diff = ptrdiff_t, 
         class _Pointer = _Ty *, 
         class _Reference = _Ty&>
    struct iterator
{    
    // base type for all iterator classes
    typedef _Category iterator_category;
    typedef _Ty value_type;
    typedef _Diff difference_type;
    typedef _Diff distance_type;    // retained
    typedef _Pointer pointer;
    typedef _Reference reference;
};
```

Iterator其中没有任何成员，只是定义了一组类型，所以继承它并不会让你的struct变大，这组类型是STL的内部契约，STL中的函数假设每个迭代器都定义了这些类型，所以只要你的迭代器定义了这些类型，就可以和STL函数集合一起使用。