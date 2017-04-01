---
title: C++关键字
date: 2017-03-27 19:32:11
tags: C/C++
categories: 编程
---
# 1. static关键字
C语言代码是以文件为单位来组织的，在一个源程序的所有源文件中，一个外部变量（注意不是局部变量）或者函数只能在一个源程序中定义一次，如果有重复定义的话编译器就会报错。伴随着不同源文件变量和函数之间的相互引用以及相互独立的关系，产生了extern和static关键字。
<!--More-->
## 1.1 C中的static
一个进程在内存中的布局如下表所示:
|:-----:|
|栈区|
|堆栈增长区|
|堆区|
|其他段|
|.bss段|
|.data段|
|.text段|
其中
.text段保存进程所执行的程序二进制文件，
.data段保存进程所有的已初始化的全局变量，
.bss段保存进程未初始化的全局变量。
在进程的整个生命周期中，.data段和.bss段内的数据时跟整个进程同生共死的，也就是在进程结束之后这些数据内存才会被释放。

当一个进程的全局变量被声明为static之后，它的中文名叫静态全局变量。静态全局变量和其他的全局变量的存储地点并没有区别，都是在.data段（已初始化）或者.bss段（未初始化）内，但是它只在定义它的源文件内有效，其他源文件无法访问它。所以，普通全局变量加上static关键字修饰后,就只能被定义它的源文件中的变量或函数访问。

static局部变量

普通的局部变量在栈空间上分配，这个局部变量所在的函数被多次调用时，每次调用这个局部变量在栈上的位置都不一定相同。局部变量也可以在堆上动态分配，但是记得使用完这个堆空间后要释放之。

static局部变量中文名叫静态局部变量。它与普通的局部变量比起来有如下几个区别：

1. 位置：静态局部变量被编译器放在全局存储区.data（注意：不在.bss段内，原因见3）），所以它虽然是局部的，但是在程序的整个生命周期中存在。

2. 访问权限：静态局部变量只能被其作用域内的变量或函数访问。也就是说虽然它会在程序的整个生命周期中存在，由于它是static的，它不能被其他的函数和源文件访问。

3. 值：静态局部变量如果没有被用户初始化，则会被编译器自动赋值为0，以后每次调用静态局部变量的时候都用上次调用后的值。这个比较好理解，每次函数调用静态局部变量的时候都修改它然后离开，下次读的时候从全局存储区读出的静态局部变量就是上次修改后的值。

static函数
相信大家还记得C++面向对象编程中的private函数，私有函数只有该类的成员变量或成员函数可以访问。在C语言中，也有“private函数”，它就是接下来要说的static函数，完成面向对象编程中private函数的功能。

当你的程序中有很多个源文件的时候，你肯定会让某个源文件只提供一些外界需要的接口，其他的函数可能是为了实现这些接口而编写，这些其他的函数你可能并不希望被外界（非本源文件）所看到，这时候就可以用static修饰这些“其他的函数”.所以static函数的作用域是本源文件，把它想象为面向对象中的private函数就可以了。

