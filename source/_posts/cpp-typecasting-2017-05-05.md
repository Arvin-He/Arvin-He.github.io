---
title: C++之类型转换(Type Casting)
date: 2017-05-05 09:21:54
tags: [C/C++]
categories: 编程
---
### 类型转换分类
类型转换是将给定类型的表达式转换为另一种类型。C++中的转型可分为两种：隐式类型转换和显式类型转换。下面将详细介绍这两种转型操作，以及各自的适用场景，潜在问题，最终将总结使用类型转换操作应牢记的原则。

C风格的强制类型转换(Type Cast)很简单，不管什么类型的转换统统是：`TYPE b = (TYPE)a`
C++风格的类型转换提供了4种类型转换操作符来应对不同场合的应用。
```cpp
const_cast，字面上理解就是去const属性。
static_cast，命名上理解是静态类型转换。如int转换成char。
dynamic_cast，命名上理解是动态类型转换。如子类和父类之间的多态类型转换。
reinterpret_cast，仅仅重新解释类型，但没有进行二进制的转换。
```
4种类型转换的格式，如：TYPE B = static_cast(TYPE)(a)。

### 隐式类型转换
隐式类型转换是C中的遗留物，在C++中并不推荐使用（C++有专门的转型操作符，见下文的显式转型）。将某种类型的对象拷贝到另一种不同类型的对象中时就会发生隐式转型。比如异型赋值，返回值（函数声明的返回值与代码块实际返回值不同的情况下），按值传递异型参数等情况均会发生隐式类型转换。
```cpp
short a = 128;
int b;
b = a;
```
如上所示，short 类型的对象被赋值给 int 型的对象，这是C++语言内建支持的标准转换。
情形一：标准转换支持**数值类型**，**bool**以及**某些指针**之间相互转换。注意：某些转换可能会导致精度丢失，比如从 long 转换到 int。
情形二：可被单参调用（只有一个参数或多个参数但至少从第二个参数起均带有缺省值）的构造函数或隐式类型转换操作符也会引起隐式类型转换。比如：
```cpp
class A {}; 
class B
{
    public: B (A a) {}
    public: B (int c, int d = 0);
    public: operator double() const; 
};

A a;
B b1 = a;
B b2 = 10;
B b3;
double d;
d = 10 + b3;
```
上面的代码里就存在只带有一个参数的构造函数，多个参数但至少从第二个参数起均带有缺省值以及用户自定义类型转换操作符这三种情况。
隐式类型转换是件麻烦事，它们很可能导致错误或非预期的函数被调用；此外 C++ 也不能在一个转换过程中连续进行多于一次的用户自定义转换操作（即情形二中的转换），如下所示：接上面的代码，A （类型的对象，后略）可被隐式转换为 B，B 可被隐式转换为 C，但 A 却非常不合逻辑地不可被隐式转换为 C。
```cpp
class C
{
    public: C(B b) {};
}

A a;
C c;
c = a; // 错误！
```
因此应该尽量避免隐式类型转换，为此 C++ 提供了关键字 explicit 来规避可被单参调用的构造函数引起的隐式类型转换。但**标准转换**以及**隐式类型转换操作符**引起的转换只能由程序员来小心处理了。当然 C++ 语言还是提供了必要的工具，这些工具就是下面要讲的显式类型转换关键字：static\_cast, const\_cast, dynamic\_cast 以及 reinterpret\_cast。

### 显示类型转换
C++ 是一门强类型转换，因此不同自定义类型之间的转换必须进行显式转换，基础数据类型也可以进行显式转换。
```cpp
 short a = 10;
 int b;
 b = (int) a;    // c-like cast notation
 b = int (a);    // functional notation
```
以上是基础数据类型之间进行传统的强制类型转换。这种强制类型转换可以在两种指向不同类型对象的指针之间进行，这很可能是相当危险的事情。所以 C++ 提供四种转换操作符来细分显式类型转换：
```cpp
static_cast <new_type> (expression)
const_cast <new_type> (expression)
dynamic_cast <new_type> (expression)
reinterpret_cast <new_type> (expression)
```

### static_cast
static\_cast 很像 C 语言中的旧式类型转换。它能进行基础类型之间的转换，也能将带有可被单参调用的构造函数或用户自定义类型转换操作符的类型转换，还能在存有继承关系的类之间进行转换（即可将基类转换为子类，也可将子类转换为基类），还能将 non-const对象转换为 const对象（注意：反之则不行，那是const_cast的职责）
1. 基类和子类之间转换：其中子类指针转换成父类指针是安全的；但父类指针转换成子类指针是不安全的。(基类和子类之间的动态类型转换建议用dynamic_cast)
2. 基本数据类型转换。enum, struct, int, char, float等。static_cast不能进行无关类型（如非基类和子类）指针之间的转换。
3. 把空指针转换成目标类型的空指针。
4. 把任何类型的表达式转换成void类型。
5. static\_cast不能去掉类型的const、volitale属性(用const\_cast)。
```cpp
double d = 3.14159265;
int i = static_cast<int>(d);
class A {};
class B
{
public:
    B (A a) {};
}; 
A a;
B b = static_cast<B>(a);
 
class CBase {};
class CDerived: public CBase {};
CBase * a = new CBase;
CDerived * b = static_cast<CDerived *>(a);
```
**注意：**static\_cast 转换时并不进行运行时安全检查，所以是非安全的，很容易出问题。因此 C++ 引入 dynamic\_cast 来处理安全转型。

