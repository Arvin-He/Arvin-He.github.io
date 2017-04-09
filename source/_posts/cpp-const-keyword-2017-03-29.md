---
title: C/C++之const关键字
date: 2017-03-29 14:34:05
tags: C/C++
categories: 编程
---

### 1. C中的const关键字
const是C语言中保留的一个关键字，它用来限定一个变量是只读的.
注意:在C语言中，用const修饰的变量必须在声明时进行初始化(用来修饰函数的形参除外).
```c
const int n;   //这种声明方式是错误的
const int n=5; //正确
void fun(const int n); //正确
const char a;  //错误
char * const p;  //错误
const char *p;  //正确(注意这种为什么是正确的),因为这里const是修饰p指向的变量,而不是指针变量p本身 
```
注意: 一旦一个变量被const修饰后，在程序中除初始化外对这个变量进行的赋值都是错误的。
总结:
- 在C语言中用const去修饰一个变量，表示这个变量是只读的，不可通过显式的调用a去修改a的值，并且此时a仍然是一个变 量，不能等同于常量;
- 要注意const在声明变量时所处的位置，位置不同，在意义上可能会有很大的不同。如果const在'*'左边，则表示指针指向的变量的值不可变;如果const在'*'右边，则表示指针的值是不可变的;

### 2. C++中的关键字
const修饰符把对象转变成常数对象，即用const进行修饰的变量的值在程序的任意位置将不能再被修改，就如同常数一样使用！任何修改该变量的尝试都会导致编译错误.
**注意：**常量在定以后就不能被修改，所以定义时必须初始化.
对于类中的const成员变量则必须通过初始化列表进行初始化
```cpp
class A
{
public:
    A(int i);
    
    const int &r;
private:
    const int a;
    static const int b;
};

const int A::b=10;
A::A(int i):a(i), r(a)
{}
```

#### const对象作用域
在全局作用域里定义非const变量时，它在整个程序中都可以访问，我们可以把一个非const变量定义在一个文件中，假设已经做了合适的声明，就可以在另外的文件中使用这个变量：
```cpp
// file1.cpp
int count;

// file2.cpp
extern int count;
++count; 
```
在全局作用域声明的const变量是定义该const变量的文件的局部变量。此const变量只存在于那个文件中，不能被其他文件访问。通过指定const变量为extern，就可以在整个程序中访问const对象。
```cpp
//file1.cpp
extern const int count = 10;  //定义的时候就要指定extern,假如在整个程序中访问的话

//file2.cpp
extern const int count;
for (int index=0; index!=count; ++index)
{...}

```
**注意:**非const变量默认为extern。要使const变量能够在其他文件中访问，必须在文件中**显式指定**它为extern。

#### const引用
const引用是指向const对象的引用
```cpp
const int count = 1024;
const int &refCount = count; //两者均为const对象
int &ref2 = count;       //错误!不能用非const引用指向const对象
```
const 引用可以初始化为不同类型的对象或者初始化为右值。如字面值常量：
```cpp
int i = 10;
//仅对const引用合法
const int &j = 10;
const int &k = j + i;
```
同样的初始化对于非const引用却是不合法的，而且会导致编译时错误。其原因非常微妙，值得解释一下。观察将引用绑定到不同的类型时所发生的事情，最容易理解上述行为
```cpp
int a = 5;
const int &b = a; //不合法
编译器会做如下转换:
int temp = a;
const int &b = temp;
```
注意：引用在内部存放的是一个对象的地址，它是该对象的别名。对于不可寻址的值，如文字常量，以及不同类型的对象，编译器为了实现引用，必须生成一个临时对象，引用实际上指向该对象，但用户不能访问它.
如果b不是const,修改b的同时也修改了temp,但不是a,而期望修改b的同时修改了a的程序员会发现a没有被修改,这会造成bug.所以仅允许const引用绑定到需要临时使用的值完全避免了这个问题，直接告诉程序员不能修改，因为const引用是只读的.(其实就是避免程序员心理预期产生反差)
注意：非const引用只能绑定到与该引用相同类型的对象。 const引用则可以绑定到不同但相关的类型的对象或绑定到右值。


#### const对象的动态数组
如果在自由存储区中创建的数组存储了内置类型的const对象，则必须为这个数组提供初始化： 因为数组元素都是const对象，无法赋值。实现这个要求的唯一方法是对数组做值初始化.
```cpp
const int *pci = new const int[100];  // error
const int *pci_ok = new const int[100]();  //OK
```
C++允许定义类类型的const数组，但该类类型必须提供默认构造函数：
```cpp
// 这里便会调用string类的默认构造函数初始化数组元素
const string *pcs = new string[100]
```

#### 指针和const限定符的关系
指针常量:即指针本身的值是不可改变的，而指针指向的变量的值是可以改变的;