[参考](http://blog.csdn.net/keyeagle/article/details/6708077)

## 1.2 C++中的static
## 1.1 为什么需要static
对于特定类类型的全体对象而言，访问一个全局变量有时是必要的。然而，全局变量会破坏封装,对象需要支持特定类抽象的实现。如果对象是全局的，一般的用户代码就可以修改这个值。鉴于此，类可以定义类静态成员，而不是定义一个可普遍访问的全局对象。

static数据成员的特性
通常，非static数据成员存在于类类型的每个对象中。然而，static数据成员独立于该类的任意对象而存在；
每个static数据成员是与类关联的对象，而不是与该类的对象相关联。
static数据成员独立于该类的任意对象而存在，每个static数据成员是与类关联,属于类的一部分.
static数据成员是存储在程序的静态存储区，而并不是在栈空间上。既然是static数据成员，所以关键字static是必不可少的.

static成员函数，static成员是类的组成部分并不是任何对象的组成部分，因此,static成员函数没有this指针.
类中的成员函数具有一个附加的隐含实参，即指向该类对象的一个指针。这个隐含实参命名为this。
因为static成员函数不是任何对象的组成部分，所以static成员函数就没有this形参了。
由于成员函数声明为const说明该成员函数不会修改该成员函数所属的对象，所以static成员函数不能声明为const。
static成员函数不是任何对象的组成部分。static成员函数可以直接访问所属类的static成员，但是不能直接使用非static成员函数

类中的static成员函数
在类中也可以定义static成员函数。类中的static成员函数没有this形参，它可以直接访问所属类的static成员，但不能直接使用非static成员。注意：类的非static成员函数是可以直接访问类的static和非static成员，而不用作用域操作符(::)。

使用static数据成员的优点：
避免命名冲突：static成员的名字在类的作用域中，因此可以避免与其他类的成员或全局对象名字冲突。
可以实施封装：static成员可以是私有成员，而全局对象不可以。
易读性：static成员是与特定类关联的，可显示程序员的意图。
static数据成员 与 非static数据成员调用方法：
非static成员通过对象调用。
static成员通过作用域操作符（直接调用）、对象、引用、指向该类类型对象的指针（间接调用）
```
class Person
{ 
    static double getHeight();
    static const age = 30;
    string school;
};
Person xiaoming;
Person *someone = &xiaoming;
double height;

height = Person::getHeight(); //static成员通过作用域操作符（直接调用）
height = xiaoming.getHeight();　　　//static成员通过对象（间接调用）
height = someone->getHeight();　　　//static成员通过指向该类类型对象的指针（间接调用）
```
static数据成员定义：
1、一般情况下，static数据成员是类内声明，类外定义;
2、static成员不通过类构造函数初始化，而是在定义时进行初始化；
3、一个例外：初始化式为常量表达式，整型static const 数据成员（static const int） 可以在类的定义体内进行初始化：

值得注意的是：const static数据成员在类的定义体中出始化时，该数据成员仍必须在类的定义体外定义，只是不再指定初始值：const int Person::age;

常实型 static const数据成员不可在类内初始化。一个好的解决方法是使用宏定义： #define age 30

常整型静态数据成员可以在类中直接初始化，而常实型静态数据成员不可以
```
class circle
{
int a; 　　　　　　　　　　　　　　　　　　　　 // 普通变量，不能在类中初始化
static int b; 　　　　　　　　　　　　　　　　　　// 静态变量，不能在类中初始化
static const int c=2; 　　　　　　　　　　　　　　// 静态常整型变量，可以在类中初始化
static const double PI=3.1416;//error C2864: //只有静态常量整型数据成员才可以在类中初始化
} ;

const int cicle::c ; //const static数据成员在类的定义体中出始化时，该数据成员仍必须在类的定义体外定义，只是不再指定初始值
```
b可以在类外进行初始化，且所有对象共享一个b的值：
int circle::b = 2;
double circle::PI = 3.1416;

知乎问答:static函数在头文件中定义有什么好处么？
没有好处，不要这么做。除非该头文件只会被一个翻译单元（translation unit）所使用，那么static是可用作表示内部链接（internal linkage）。不过这种「头文件」和一般所指的头文件不同，通常会使用.inc文件后缀.
为了内联. 一般定义成static inline.
隐患: 头文件中的 static 函数会在每个文件中生成一份代码，这造成代码冗余倒不是最大的问题，最大的问题是可能带来库文件与工程文件同一函数的代码的不一致性，这有风险。

# 2. const关键字

# 3. inline关键字

# 4. extern关键字

# 5. explicit关键字


# 6. volatile关键字

# 7. using关键字

# 8. 四种类型Cast


# 9. 指针


# 10. 函数指针


# 11. 回调函数


# 12. 线程

# 13. 进程


# 14. OOP六大原则

# 15. 宏函数
什么是宏函数
在Ｃ/C++语言中允许用一个标识符来表示一个字符串，称为宏，该字符串可以是常数、表达式、格式串等。在编译预处理时，对程序中所有出现的“宏名”，都用宏定义中的字符串去替换，这称为“宏替换”或“宏展开”。宏定义是由源程序中的宏定义命令完成的。宏替换是由预处理程序自动完成的。若字符串是表达式，则称之为函数式宏定义，

函数式宏定义与普通函数区别
我们以下面两行代码为例，展开描述：函数式宏定义：

普通函数：

MAX(a,b) { return a>b?a:b;}
宏函数的作用
几个比较经典的宏函数
1.
```
#define TO_PROG_LEN(mm) ((mm) / (settings.prog_length_units == CANON_UNITS_INCHES ? 25.4 : 1.0))
    
#define TO_PROG_ANG(deg) (Rad2Deg(deg))
    
#define CHK(bad, error_code)            \
do{                                        \
    if (bad) { return error_code; }        \
} while(0)
```
# 16. C++禁止对象拷贝复制
为什么有的时候需要禁止对象的拷贝复制
在类的设计里，不去实现拷贝构造和赋值操作不就完了吗？其实不行！C++会在背后偷偷的帮你现实一个默认的拷贝构造的版本，必须注意这个后门。

如何禁止对象的拷贝复制
```
class CPeople  
{  
    // ...  
private:  
    // 将复制相关的操作定义为私有  
     CPeople(){...};  
    const CPeople& operator=(const CPeople& rhis){...}  
}; 
``` 
这样的设计，可以部分的禁止的类的复制，但是对于友元函数和类成员函数来说，还是可以调用其相关的复制操作的。
```
class CPeople  
{  
    // ...  
private:  
    // 将复制相关的操作定义为私有  
     CPeople(); // 只声明不实现  
    const CPeople& operator=(const CPeople& rhis); // 只声明不实现  
}; 
``` 
将拷贝构造函数与赋值函数，声明为private，并且不给出实现。这样就实现了类复制的完全禁止
用户代码中的复制尝试将在编译时标记为错误，而成员函数与友元函数中的复制尝试将在链接时出现错误。
上面介绍的这种技术在你熟悉的std::iostream类中已经得到了很好的应用，诸如ios_base、basic_ios和sentry，都采用这样的方式不允许复制操作。

Boost为我们提供了另一种解决方式，这种方式更加完美，因为它可以将链接错误提前到编译时，毕竟早一点发现错误比晚发现要好。特意声明一个不可复制的类
```
boost::noncopyable
#ifndef BOOST_NONCOPYABLE_HPP_INCLUDED    
#define BOOST_NONCOPYABLE_HPP_INCLUDED        
namespace boost {    
//  Private copy constructor and copy assignment ensure classes derived from    
//  class noncopyable cannot be copied.    
//  Contributed by Dave Abrahams    
    namespace noncopyable_  // protection from unintended ADL    
    {    
    class noncopyable    
    {    
    protected:    
        noncopyable() {}    
        ~noncopyable() {}    
    private:  // emphasize the following members are private    
        noncopyable( const noncopyable& );    
        const noncopyable& operator=( const noncopyable& );    
    };    
    }    
    typedef noncopyable_::noncopyable noncopyable;      
} // namespace boost    
#endif  // BOOST_NONCOPYABLE_HPP_INCLUDED 
```
为了禁止拷贝对象，我们只需要让其私有继承自boost::noncopyable，
```
class student:private boost::noncopyable
{
......
}
```
当调用到派生类的拷贝构造函数或赋值函数进行复制时，不可避免的要调用基类对应的函数，因为这些操作是private，这样的操作会被编译器拒绝。
需要注意，多重继承有时会使空基类noncopyable优化失效，所以这不适合用于多重继承的情形。

另外，如果只是不想要使用默认的拷贝构造函数或赋值函数，可以使用C++11提供的delete，
C++11则使用delete关键字显式指示编译器不生成函数的默认版本。比如：
```
class MyClass
{
public:

MyClass()=default;
MyClass(const MyClass& )=delete;
......
}
```
当然，一旦函数被delete过了，那么重载该函数也是非法的，该函数我们习惯上称为删除函数。

# 17. C++虚析构函数
什么情况下声明为虚析构函数
并不是要把所有类的析构函数都写成虚函数。因为当类里面有虚函数的时候，编译器会给类添加一个虚函数表，里面来存放虚函数指针，这样就会增加类的存储空间。所以，只有当一个类被用来作为基类的时候，才把析构函数写成虚函数。

为什么基类的析构函数要声明成虚析构函数
在实现多态时，当用基类操作派生类，在析构时防止只析构基类而不析构派生类的状况发生。
直接的讲，C++中基类采用virtual虚析构函数是为了防止内存泄漏。具体地说，如果派生类中申请了内存空间，
并在其析构函数中对这些内存空间进行释放。假设基类中采用的是非虚析构函数，当删除基类指针指向的派生类对象时就不会触发动态绑定，
因而只会调用基类的析构函数，而不会调用派生类的析构函数。那么在这种情况下，派生类中申请的空间就得不到释放从而产生内存泄漏。
所以，为了防止这种情况的发生，C++中基类的析构函数应采用virtual虚析构函数。

为什么派生类中定义了析构函数来释放其申请的资源，但是并没有得到调用。
原因是基类指针指向了派生类对象，而基类中的析构函数却是非virtual的，而虚函数是动态绑定的基础。
现在析构函数不是virtual的，因此不会发生动态绑定，而是静态绑定，指针的静态类型为基类指针，因此在delete时候只会调用基类的析构函数，而不会调用派生类的析构函数。
这样，在派生类中申请的资源就不会得到释放，就会造成内存泄漏，这是相当危险的：如果系统中有大量的派生类对象被这样创建和销毁，就会有内存不断的泄漏，久而久之，系统就会因为缺少内存而崩溃。
也就是说，在基类的析构函数为非虚析构函数的时候，并不一定会造成内存泄漏；当派生类对象的析构函数中有内存需要收回，并且在编程过程中采用了基类指针指向派生类对象，
如为了实现多态，并且通过基类指针将该对象销毁，这时，就会因为基类的析构函数为非虚析构函数而不触发动态绑定，从而没有调用派生类的析构函数而导致内存泄漏。
因此，为了防止这种情况下内存泄漏的发生，最好将基类的析构函数写成virtual虚析构函数。

# 18. C/C++中的#if defined() 和 #ifdef的区别
1.使用#ifdef
如果使用#ifdef,则后面的内容不能加括号.If you use #ifdef syntax, remove the brackets.

2. #if defined() 和 #ifdef的区别
1. #ifdef只能使用单个条件,
2. #if defined(NAME)则可以组合多个条件
The difference between the two is that #ifdef can only use a single condition,while #if defined(NAME) can do compound conditionals.
```
    #ifndef __EXPORT_H__
    #define __EXPORT_H__

    #if (defined WIN32 || defined _WIN32 || defined WINCE) && defined INTERPRETER_LIBRARY
    #  define INTERP_API __declspec(dllexport)
    #else
    #  define INTERP_API
    #endif
    #endif // __EXPORT_H__
```
# 19. C++之throw
void ff(void)throw();
这样的函数声明只是告诉此函数的使用者说，我不会抛出异常，方便调用者捕捉异常。
而至于实现不实现异常捕获是你自己的事。函数是给别人用的，不是给自己用的。
