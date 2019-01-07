---
title: C/C++中const关键字总结
date: 2017-11-28 16:32:00
categories: 
- C/C++
tags:
- C
- C++
- const
- const对象
- const引用
- 常引用
- const指针
- 常量指针
- 指针常量
- 常成员函数
- const成员函数
- 函数重载
---
常类型是指使用类型修饰符`const`说明的类型，常类型的变量或对象的值是不能被更新的。不管出现在任何上下文都是为这个目的而服务的。    
## 0. 声明方式

 - 常变量：`const 类型说明符 变量名`
 例：`const int a=10;` 等价 `int const a=10;`
 - 常数组：`const 类型说明符 数组名[大小]`。
 例：`const int arr[3]={1,2,3};` 等价 `int const arr[3]={1,2,3};`
 - 常引用：`const 类型说明符& 引用名`
 例：`const int& a=x;`等价`int const &a=x;`
 - 指针常量：`const 类型说明符* 指针名`
 例：`const int* a = &b`等价`int const *a = &b` 
 - 常量指针：`类型说明符* const 指针名`
 例：`int* const a = &b`      
 - 常对象：`const 类名 对象名`
 例：`const test t;`等价于`test const t;`
 - 常成员函数：类体中：`类型说明符 函数名(形参) const` 类体外：`类型说明符 类名::函数名(形参) const`
 例：类体中：`int GetCount(void) const;` 类体外：`int Stack::GetCount(void) const;`

注：在常变量、常数组、常引用、常对象中，`const`与`类型说明符`或`类名`（其实类名是一种自定义的类型说明符） 的位置可以互换。
<!-- more --> 
## 1.const对象

（1）`const`修饰符把对象转变成常量对象，该对象的值不能再被修改，否则致编译错误：
```C
const int bufferSize = 512;
bufferSize = 0; //Error!
```
（2）因为常量对象在定义后就不能被修改，所以定义时必须初始化：
 
```C
const i, j=0; //Error，i未初始化
```
 
（3）`const`对象默认为文件的局部变量。
　　在全局作用域里定义`非const`变量时，它在整个程序中都可以访问，我们可以把一个`非const`变量定义在一个文件中（假设已经做了合适的声明），就可以在另外的文件中使用这个变量：

```C++
//file_1.cc
int counter; //definition
```
```C++
//file_2.cc
extern int counter; //use counter from file_1.h
++counter; //increments counter defined in file_1.h
```
　　在全局作用域声明的`const`变量是定义该对象的文件的局部变量。此变量只存在于那个文件中，不能被其他文件访问。通过指定`const`变量为`extern`，就可以在整个程序中访问`const`对象。
```C++
//file_1.cc
extern const int bugSize = 500;
```
```C++
//file_2.cc
extern const int bugSize; //use bufSize from file_1.h
for(int index=0;index != bufSize;++index){}
```
　　综上，`非const`变量默认为`extern`。要使`const`变量能够在其他文件中访问，必须在文件中显式地指定它为`extern`。