### dynamic_cast 
dynamic\_cast 主要用来在**继承体系**中的**安全向下**转型。它能安全地将指向基类的指针转型为指向子类的指针或引用，并获知转型动作成功是否。如果转型失败会返回null（转型对象为指针时）或抛出异常（转型对象为引用时）。
dynamic\_cast 会动用运行时信息（RTTI）来进行类型安全检查，因此 dynamic\_cast 存在一定的效率损失。（曾见过属于优化代码80/20法则中的20那一部分的一段游戏代码，起初使用的是 dynamic\_cast，后来被换成 static\_cast 以提升效率，当然这仅是权宜之策，并非好的设计。）
有条件转换，动态类型转换，运行时类型安全检查(转换失败返回NULL)：
1. 安全的基类和子类之间转换。
2. 必须要有虚函数。
3. 相同基类不同子类之间的交叉转换。但结果是NULL。
```cpp
class BaseClass 
{
public:
  int m_iNum;
  virtualvoid foo(){}; //基类必须有虚函数。保持多台特性才能使用dynamic_cast
};
 
class DerivedClass: public BaseClass 
{
public:
  char*m_szName[100];
  void bar(){};
};
 
BaseClass* pb =new DerivedClass();
DerivedClass *pd1 = static_cast<DerivedClass *>(pb); //子类->父类，静态类型转换，正确但不推荐
DerivedClass *pd2 = dynamic_cast<DerivedClass *>(pb); //子类->父类，动态类型转换，正确
 
BaseClass* pb2 =new BaseClass();
DerivedClass *pd21 = static_cast<DerivedClass *>(pb2); //父类->子类，静态类型转换，危险！访问子类m_szName成员越界
DerivedClass *pd22 = dynamic_cast<DerivedClass *>(pb2); //父类->子类，动态类型转换，安全的。结果是NULL
```

```cpp
class CBase { };
class CDerived: public CBase { };
CBase b;
CBase* pb;
CDerived d;
CDerived* pd;
pb = dynamic_cast<CBase*>(&d);     // ok: derived-to-base
pd = dynamic_cast<CDerived*>(&b);  // error: base-to-derived
```
上面的代码中最后一行 VS2010 会报如下错误：error C2683: 'dynamic\_cast' : 'CBase' is not a polymorphic type IntelliSense: the operand of a runtime dynamic\_cast must have a polymorphic class type.
这是因为 dynamic\_cast 只有在基类带有虚函数的情况下才允许将基类转换为子类。
```cpp
class CBase
 {
    virtual void dummy() {}
 };
 class CDerived: public CBase
 {
     int a;
 };

  int main ()
 {
    CBase * pba = new CDerived;
    CBase * pbb = new CBase;
    CDerived * pd1, * pd2;
    pd1 = dynamic_cast<CDerived*>(pba);
    pd2 = dynamic_cast<CDerived*>(pbb);
   return 0;
 }
```
结果是：上面代码中的 pd1 不为 null,而 pd2 为 null。
dynamic\_cast 也可在 null 指针和指向其他类型的指针之间进行转换，也可以将指向类型的指针转换为 void 指针（基于此，我们可以获取一个对象的内存起始地址 const void * rawAddress = dynamic_cast<const void *> (this);）。

### const_cast
const\_cast 可去除对象的常量性（const），它还可以去除对象的易变性（volatile）。
const\_cast 的唯一职责就在于此，若将 const_cast 用于其他转型将会报错。
```cpp
 void print (char * str)
 {
   cout << str << endl;
 }
 int main ()
 {
   const char * c = "hello, world";
   print ( const_cast<char *> (c) );
   return 0;
 }
```

### reinterpret_cast
reinterpret\_cast 用来执行低级转型，如将一个 int 指针强转为 int。其转换结果与编译平台息息相关，不具有可移植性，因此在一般的代码中不常见到它。reinterpret\_cast 常用的一个用途是转换函数指针类型，即可以将一种类型的函数指针转换为另一种类型的函数指针，但这种转换可能会导致不正确的结果。
总之，reinterpret\_cast 只用于底层代码，一般我们都用不到它，如果你的代码中使用到这种转型，务必明白自己在干什么。

1. 转换的类型必须是一个指针、引用、算术类型、函数指针或者成员指针。
2. 在比特位级别上进行转换。它可以把一个指针转换成一个整数，也可以把一个整数转换成一个指针（先把一个指针转换成一个整数，在把该整数转换成原类型的指针，还可以得到原先的指针值）。但不能将非32bit的实例转成指针。
3. 最普通的用途就是在函数指针类型之间进行转换。
4. 很难保证移植性。

### typeid：获取表达式的类型
typeid 定义在标准头文件`<typeinfo>`中，用于获取表达式的类型，它返回一个数据类型或类名字的字符串。当 typeid 用于自定义类型时，它使用 RTTI 信息来获取对象的动态类型。基于 typeid，我们可以构建出比较对象（动态）类型的操作。

### 使用原则：尽量避免类型转换操作；优先使用 C++ 风格的转型
1. 鉴于类型转换的隐蔽，不安全，易引起非预期的函数调用，对象切割等诸多问题，应该尽量避免类型转换操作。如使用 explicit 声明可被单参调用的构造函数，按引用传递参数或返回值，使用虚函数机制等等可避免类型转换；
2. 若类型转换不可避免，优先使用 C++ 风格的新式类型转换。C++ 风格的类型转换一则易于辨识，二则有着其特有惯用手法，遵循这些惯用手法好处多多。


### 总结
去const属性用const_cast;
基本类型转换用static_cast;
多态类之间的类型转换用daynamic_cast;
不同类型的指针类型转换用reinterpret_cast;

### 参考

- [参考文章](http://www.cppblog.com/kesalin/archive/2012/10/28/type_cast.html)
- [参考文章](http://www.cnblogs.com/goodhacker/archive/2011/07/20/2111996.html)