常量指针:即指针指向的变量的值是不可改变的，而指针本身的值是可以改变的;

可以这样理解:因为指针本身也是一个变量，只不过指针存放的是地址而已，而一旦指针变成了常量，即指针本身的值是不可变的，此时指针只能指向固定的存储单元;指针一般会指向一个变量，如果该变量成为一个常量，那么该变量的值就不能被修改，即常量指针，指针指向的是一个不可变的变量。

#### 函数和const限定符的关系
1. 类中的const成员函数（常量成员函数）
在一个类中，任何不会修改数据成员的函数都应该声明为const类型。如果在编写const成员函数时，不慎修改了数据成员，或者调用了其它非const成员函数，编译器将指出错误，这无疑会提高程序的健壮性。使用const关键字进行说明的成员函数，称为常成员函数。只有常成员函数才有资格操作常量或常对象，没有使用const关键字说明的成员函数不能用来操作常对象。常成员函数说明格式如下：　<类型说明符> <函数名> (<参数表>) const;
其中，const是加在函数生明后面的类型修饰符，它是函数类型的一个组成部分，因此，在函数实现部分也要带const关键字。下面举一例子说明常成员函数的特征。
```cpp
class Stack
{
private:
  int m_num;
  int m_date[100];
public:
  void Push(int elem);
  int Pop(void);
  int GetCount() const; //定义为const函数
};

int Stack::GetCount() const
{
  ++m_num;  // error 编译错误,企图修改数据成员m_num
  Pop();    // error 编译错误,企图调用非const函数
  return m_num;
}
```
既然const是定义为const函数的组成部分，那么就可以通过添加const实现函数重载.
```cpp
class R
{
public:
  R(int r1, int r2)
  {
    R1 = r1;
    R2 = r2;
  }  

  void print();
  void print() const;
private:
  int R1, R2;
};

void R::print()
{
  cout<<R1;
}

void R::print() const
{
  cout<<R2;
}

void main()
{
  R a(5, 4);
  a.print();
  const R b(20, 52);
  b.print(); //const对象默认调用const成员函数
}

5
52
```

const 修饰函数返回值
const修饰函数返回值其实用的并不是很多，它的含义和const修饰普通变量以及指针的含义基本相同.
```cpp
const int func1(); //这个其实无意义,函数返回本身就是作为右值赋值.
// 调用时 const int *pValue = func2();
//func2可以看作一个变量,即指针内容不可变
const int *func2(); 
// 调用时 const int *pValue = func3();
//func3可以看作一个变量,即指针本身不可变
int* const func3();
```
一般情况下，函数的返回值为某个**对象**时，如果将其声明为const时，多用于操作符的重载。
通常不建议用const修饰函数的返回值类型为某个对象或对某个对象引用的情况。
原因如下：如果返回值为某个对象为const（const A test = A 实例）或某个对象的引用为const（const A& test = A实例） ，则返回值具有const属性，则返回实例只能访问类A中的公有（保护）数据成员和const成员函数，并且不允许对其进行赋值操作，这在一般情况下很少用到。  

const修饰函数参数

```cpp
//传递过来的参数在函数内不可以改变(无意义，因为Var本身就是形参)
void func(const int var);

//参数指针所指内容为常量不可变
void func2(const char* var);

// 参数指针本身为常量不可变(也无意义，因为char* Var也是形参)
void func3(char* const var);

//参数为引用，为了增加效率同时防止修改。修饰引用参数时：
void func4(const Class& var); //引用参数在函数内不可以改变
void func5(const TYPE& Var);  //引用参数在函数内为常量不可变
```
这样的一个const引用传递和最普通的函数按值传递的效果是一模一样的,他禁止对引用的对象的一切修改,唯一不同的是按值传递会先建立一个类对象的副本, 然后传递过去,而它直接传递地址,所以这种传递比按值传递更有效.另外只有引用的const传递可以传递一个临时对象,因为临时对象都是const属性, 且是不可见的,他短时间存在一个局部域中,所以不能使用指针,只有引用的const传递能够捕捉到这个家伙.

### const限定符和static的区别