-----
 （待，放在类中，单独总结一下类）3.const 修饰类的数据成员
 对于类中的const成员变量必须通过初始化列表进行初始化，如下所示：
 ![](http://images.cnitblog.com/blog/460416/201310/06213502-292090335e124b14a9a0b5ae9d49f3e0.png)
const数据成员只在某个对象生存期内是常量，而对于整个类而言却是可变的。因为类可以创建多个对象，不同的对象其const数据成员的值可以不同。所以不能在类声明中初始化const数据成员，因为类的对象未被创建时，编译器不知道const 数据成员的值是什么。
```C++
class A{
 const int size = 100;    //错误
 int array[size];         //错误，未知的size
};
```
const数据成员的初始化只能在类的构造函数的初始化表中进行。
要想建立在整个类中都恒定的常量，应该用类中的枚举常量来实现.如:
```C++
class A{
enum {size1=100, size2 = 200 };
int array1[size1];
int array2[size2];
};
```
枚举常量不会占用对象的存储空间，他们在编译时被全部求值。但是枚举常量的隐含数据类型是整数，其最大值有限，且不能表示浮点数。

---

## 2.const对象的动态数组

如果我们在`自由存储区`中创建的数组存储了内置类型的`const对象`，则必须为这个数组提供初始化。

因为数组元素都是`const对象`，无法赋值。实现这个要求的唯一方法是对数组做值初始化：
```C++
//error
const int* pci_bad = new const int[100];
//ok
const int* pci_ok = new const int[100]();
```
C++允许定义`类类型`的`const数组`，但该`类类型`必须提供`默认构造函数`：
```C++
const string* pcs = new string[100];
```
这里便会调用`string类`的`默认构造函数`初始化数组元素。

## 3.const引用

（1）`const引用`是指向`const对象`的引用,将普通的引用绑定到`const对象`是不合法的：

```C++
const int iVal = 1024;
const int& refVal = iVal;  //两者均为const对象
int& refVal2=iVal;         //错误！不能使用非const引用指向const对象
```
`refVal2`是普通的`非const引用`，因此可以用来修改`refVal2`指向的对象的值,为防止这样的修改，需要规定将普通的引用绑定到`const对象`是不合法的。

（2）`const引用`可以初始化为`不同类型的对象`或者初始化为`右值`。

如字面值常量：
```C++
int i = 42;
//仅对const引用合法
const int& r2 = r+i;
const int& r = 42;
```
同样的初始化对于`非const引用`却是不合法的，而且会导致编译时错误。
观察将引用绑定到不同的类型时所发生的事情，最容易理解上述行为。对于下一段代码：
```C++
double dval = 3.14;
const int& ri = dval;
```
编译器会将这些代码转换为：
```C++
int temp = dval;
const int& ri = temp;
```
编译器会创建一个`int`类型的临时变量存储`dval`，然后将`ri`绑定到`temp`上。
注意：引用在内部存放的是一个对象的地址，它是该对象的别名。对于不可寻址的值，如`文字常量`，以及`不同类型的对象`，编译器为了实现引用，必须生成一个`临时对象`，引用实际上指向该对象，但用户不能访问它。
如果`ri`不是`const`，那么可以给`ri`赋一新值。这样做不会修改`dval`的，而是修改了`temp`。期望对`ri`赋值会修改`dval`的程序员会发现`dval`没有被修改。仅允许`const引用`绑定到需要临时使用的值完全避免了这个问题，直接告诉程序员不能修改，因为`const引用`是只读的。
注意：`非const引用`只能绑定到`与该引用相同类型的对象`。 `const引用`则可以绑定到`不同但相关的类型的对象`或绑定到`右值`。

## 4.const和指针
const限定符和指针结合起来常见的情况有以下几种：指针常量（指向const对象的指针）、常量指针（const指针）、指向常量的常指针（指向const对象的const指针）。
### 4.1 指针常量（指向const对象的指针）
（1）C++为了保证不允许使用指针改变所指的`const对象`的值的这个特性，强制要求这个指针也必须具备`const`特性：
```C++
const double* cptr;
```
这里`cptr`是一个指向`double类型`的`const对象`的指针，`const`先定义了`cptr`指向的对象的类型，而并非`cptr`本身，所以`cptr`本身并不是`const`。所以定义的时候并不需要对它进行初始，如果需要的话，允许给`cptr`重新赋值，让其指向另一个`const对象`。但不能通过`cptr`修改其所指对象的值。
```C++
*cptr=42; //error
```

（2）将一个`const对象`的地址赋给一个普通的`非const`指针会导致编译错误:
```C++
const double pi = 3.14;
double* ptr = &pi;        //error
const double* cptr = &pi; //ok
```
不能使用`void*`指针保存`const对象`的地址，必须使用`const void*`类型的指针保存`const`对象的地址。
```C++
const int universe = 42;
const void* cov = &universe; //ok
void* pv = &universe;        //error
```
（3）允许把`非const对象`的地址赋给指向`const对象`的指针，例如：
```C++
double dval = 3.14;
const double* cptr = &dval;
```
但是我们不能通过`cptr`指针来修改`dval`的值，即使它指向的是`非const对象`。
### 4.2 常指针(const指针)
C++中还提供了`const指针`——本身的值不能被修改。
```C++
int errNumb = 0;
int* const curErr = &errNumb; //curErr是一个const指针
```
我们可以从右往左把上述定义语句读作"指向int型对象的const指针"。与其他`const对象`一样，`const指针`的值不能被修改，这意味着不能使`curErr`指向其他对象。`const指针`也必须在定义的时候初始化:
```C++
curErr = curErr; //error！即使赋给其相同的值
```
 指针本身是`const`的事实并没有说明是否能用该指针修改其所指向的对象的值。指针对象的值能否修改完全取决于该对象的类型。
### 4.3 指向常量的常指针（指向const对象的const指针）
 如下可以这样定义：
```C++
const double pi = 3.1415926;
const double* const pi_ptr = &pi;
```
这样`pi_ptr`首先是一个`const指针`，然后其指向一个`const对象`。
###**4.4 小结**
（1）分清以下四种情况：
```C++
int b = 500; 
const int* a = &b  //1. 指针常量（指向常量的指针）
int const *a = &b  //2. 同1
int* const a = &b  //3. 常量指针（指针本身为常量，const指针）
const int* const a = &b //4.指向常量的常指针
```
（2）允许把`非const对象`的地址赋给指向`const对象`的指针,不允许把一个 `const对象`的地址赋给一个`非const对象`的指针。

## 5.const和函数
### 5.1 类中的const成员函数（常成员函数）
（1）使用`const`关键字进行说明的成员函数，称为`常成员函数`。在一个类中，任何不会修改数据成员的函数都应该声明为`const类型`。
对于`const成员函数`：
（1）只可读取数据成员，不可修改数据成员；
（2）不可调用其它`非const成员函数`。
只有`常成员函数`才有资格操作`常量`或`常对象`，没有使用`const`关键字说明的成员函数不能用来操作`常对象`。
其中，`const`是加在函数说明后面的类型修饰符，它是函数类型的一个组成部分，因此，在函数实现部分也要带`const`关键字。
下面举一例子说明常成员函数的特征:
```C++
class Stack
{
private:
    int m_num;
    int m_data[100];
public:
    void push(int elem);
    int pop(void);
    int GetCount(void) const; //定义为const成员函数
};
int Stack::GetCount(void) const
{
++m_num;  //编译错误，企图修改数据成员m_num
pop();    //编译错误，企图修改非const成员函数
return m_num;//正确，只能读取同一类中的数据成员的值
}
```

const对象的成员是不能修改的，而通过指针维护的对象确实可以修改的；
const对象只能访问const成员函数，而非const对象可以访问任意的成员函数，包括const成员函数。
const成员函数不可以修改对象的数据，不管对象是否具有const性质，编译时以是否修改成员数据为依据进行检查；

### 5.2 函数重载
既然`const`是定义为`const`函数的组成部分，那么就可以通过添加`const`实现函数重载，即：两个成员函数，名字和参数表都一样，但是一个是`const`，一个不是，算重载。
具体调用哪个函数是根据调用对象是`常量对象`还是`非常量对象`来决定的。`常量对象函数`调用`常量成员`；`非常量对象`调用`非常量成员函数`：
```C++
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
    int R1,int R2;
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
R a(5,4);
a.print();
const R b(20,52);
b.print();
}
```
输出结果为:5,52。
其中`print`成员函数就实现了重载。`const对象`默认调用`const成员函数`。

### 5.3 const修饰函数参数 
（1）传递过来的参数在函数内不可以改变(无意义，因为`var`本身就是形参)：
```C++
void func(const int var);
```
（2）参数指针所指内容为常量不可变：
```C++
void func(const char* var);
```
（3）参数指针本身为常量不可变(也无意义，因为`char* var`也是形参)
```C++
void func(char* const var);
```
（4）参数为引用，为了`增加效率`同时`防止修改`。修饰引用参数时：
```C++
void func(const Class& var); //引用参数在函数内不可以改变
void func(const TYPE& var);  //引用参数在函数内为常量不可变
```
  这样的一个`const引用`传递和最普通的函数`按值传递`的效果是一模一样的,他禁止对引用的对象的一切修改。唯一不同的是，`按值传递`会先建立一个类对象的副本, 然后传递过去,而它直接传递地址,所以这种传递比按值传递更有效。另外只有`引用的const传递`可以传递一个`临时对象`,因为`临时对象`都是`const属性`, 且是不可见的,他短时间存在一个局部域中,所以不能使用指针,只有`引用的const传递`能够捕捉到这个家伙.
  
### 5.4 const修饰函数返回值
`const`修饰函数返回值其实用的并不是很多，它的含义和`const`修饰`普通变量`以及指针的含义基本相同。如下所示：
```C++
//无意义，因为参数返回本身就是赋值
const int func1();

//调用时 const int* pVal = func2(); //必须这样可以理解
//可把func2()看成一个变量，即指针内容不可变
const int* func2();

//调用时 int* const pVal = func2(); //可以 int* pVal = func2()吧，不可理解
//可把func2()看成一个变量，即指针本身不可变
int* const func3();
```

（待）一般情况下，函数的返回值为某个对象时，如果将其声明为`const`时，多用于操作符的重载。

## 6.const限定符和static的区别（待总结完static来看）
const定义的常量在超出其作用域之后其空间会被释放，而static定义的静态常量在函数执行后不会释放其存储空间。
static表示的是静态的。类的静态成员函数、静态成员变量是和类相关的，而不是和类的具体对象相关的。即使没有具体对象，也能调用类的静态成员函数和成员变量。一般类的静态函数几乎就是一个全局函数，只不过它的作用域限于包含它的文件中。
在C++中，static静态成员变量不能在类的内部初始化。在类的内部只是声明，定义必须在类定义体的外部，通常在类的实现文件中初始化，如：double Account::Rate=2.25; static关键字只能用于类定义体内部的声明中，定义时不能标示为static
在C++中，const成员变量也不能在类定义处初始化，只能通过构造函数初始化列表进行，并且必须有构造函数。
const数据成员,只在某个对象生存期内是常量，而对于整个类而言却是可变的。因为类可以创建多个对象，不同的对象其const数据成员的值可以不同。所以不能在类的声明中初始化const数据成员，因为类的对象没被创建时，编译器不知道const数据成员的值是什么。
const数据成员的初始化只能在类的构造函数的初始化列表中进行。要想建立在整个类中都恒定的常量，应该用类中的枚举常量来实现，或者static const。
const成员函数主要目的是防止成员函数修改对象的内容。即const成员函数不能修改成员变量的值，但可以访问成员变量。当方法成员函数时，该函数只能是const成员函数。
static成员函数主要目的是作为类作用域的全局函数。不能访问类的非静态数据成员。类的静态成员函数没有this指针，这导致：1、不能直接存取类的非静态成员变量，调用非静态成员函数2、不能被声明为virtual 
![](http://images.cnitblog.com/blog/460416/201310/06213512-d6ae0722b2ae42f9b60ed2eff799c6be.png)
其中关于static、const、static cosnt、const static成员的初始化问题：

类里的const成员初始化
 在一个类里建立一个const时，不能给他初值
 ![](http://images.cnitblog.com/blog/460416/201310/06213513-3befd1d1772d455a8c46f0f095139d72.png)
 类里的static成员初始化：
 类中的static变量是属于类的，不属于某个对象，它在整个程序的运行过程中只有一个副本，因此不能在定义对象时 对变量进行初始化，就是不能用构造函数进行初始化，其正确的初始化方法是：
数据类型 类名::静态数据成员名=值
![](http://images.cnitblog.com/blog/460416/201310/06213513-2d8b85aa88ad47e1b6170dd1d960f5bb.png)
类里的static cosnt 和 const static成员初始化（这两种写法是一致的！！）
![](http://images.cnitblog.com/blog/460416/201310/06213514-e6e0265323ec44d88ab3331af53fb3e2.png)
   最后通过一个完整的例子展示以上结果：
   ![](http://images.cnitblog.com/blog/460416/201310/06213515-ec5f6c035b9748a1a5849853853867e0.png)

## 7.参考
1.C++ prime, Stanley
2.[C/C++中const关键字详解](http://www.cnblogs.com/yc_sunniwell/archive/2010/07/14/1777416.html)
3.[C++中const关键字的使用方法](http://www.cnblogs.com/jiabei521/p/3335676.html)
4.[C++ const的用法详解](http://blog.csdn.net/wangkai_123456/article/details/76598917)