1. const定义的常量在超出其作用域之后其空间会被释放，而static定义的静态常量在函数执行后不会释放其存储空间。
2. static表示的是静态的。类的静态成员函数、静态成员变量是和类相关的，而不是和类的具体对象相关的。即使没有具体对象，也能调用类的静态成员函数和成员变量。一般类的静态函数几乎就是一个全局函数，只不过它的作用域限于包含它的文件中。
3. 在C++中，static静态成员变量不能在类的内部初始化。在类的内部只是声明，定义必须在类定义体的外部，通常在类的实现文件中初始化，如：double Account::Rate=2.25; static关键字只能用于类定义体内部的声明中，定义时不能标示为static
4. 在C++中，const成员变量也不能在类定义处初始化，只能通过构造函数初始化列表进行，并且必须有构造函数。
5. const数据成员,只在某个对象生存期内是常量，而对于整个类而言却是可变的。因为类可以创建多个对象，不同的对象其const数据成员的值可以不同。所以不能在类的声明中初始化const数据成员，因为类的对象没被创建时，编译器不知道const数据成员的值是什么。
6. const数据成员的初始化只能在类的构造函数的初始化列表中进行。要想建立在整个类中都恒定的常量，应该用类中的枚举常量来实现，或者static const。
7. const成员函数主要目的是防止成员函数修改对象的内容。即const成员函数不能修改成员变量的值，但可以访问成员变量。当方法成员函数时，该函数只能是const成员函数。
8. static成员函数主要目的是作为类作用域的全局函数。不能访问类的非静态数据成员。类的静态成员函数没有this指针，这导致：1、不能直接存取类的非静态成员变量，调用非静态成员函数2、不能被声明为virtual 

#### 关于static、const、static cosnt、const static成员的初始化问题

1. 类里的const成员初始化
在一个类里建立一个const时，不能给他初值
```cpp
class foo
{
public:
  foo():i(100){}
private:
  const int i=100;  //error!!! 不能在类中初始化
};
//或者通过这样的方式初始化
foo::foo():i(100)
{

}
```

2. 类里的static成员初始化：
类中的static变量是属于类的，不属于某个对象，它在整个程序的运行过程中只有一个副本，因此不能在定义对象时 对变量进行初始化，就是不能用构造函数进行初始化，其正确的初始化方法是：
数据类型 类名::静态数据成员名=值
```cpp
class foo
{
public:
  foo():i(100){}
private:
  static int i;
};

int foo::i=20;
```
- 初始化在类体外进行,且前面不佳static,以免与一般静态变量或对象混淆
- 初始化时不加该成员的访问权限控制符
- 初始化时使用作用域操作符表明所属的类

3. 类里的static cosnt 和 const static成员初始化（这两种写法是一致的！！）

```cpp
class Test
{
public:
    static const int mask1;
    const static int mask2;
};

const Test::mask1 = 0xffff;
const Test::mask2 = 0xffff;
```
- 它们初始化没有区别,虽然一个是静态常量一个是常量静态
- 静态都将存储在全局变量区域,其实最后结果都一样
- 可能不同编译器处理不同,但最后结果都一样

例子:
```cpp
#include <iostream>
using namespace std;
class A
{
public:
  A(int a);
  static void print();  //静态成员函数
private:
  static int aa;  //静态数据成员的声明
  static const int count;  //常量静态数据成员(可以在构造函数中初始化)
  const int bb;  //常量数据成员
};

int A::aa=0; //静态成员定义+初始化
const int A::count=25; //静态常量成员定义+初始化
A::A(int a):bb(a)   //常量成员的初始化
{
  aa+=1;
}

void A::print()
{
  cout<<"count="<<count<<endl;
  cout<<"aa="<<aa<<endl;
}

void main()
{
  A a(10);
  A::print();  //通过类访问静态成员函数
  a.print();   //通过对象访问静态成员函数
}
```
#### const的难点
如果函数需要传入一个指针，面试官可能会问是否需要为该指针加上const，把const加在指针不同的位置有什么区别；如果写的函数需要传入的参数是一个复杂类型的实例，面试官可能会问传入值参数或者引用参数有什么区别，什么时候需要为传入的引用参数加上const。 const是用来声明一个常量的，当你不想让一个值被改变时就用const，const int max和int const max 是没有区别的，都可以。不涉及到指针const很好理解。一旦涉及到指针，则比较容易出问题。
```cpp
int b = 100;
const int *a = &b;   //[1]
int const *a = &b;  //[2]
int* const a = &b;  //[3]
const int* const a = &b;  //[4]
```
如果const位于星号的左侧，则const就是用来修饰指针所指向的变量，即指针指向的对象为常量；如果const位于星号的右侧，const就是修饰指针本身，即指针本身是常量。

因此，[1]和[2]的情况相同，都是指针所指向的内容为常量（const放在变量声明符的位置无关），这种情况下不允许对内容进行更改操作，如不能*a = 3 ；[3]为指针本身是常量，而指针所指向的内容不是常量，这种情况下不能对指针本身进行更改操作，如a++是错误的；[4]为指针本身和指向的内容均为常量。


参考:
* [C++中的const关键字](http://www.cnblogs.com/jiabei521/p/3335676.html)
* [浅谈C和C++中的const关键字](http://www.cnblogs.com/dolphin0520/archive/2011/04/18/2020248.html)