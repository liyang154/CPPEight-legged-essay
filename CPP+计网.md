# C++

## 编程基础

### 全局变量、局部变量、静态全局变量、静态局部变量的区别

C++ 变量根据定义的位置的不同的生命周期，具有不同的作用域，作用域可分为 6 种：全局作用域，局部作用域，语句作用域，类作用域，命名空间作用域和文件作用域。

从作用域看：

全局变量：具有全局作用域。全局变量只需在一个源文件中定义，就可以作用于所有的源文件。当然，其他不包含全局变量定义的源文件需要用 extern 关键字再次声明这个全局变量。
静态全局变量：具有文件作用域。它与全局变量的区别在于如果程序包含多个文件的话，它作用于定义它的文件里，不能作用到其它文件里，即被 static 关键字修饰过的变量具有文件作用域。这样即使两个不同的源文件都定义了相同名字的静态全局变量，它们也是不同的变量。
局部变量：具有局部作用域。它是自动对象（auto），在程序运行期间不是一直存在，而是只在函数执行期间存在，函数的一次调用执行结束后，变量被撤销，其所占用的内存也被收回。
静态局部变量：具有局部作用域。它只被初始化一次，自从第一次被初始化直到程序运行结束都一直存在，它和全局变量的区别在于全局变量对所有的函数都是可见的，而静态局部变量只对定义自己的函数体始终可见。
从分配内存空间看：

静态存储区：全局变量，静态局部变量，静态全局变量。
栈：局部变量。
说明：

静态变量和栈变量（存储在栈中的变量）、堆变量（存储在堆中的变量）的区别：静态变量会被放在程序的静态数据存储区（.data 段）中（静态变量会自动初始化），这样可以在下一次调用的时候还可以保持原来的赋值。而栈变量或堆变量不能保证在下一次调用的时候依然保持原来的值。
静态变量和全局变量的区别：静态变量用 static 告知编译器，自己仅仅在变量的作用范围内可见。

### 全局变量定义在头文件有什么问题

如果在头文件中定义全局变量，当该头文件被多个文件 `include` 时，该头文件中的全局变量就会被定义多次，导致重复定义，因此不能再头文件中定义全局变量。

### 如何限制类的对象只能在堆上创建？如何限制对象只能在栈上创建？

说明：C++ 中的类的对象的建立分为两种：静态建立、动态建立。

静态建立：由编译器为对象在栈空间上分配内存，直接调用类的构造函数创建对象。例如：A a;
动态建立：使用 new 关键字在堆空间上创建对象，底层首先调用 operator new() 函数，在堆空间上寻找合适的内存并分配；然后，调用类的构造函数创建对象。例如：A *p = new A();

**限制对象只能建立在堆上**：

最直观的思想：**避免直接调用类的构造函数，因为对象静态建立时，会调用类的构造函数创建对象**。但是直接将类的构造函数设为私有并不可行，因为当构造函数设置为私有后，不能在类的外部调用构造函数来构造对象，只能用 new 来建立对象。但是由于 new 创建对象时，底层也会调用类的构造函数，将构造函数设置为私有后，那就无法在类的外部使用 new 创建对象了。因此，这种方法不可行。

解决方法 1：

将析构函数设置为私有。原因：静态对象建立在栈上，是由编译器分配和释放内存空间，编译器为对象分配内存空间时，会对类的非静态函数进行检查，即编译器会检查析构函数的访问性。当析构函数设为私有时，编译器创建的对象就无法通过访问析构函数来释放对象的内存空间，因此，编译器不会在栈上为对象分配内存。

```c++
class A
{
public:
    A() {}
    void destory()
    {
        delete this;
    }

private:
    ~A(){}
};
```

该方法存在的问题：

用 new 创建的对象，通常会使用 delete 释放该对象的内存空间，但此时类的外部无法调用析构函数，因此类内必须定义一个 destory() 函数，用来释放 new 创建的对象。

无法解决继承问题，因为如果这个类作为基类，析构函数要设置成 virtual，然后在派生类中重写该函数，来实现多态。但此时，析构函数是私有的，派生类中无法访问。

解决方法 2：

构造函数设置为 protected，并提供一个 public 的静态函数来完成构造，而不是在类的外部使用 new 构造；将析构函数设置为 protected。原因：类似于单例模式，也保证了在派生类中能够访问析构函数。通过调用 create() 函数在堆上创建对象。

```c++
class A
{
protected:
    A() {}
    ~A() {}

public:
    static A *create()
    {
        return new A();
    }
    void destory()
    {
        delete this;
    }
};
```

**限制对象只能建立在栈上：**

解决方法：**将 operator new() 设置为私有**。原因：当对象建立在堆上时，是采用 new 的方式进行建立，其底层会调用 operator new() 函数，因此只要对该函数加以限制，就能够防止对象建立在堆上。

```c++
class A
{
private:
    void *operator new(size_t t) {}    // 注意函数的第一个参数和返回值都是固定的
    void operator delete(void *ptr) {} // 重载了 new 就需要重载 delete
public:
    A() {}
    ~A() {}
};
```

### 结构体内存对齐问题？

- 结构体内成员按照声明顺序存储，第一个成员地址和整个结构体地址相同。
- 未特殊说明时，按结构体中size最大的成员对齐（若有double成员），按8字节对齐。
- static变量不属于结构体成员

### static作用是什么？在C和C++中有何区别？

- static可以修饰局部变量（静态局部变量）、全局变量（静态全局变量）和函数，被修饰的变量存储位置在静态区。对于静态局部变量，相对于一般局部变量其生命周期长，直到程序运行结束而非函数调用结束，且只在第一次被调用时定义；对于静态全局变量，相对于全局变量其可见范围被缩小，只能在本文件中可见；修饰函数时作用和修饰全局变量相同，都是为了限定访问域。
- C++的static除了上述两种用途，还可以修饰类成员（静态成员变量和静态成员函数），静态成员变量和静态成员函数不属于任何一个对象，是所有类实例所共有。注意：类的静态成员函数中只能访问静态成员变量或者静态成员函数，不能将静态成员函数定义成虚函数
- static的数据记忆性可以满足函数在不同调用期的通信，也可以满足同一个类的多个实例间的通信。
- 未初始化时，static变量默认值为0。

```c++
#include<iostream>
using namespace std;

class A
{
private:
    int var;
    static int s_var; // 静态成员变量
public:
    void show()
    {
        cout << s_var++ << endl;
    }
    static void s_show()
    {
        cout << s_var << endl;
		// cout << var << endl; // error: invalid use of member 'A::a' in static member function. 静态成员函数不能调用非静态成员变量。无法使用 this.var
        // show();  // error: cannot call member function 'void A::show()' without object. 静态成员函数不能调用非静态成员函数。无法使用 this.show()
    }
};
int A::s_var = 1;  // 静态成员变量在类外进行初始化赋值，默认初始化为 0

int main()
{
    
    // cout << A::sa << endl;    // error: 'int A::sa' is private within this context
    A ex;
    ex.show();
    A::s_show();
}
```

### static 在类中使用的注意事项（定义、初始化和使用）

static 静态成员变量：

1.静态成员变量是在类内进行声明，在类外进行定义和初始化，在类外进行定义和初始化的时候不要出现 static 关键字和private、public、protected 访问规则。
2.静态成员变量相当于类域中的全局变量，被类的所有对象所共享，包括派生类的对象。
3.静态成员变量可以作为成员函数的参数，而普通成员变量不可以。
4.返回函数中的静态局部变量在离开函数的作用域之后，变量不会销毁，返回到主函数中，该变量依然存在，从而使程序得到正确的运行结果。但是，该静态局部变量直到程序运行结束后才销毁，浪费内存空间。

```c++
#include <iostream>
using namespace std;

int * fun(int tmp){
    static int var = 10;
    var *= tmp;
    return &var;
}

int main() {
    cout << *fun(5) << endl;
    return 0;
}

/*
运行结果：
50
*/
```

```c++
#include <iostream>
using namespace std;

class A
{
public:
    static int s_var;
    int var;
    void fun1(int i = s_var); // 正确，静态成员变量可以作为成员函数的参数
    void fun2(int i = var);   //  error: invalid use of non-static data member 'A::var'
};
int main()
{
    return 0;
}
```

4.静态数据成员的类型可以是所属类的类型，而普通数据成员的类型只能是该类类型的指针或引用。

**static 静态成员函数：**

1、静态成员函数不能调用非静态成员变量或者非静态成员函数，因为静态成员函数没有 this 指针。静态成员函数做为类作用域的全局函数。
2、静态成员函数不能声明成虚函数（virtual）、const 函数和 volatile 函数。

### 为什么静态成员函数没有this指针

this指针是相当于一个类的实例的指针，this是用来操作对象实例的内容的，既然静态成员函数和变量都是独立于类的实例对象之外的，他就不能用this指针。

### 为什么静态成员函数不能声明成const

一个静态成员函数访问的值是其参数、静态数据成员和全局变量，而这些数据都不是对象状态的一部分。而对成员函数中使用关键字const是表明：**函数不会修改该函数访问的目标对象的数据成员。**既然一个静态成员函数根本不访问非静态数据成员，那么就没必要使用const了。

### 为什么静态成员函数不能声明成virtual

 虚函数依靠vptr和vtable来处理。vptr是一个指针，在类的构造函数中创建生成，并且只能用**this指针**来访问它， 对于静态成员函数，它没有this指针，所以无法访问vptr. 这就是为何static函数不能为virtual.

### 为什么静态成员函数不能声明成volatile

道理和const类似。因为static成员函数没有this指针。

### static全局变量和局部全局变量的异同

相同点：

存储方式：普通全局变量和 static 全局变量都是静态存储方式。
不同点：

作用域：普通全局变量的作用域是整个源程序，当一个源程序由多个源文件组成时，普通全局变量在各个源文件中都是有效的；静态全局变量则限制了其作用域，即只在定义该变量的源文件内有效，在同一源程序的其它源文件中不能使用它。由于静态全局变量的作用域限于一个源文件内，只能为该源文件内的函数公用，因此可以避免在其他源文件中引起错误。
初始化：静态全局变量只初始化一次，防止在其他文件中使用。

### const 作用及用法

作用：

- const 修饰成员变量，定义成 const 常量，相较于宏常量，可进行类型检查，节省内存空间，提高了效率。
- const 修饰函数参数，使得传递过来的函数参数的值不能改变。
- const 修饰成员函数，使得成员函数不能修改任何类型的成员变量（mutable 修饰的变量除外），也不能调const 成员函数，因为非 const 成员函数可能会修改成员变量。
  在类中的用法：

const 成员变量：

- const 成员变量只能在类内声明、定义，在构造函数初始化列表中初始化。

- const 成员变量只在某个对象的生存周期内是常量，对于整个类而言却是可变的，因为类可以创建多个对象，不同类的 const 成员变量的值是不同的。因此不能在类的声明中初始化 const 成员变量，类的对象还没有创建，编译器不知道他的值。

const 成员函数：

- 不能修改成员变量的值，除非有 mutable 修饰；只能访问成员变量。
- 不能调用非常量成员函数，以防修改成员变量的值。

### 结构体和类的区别？

- 结构体的默认限定符是public；类是private。
- class 继承默认是 private 继承，而 struct 继承默认是 public 继承
- class 可以使用模板，而 struct 不能

### malloc和new的区别？

- malloc和free是标准库函数，支持覆盖；new和delete是运算符，并且支持重载。
- malloc仅仅分配内存空间，free仅仅回收空间，不具备调用构造函数和析构函数功能，用malloc分配空间存储类的对象存在风险；new和delete除了分配回收功能外，还会调用构造函数和析构函数。
- malloc和free返回的是void类型指针（必须进行类型转换），new和delete返回的是具体类型指针。
- 分配失败：new抛出bac_alloc异常，malloc分配失败返回NULL
- malloc分配空间时需要指定申请内存的大小，new会调用构造函数，不用指定内存的大小
- 对于自定义的类型，new 首先调用 operator new() 函数申请空间（底层通过 malloc 实现），然后调用构造函数进行初始化，最后返回自定义类型的指针；delete 首先调用析构函数，然后调用 operator delete() 释放空间（底层通过 free 实现）。malloc、free 无法进行自定义类型的对象的构造和析构。
- `new` 操作符从自由存储区上为对象动态分配内存，而 `malloc` 函数从堆上动态分配内存。（自由存储区不等于堆）

### malloc的原理

malloc 的原理:

当开辟的空间小于 128K 时，调用 brk() 函数，通过移动 _enddata 来实现；
当开辟空间大于 128K 时，调用 mmap() 函数，通过在虚拟地址空间中开辟一块内存空间来实现。
malloc 的底层实现：

brk() 函数实现原理：向高地址的方向移动指向数据段的高地址的指针 _enddata。
mmap 内存映射原理：
进程启动映射过程，并在虚拟地址空间中为映射创建虚拟映射区域；
调用内核空间的系统调用函数 mmap()，实现文件物理地址和进程虚拟地址的一一映射关系；
进程发起对这片映射空间的访问，引发缺页异常，实现文件内容到物理内存（主存）的拷贝。

### delete和delete[]区别？

- delete只会调用一次析构函数。
- delete[]会调用数组中每个元素的析构函数。

`delete` 的实现原理：

- 首先执行该对象所属类的析构函数；
- 进而通过调用 `operator delete` 的标准库函数来释放所占的内存空间。

### C 和 C++ struct 的区别？

在 C 语言中 struct 是用户自定义数据类型；在 C++ 中 struct 是抽象数据类型，支持成员函数的定义。
C 语言中 struct 没有访问权限的设置，是一些变量的集合体，不能定义成员函数；C++ 中 struct 可以和类一样，有访问权限，并可以定义成员函数。
C 语言中 struct 定义的自定义数据类型，在定义该类型的变量时，需要加上 struct 关键字，例如：struct A var;，定义 A 类型的变量；而 C++ 中，不用加该关键字，例如：A var;

C++ 是在 C 语言的基础上发展起来的，为了与 C 语言兼容，C++ 中保留了 `struct`。

### struct 和 union 的区别

说明：union 是联合体，struct 是结构体。
区别：

联合体和结构体都是由若干个数据类型不同的数据成员组成。使用时，联合体只有一个有效的成员；而结构体所有的成员都有效。
对联合体的不同成员赋值，将会对覆盖其他成员的值，而对于结构体的对不同成员赋值时，相互不影响。
联合体的大小为其内部所有变量的最大值，按照最大类型的倍数进行分配大小；结构体分配内存的大小遵循内存对齐原则。

```c++
#include <iostream>
using namespace std;

typedef union
{
    char c[10];
    char cc1; // char 1 字节，按该类型的倍数分配大小
} u11;

typedef union
{
    char c[10];
    int i; // int 4 字节，按该类型的倍数分配大小
} u22;

typedef union
{
    char c[10];
    double d; // double 8 字节，按该类型的倍数分配大小
} u33;

typedef struct s1
{
    char c;   // 1 字节
    double d; // 1（char）+ 7（内存对齐）+ 8（double）= 16 字节
} s11;

typedef struct s2
{
    char c;   // 1 字节
    char cc;  // 1（char）+ 1（char）= 2 字节
    double d; // 2 + 6（内存对齐）+ 8（double）= 16 字节
} s22;

typedef struct s3
{
    char c;   // 1 字节
    double d; // 1（char）+ 7（内存对齐）+ 8（double）= 16 字节
    char cc;  // 16 + 1（char）+ 7（内存对齐）= 24 字节
} s33;

int main()
{
    cout << sizeof(u11) << endl; // 10
    cout << sizeof(u22) << endl; // 12
    cout << sizeof(u33) << endl; // 16
    cout << sizeof(s11) << endl; // 16
    cout << sizeof(s22) << endl; // 16
    cout << sizeof(s33) << endl; // 24

    cout << sizeof(int) << endl;    // 4
    cout << sizeof(double) << endl; // 8
    return 0;
}
```

### 指针和引用区别？

   - 引用只是别名，不占用具体存储空间，只有声明没有定义；指针是具体变量，需要占用存储空间。
   - 引用在声明时必须初始化为另一变量，一旦出现必须为typename refname &varname形式；指针声明和定义可以分开，可以先只声明指针变量而不初始化，等用到时再指向具体变量。
   - 引用一旦初始化之后就不可以再改变（变量可以被引用为多次，但引用只能作为一个变量引用）；指针变量可以重新指向别的变量。
   - 不存在指向空值的引用，必须有具体实体；但是存在指向空值的指针。

### 宏定义和函数有何区别？

- 宏在**编译**时完成替换，之后被替换的文本参与编译，相当于直接插入了代码，**运行时不存在函数调用，执行起来更快**；函数调用在运行时需要跳转到具体调用函数。
- 宏函数属于在结构中插入代码，**没有返回值**；函数调用具有返回值。
- 宏函数参数没有类型，不进行类型检查；函数参数具有类型，需要检查类型。
- 宏函数不要在最后加分号。

### 宏定义和const区别？

- **宏（发生在预处理时）替换发生在编译阶段之前**，属于文本插入替换；const作用发生于编译过程中。
- 宏不检查类型；const会检查数据类型。
- 宏定义的数据没有分配内存空间，只是插入替换掉；const定义的变量只是值不能改变，但要分配内存空间。
- `define` 定义的宏常量不能调试，因为在预编译阶段就已经进行替换了；`const` 定义的常量可以进行调试。

### 宏定义和typedef区别？

- 宏主要用于定义常量及书写复杂的内容；typedef主要用于定义类型别名。
- 宏替换发生在编译阶段之前，属于文本插入替换；typedef是编译的一部分。
- 宏不检查类型；typedef会检查数据类型。
- 宏不是语句，不在在最后加分号；typedef是语句，要加分号标识结束。
- 注意对指针的操作，typedef char * p_char和#define p_char char *区别巨大。

```c++
Typedef int * pint；

#define PINT int *

Const pint p；//p是指针常量，相当于 int * const p，不可更改，p指向的内容可以更改，;

Const PINT p；//p是常量指针，相当于 const int *p；或 int const *p，p可以更改，p指向的内容不能更改，；


pint s1, s2; //s1和s2都是int型指针
PINT s3, s4; //相当于int * s3，s4；只有一个是指针。
```

### 宏定义和内联函数(inline)区别？

- 在使用时，宏只做简单字符串替换（编译前）。而内联函数可以进行参数类型检查（编译时），且具有返回值。
- 内联函数本身是函数，强调函数特性，具有重载等功能。
- 内联函数可以作为某个类的成员函数，这样可以使用类的保护成员和私有成员，一般，类的内部定义了函数体的函数，被默认为内联函数，而不管是否有inline关键字。而当一个表达式涉及到类保护成员或私有成员时，宏就不能实现了。
- 宏定义容易产生歧义，不能访问对象的私有成员。

### 用宏实现比较大小

```c++
#include <iostream>
#define MAX(X, Y) ((X)>(Y)?(X):(Y))
#define MIN(X, Y) ((X)<(Y)?(X):(Y))
using namespace std;

int main ()
{
    int var1 = 10, var2 = 100;
    cout << MAX(var1, var2) << endl;
    cout << MIN(var1, var2) << endl;
    return 0;
}
/*
程序运行结果：
100
10
*/
```

### 内联函数

只有当函数只有 10 行甚至更少时才将其定义为内联函数.

定义: 当函数被声明为内联函数之后, 编译器会将其内联展开, 而不是按通常的函数调用机制进行调用.
优点: 当函数体比较小的时候, 内联该函数可以令目标代码更加高效. 对于存取函数以及其它函数体比较短, 性能关键的函数, 鼓励使用内联.
缺点: 滥用内联将导致程序变慢. 内联可能使目标代码量或增或减, 这取决于内联函数的大小. 内联非常短小的存取函数通常会减少代码大小, 但内联一个相当大的函数将戏剧性的增加代码大小. 现代处理器由于更好的利用了指令缓存, 小巧的代码往往执行更快。
结论: 一个较为合理的经验准则是, 不要内联超过 10 行的函数. 谨慎对待析构函数, 析构函数往往比其表面看起来要更长, 因为有隐含的成员和基类析构函数被调用!
另一个实用的经验准则: 内联那些包含循环或 switch 语句的函数常常是得不偿失 (除非在大多数情况下, 这些循环或 switch 语句从不被执行).
有些函数即使声明为内联的也不一定会被编译器内联, 这点很重要; 比如虚函数和递归函数就不会被正常内联. 通常, 递归函数不应该声明成内联函数.(递归调用堆栈的展开并不像循环那么简单, 比如递归层数在编译时可能是未知的, 大多数编译器都不支持内联递归函数). 

内联函数不能为虚函数，原因在于虚表机制需要一个真正的函数地址，而内联函数展开以后，就不是一个函数，而是一段简单的代码（多数C++对象模型使用虚表实现多态，对此标准提供支持），可能有些内联函数会无法内联展开，而编译成为函数。

inlined意味着编译期将调用端的调用动作被函数本体取代，若无法知道哪个函数该被调用时，编译器没法将该函数加以inlining。

工作原理：

内联函数不是在调用时发生控制转移关系，而是在编译阶段将函数体嵌入到每一个调用该函数的语句块中，编译器会将程序中出现内联函数的调用表达式用内联函数的函数体来替换。
普通函数是将程序执行转移到被调用函数所存放的内存地址，当函数执行完后，返回到执行此函数前的地方。转移操作需要保护现场，被调函数执行完后，再恢复现场，该过程需要较大的资源开销。

### 条件编译#ifdef, #else, #endif作用？

- 可以通过加#define，并通过#ifdef来判断，将某些具体模块包括进要编译的内容。

- 用于子程序前加#define DEBUG用于程序调试。

- 应对硬件的设置（机器类型等）。

- 条件编译功能if也可实现，但条件编译可以减少被编译语句，从而减少目标程序大小。

### 区别以下几种变量？



```c
const int a;
int const a;
const int *a;
int *const a;
```

- int const a和const int a均表示定义常量类型a。

- const int *a，其中a为指向int型变量的指针，const在 * 左侧，表示a指向不可变常量。(看成const (*a)，对引用加const)

- int *const a，依旧是指针类型，表示a为指向整型数据的常指针。(看成const(a)，对指针const)

### 常量指针和指针常量区别？

- 常量指针是一个指针，读成常量的指针，指向一个只读变量。如int const *p或const int *p。指向常量的指针变量称为常量指针。因为常量指针本质是指针，并且这个指针是一个指向常量的指针，指针指向的变量的值不可通过该指针修改，但是指针的指向可以改变。

```c
int a，b；
 const int *p=&a //常量指针
//那么分为一下两种操作
*p=9;//操作错误
p=&b;//操作成功
```

- 指针常量是一个不能给改变指向的指针。如int *const p。

```c
int a，b；
int * const p=&a //指针常量
//那么分为一下两种操作
*p=9;//操作成功
p=&b;//操作错误
```

### volatile有什么作用？

- volatile定义变量的值是易变的，每次用到这个变量的值的时候都要去重新读取这个变量的值，而不是读寄存器内的备份。如果没有volatile关键字，则编译器可能优化读取和存储，可能暂时使用寄存器中的值，如果这个变量由别的程序更新了的话，将出现不一致的现象。

- 多线程中被几个任务共享的变量需要定义为volatile类型。

volatile不具有原子性。例如你让一个volatile的integer自增（i++），其实要分成3步：1）读取volatile变量值到local； 2）增加变量的值；3）把local的值写回，让其它的线程可见。

volatile 对编译器的影响：使用该关键字后，编译器不会对相应的对象进行优化，即不会将变量从内存缓存到寄存器中，防止多个线程有可能使用内存中的变量，有可能使用寄存器中的变量，从而导致程序错误。

### volatile的使用场景以及能否和const一起使用

使用 volatile 关键字的场景：

当多个线程都会用到某一变量，并且该变量的值有可能发生改变时，需要用 volatile 关键字对该变量进行修饰；
中断服务程序中访问的变量或并行设备的硬件寄存器的变量，最好用 volatile 关键字修饰。

volatile 关键字和 const 关键字可以同时使用，某种类型可以既是 volatile 又是 const ，同时具有二者的属性。

### 什么是常引用？

- 常引用可以理解为常量指针，形式为const typename & refname = varname。

- **常引用下，原变量值不会被别名所修改**。

- 原变量的值可以通过原名修改。

- 常引用通常用作只读变量别名或是形参传递。

### 数组名和指针（这里为指向数组首元素的指针）区别？

- 二者均可通过增减偏移量来访问数组中的元素。
- 数组名不是真正意义上的指针，可以理解为常指针，所以数组名没有自增、自减等操作。
- 当数组名当做形参传递给调用函数后，就失去了原有特性，退化成一般指针，多了自增、自减操作，但sizeof运算符不能再得到原数组的大小了。

### 野指针是什么？

- 也叫空悬指针，不是指向null的指针，是指向垃圾内存的指针。

- 产生原因及解决办法：
  - 指针变量未及时初始化 => 定义指针变量及时初始化，要么置空。
  - 指针free或delete之后没有及时置空 => 释放操作后立即置空。

### 内存泄漏-memory leak

内存泄露是指：内存泄漏也称作"存储渗漏"，用动态存储分配函数动态开辟的空间，在使用完毕后未释放，结果导致一直占据该内存单元。直到程序结束。(其实说白了就是该内存空间使用完毕之后未回收)即所谓内存泄漏。

解决方式：

（1）重载new和delete运算符

思路：重载的new运算符（返回值为void *,重载函数调用malloc）和delete运算符(返回值为void,重载函数调用free)，为了判断申请的内存是否最后被释放，我们需要创建一个链表，当前申请一块内存的时候加入到链表中，释放一块空间的时候，从链表中删除和释放内存首地址相同的结点就可以，最后理想的情况为链表为空，如果不为空，那就说明发生了内存泄漏。

（2）智能指针

智能指针是 C++ 中已经对内存泄漏封装好了一个工具，可以直接拿来使用。

### 内存泄漏检测

valgrind 是一套 Linux 下，开放源代码（GPL V2）的仿真调试工具的集合，包括以下工具：

Memcheck：内存检查器（valgrind 应用最广泛的工具），能够发现开发中绝大多数内存错误的使用情况，比如：使用未初始化的内存，使用已经释放了的内存，内存访问越界等。
Callgrind：检查程序中函数调用过程中出现的问题。
Cachegrind：检查程序中缓存使用出现的问题。
Helgrind：检查多线程程序中出现的竞争问题。
Massif：检查程序中堆栈使用中出现的问题。
Extension：可以利用 core 提供的功能，自己编写特定的内存调试工具。

Memcheck 能够检测出内存问题，关键在于其建立了两个全局表：

Valid-Value 表：对于进程的整个地址空间中的每一个字节（byte），都有与之对应的 8 个 bits ；对于 CPU 的每个寄存器，也有一个与之对应的 bit 向量。这些 bits 负责记录该字节或者寄存器值是否具有有效的、已初始化的值。
Valid-Address 表：对于进程整个地址空间中的每一个字节（byte），还有与之对应的 1 个 bit，负责记录该地址是否能够被读写。
检测原理：

当要读写内存中某个字节时，首先检查这个字节对应的 Valid-Address 表中对应的 bit。如果该 bit 显示该位置是无效位置，Memcheck 则报告读写错误。
内核（core）类似于一个虚拟的 CPU 环境，这样当内存中的某个字节被加载到真实的 CPU 中时，该字节在 Valid-Value 表对应的 bits 也被加载到虚拟的 CPU 环境中。一旦寄存器中的值，被用来产生内存地址，或者该值能够影响程序输出，则 Memcheck 会检查 Valid-Value 表对应的 bits，如果该值尚未初始化，则会报告使用未初始化内存错误。

### 堆和栈的区别？

- 申请方式不同。

  - 栈由系统自动分配。

  - 堆由程序员手动分配。
- 申请大小限制不同。

  - 栈顶和栈底是之前预设好的，大小固定，可以通过ulimit -s查看，由ulimit -s修改。默认8M

  - 堆向高地址扩展，是不连续的内存区域，大小可以灵活调整，容易造成碎片。
- 申请效率不同。

  - 栈由系统分配，速度快，不会有碎片。
  - 堆由程序员分配，速度慢，且会有碎片。
- 对于堆来讲，生长方向是向上的，也就是向着内存地址增加的方向；对于栈来讲，它的生长方式是向下的，是向着内存地址减小的方向增长（栈底是高地址）。

### sizeof 和 strlen 的区别

（1）strlen 是头文件 <cstring> 中的函数，sizeof 是 C++ 中的运算符。对于指针sizeof在32位机器上是4， 在64位机器上是8.

（2）strlen 测量的是字符串的实际长度（其源代码如下），以 \0 结束。而 sizeof 测量的是字符数组的分配大小。

strlen源码：

```c++
size_t strlen(const char* str) {
    size_t len = 0;
    while(*str ++) {
        ++len;
    }
    return len;
}
```

sizeof源码：

非数组的sizeof:
#defne _sizeof(T) ( (size_t)((T*)0 + 1))
数组的sizeof:
#define array_sizeof(T)   ( (size_t)(&T+1)  - (size_t)(&T)  )
（3）若字符数组 arr 作为函数的形参，sizeof(arr) 中 arr 被当作字符指针来处理，strlen(arr) 中 arr 依然是字符数组，从下述程序的运行结果中就可以看出。

```c++
#include <iostream>
#include <cstring>

using namespace std;

void size_of(char arr[])
{
    cout << sizeof(arr) << endl; // warning: 'sizeof' on array function parameter 'arr' will return size of 'char*' .
    cout << strlen(arr) << endl; 
}

int main()
{
    char arr[20] = "hello";
    size_of(arr); 
    return 0;
}
/*
输出结果：
8                       这是64位机器输出
5
*/
```

（4）strlen 本身是库函数，因此在程序运行过程中，计算长度；而 sizeof 在编译时，计算长度；

（5）sizeof 的参数可以是类型，也可以是变量；strlen 的参数必须是 char* 类型的变量。

### sizeof(1==1) 在 C 和 C++ 中分别是什么结果？

```c
//c语言
#include<stdio.h>

void main(){
    printf("%d\n", sizeof(1==1));
}

/*
运行结果：
4
*/
```

```c++
//c++
#include <iostream>
using namespace std;

int main() {
    cout << sizeof(1==1) << endl;
    return 0;
}

/*
1
*/
```

C语言
sizeof（1 == 1） === sizeof（1）按照整数处理，所以是4字节，这里也有可能是8字节（看操作系统）
C++
因为有bool 类型
sizeof（1 == 1） == sizeof（true） 按照bool类型处理，所以是1个字节

### explicit 的作用（如何避免编译器进行隐式类型转换）

作用：用来声明类构造函数是显示调用的，而非隐式调用，可以阻止调用构造函数时进行隐式转换。只可用于修饰单参构造函数，因为无参构造函数和多参构造函数本身就是显示调用的，再加上 explicit 关键字也没有什么意义。

隐式转换：

```c++
#include <iostream>
#include <cstring>
using namespace std;

class A
{
public:
    int var;
    A(int tmp)
    {
        var = tmp;
    }
};
int main()
{
    A ex = 10; // 发生了隐式转换   A temp(10); ex = temp;
    return 0;
}
```

上述代码中，A ex = 10; 在编译时，进行了隐式转换，将 10 转换成 A 类型的对象，然后将该对象赋值给 ex，等同于如下操作：

为了避免隐式转换，可用 explicit 关键字进行声明：

```c++
#include <iostream>
#include <cstring>
using namespace std;

class A
{
public:
    int var;
    explicit A(int tmp)
    {
        var = tmp;
        cout << var << endl;
    }
};
int main()
{
    A ex(100);
    A ex1 = 10; // error: conversion from 'int' to non-scalar type 'A' requested
    return 0;
}
```

### memcpy 函数的底层原理

```c++
void *memcpy(void *dst, const void *src, size_t size)
{
    char *psrc;
    char *pdst;

    if (NULL == dst || NULL == src)
    {
        return NULL;
    }

    if ((src < dst) && (char *)src + size > (char *)dst) // 出现地址重叠的情况，自后向前拷贝
    {
        psrc = (char *)src + size - 1;
        pdst = (char *)dst + size - 1;
        while (size--)
        {
            *pdst-- = *psrc--;
        }
    }
    else
    {
        psrc = (char *)src;
        pdst = (char *)dst;
        while (size--)
        {
            *pdst++ = *psrc++;
        }
    }

    return dst;
}
```

### strcpy 函数有什么缺陷？

底层实现：

```c++
char * strcpy(char *a,char *b)
{ while((*(a++)=*(b++))!=0);return a;}
```

strcpy 函数的缺陷：strcpy 函数不检查目的缓冲区的大小边界，而是将源字符串逐一的全部赋值给目的字符串地址起始的一块连续的内存空间，同时加上字符串终止符，会导致其他变量被覆盖。

```c++
#include <iostream>
#include <cstring>
using namespace std;

int main()
{
    int var = 0x11112222;
    char arr[10];
    cout << "Address : var " << &var << endl;
    cout << "Address : arr " << &arr << endl;
    strcpy(arr, "hello world!");
    cout << "var:" << hex << var << endl; // 将变量 var 以 16 进制输出
    cout << "arr:" << arr << endl;
    return 0;
}

/*
Address : var 0x23fe4c
Address : arr 0x23fe42
var:11002164
arr:hello world!
*/
```

说明：从上述代码中可以看出，变量 var 的后六位被字符串 "hello world!" 的 "d!\0" 这三个字符改变，这三个字符对应的 ascii 码的十六进制为：\0(0x00)，!(0x21)，d(0x64)。
原因：变量 arr 只分配的 10 个内存空间，通过上述程序中的地址可以看出 arr 和 var 在内存中是连续存放的，但是在调用 strcpy 函数进行拷贝时，源字符串 "hello world!" 所占的内存空间为 13，因此在拷贝的过程中会占用 var 的内存空间，导致 var的后六位被覆盖。

### 强制类型转换有哪几种？

static_cast：用于数据的强制类型转换，强制将一种数据类型转换为另一种数据类型。

1、用于基本数据类型的转换。

2、用于类层次之间的基类和派生类之间 指针或者引用 的转换（不要求必须包含虚函数，但必须是有相互联系的类），进行上行转换（派生类的指针或引用转换成基类表示）是安全的；进行下行转换（基类的指针或引用转换成派生类表示）由于没有动态类型检查，所以是不安全的，最好用 dynamic_cast 进行下行转换。

3、可以将空指针转化成目标类型的空指针。

4、可以将任何类型的表达式转化成 void 类型。

const_cast：强制去掉常量属性，不能用于去掉变量的常量性，只能用于去除指针或引用的常量性，将常量指针转化为非常量指针或者将常量引用转化为非常量引用（注意：表达式的类型和要转化的类型是相同的）。

reinterpret_cast：改变指针或引用的类型、将指针或引用转换为一个足够长度的整型、将整型转化为指针或引用类型。

dynamic_cast：

1、其他三种都是编译时完成的，动态类型转换是在程序运行时处理的，运行时会进行类型检查。

2、只能用于带有虚函数的基类或派生类的指针或者引用对象的转换，转换成功返回指向类型的指针或引用，转换失败返回 NULL；不能用于基本数据类型的转换。

3、在向上进行转换时，即派生类类的指针转换成基类类的指针和 static_cast 效果是一样的，（注意：这里只是改变了指针的类型，指针指向的对象的类型并未发生改变）。

4、在下行转换时，基类的指针类型转化为派生类类的指针类型，只有当要转换的指针指向的对象类型和转化以后的对象类型相同时，才会转化成功.

### 如何判断结构体是否相等？能否用 memcmp 函数判断结构体相等？

需要重载操作符 == 判断两个结构体是否相等，不能用函数 memcmp 来判断两个结构体是否相等，因为 memcmp 函数是逐个字节进行比较的，而结构体存在内存空间中保存时存在字节对齐，字节对齐时补的字节内容是随机的，会产生垃圾值，所以无法比较。

```c++
#include <iostream>

using namespace std;

struct A
{
    char c;
    int val;
    A(char c_tmp, int tmp) : c(c_tmp), val(tmp) {}

    friend bool operator==(const A &tmp1, const A &tmp2); //  友元运算符重载函数
};

bool operator==(const A &tmp1, const A &tmp2)
{
    return (tmp1.c == tmp2.c && tmp1.val == tmp2.val);
}

int main()
{
    A ex1('a', 90), ex2('b', 80);
    if (ex1 == ex2)
        cout << "ex1 == ex2" << endl;
    else
        cout << "ex1 != ex2" << endl; // 输出
    return 0;
}
```

### 参数传递时，值传递、引用传递、指针传递的区别？

参数传递的三种方式：

- 值传递：形参是实参的拷贝，函数对形参的所有操作不会影响实参。
- 指针传递：本质上是值传递，只不过拷贝的是指针的值，拷贝之后，实参和形参是不同的指针，通过指针可以间接的访问指针所指向的对象，从而可以修改它所指对象的值。
- 引用传递：当形参是引用类型时，我们说它对应的实参被引用传递。



## 面向对象基础

### 面向对象三大特性？

- 封装性：数据和代码捆绑在一起，避免外界干扰和不确定性访问。
- 继承性：让某种类型对象获得另一个类型对象的属性和方法。
- 多态性：同一事物表现出不同事物的能力，即向不同对象发送同一消息，不同的对象在接收时会产生不同的行为（重载实现编译时多态，虚函数实现运行时多态）。

### public/protected/private的区别？

- public的变量和函数在类的内部外部都可以访问。
- protected的变量和函数只能在类的内部和其派生类中访问。
- private修饰的元素只能在类内访问。

### 类对象存储空间

（1）空类

空类也会被实例化，所以编译器会给空类隐含的添加一个字节，这样空类实例化之后就有了独一无二的地址了。所以空类的sizeof为1。

（2）含有static变量

```c++
class CBase 
{ 
	int a; 
	char p; 
}; 
```

sizeof(CBase)=8;

记得对齐的问题。int 占4字节//注意这点和struct的对齐原则很像！！！！！
char占一字节，补齐3字节

（3）含有虚函数和函数

```c++
class CBase 
{ 
public: 
	CBase(void); 
	virtual ~CBase(void); 
private: 
	int  a; 
	char *p; 
};
```

sizeof(CBase)=12；

C++ 类中有虚函数的时候有一个指向虚函数的指针（vptr），在32位系统分配指针大小为4字节。无论多少个虚函数，只有这一个指针，4字节。//注意一般的函数是没有这个指针的，而且也不占类的内存。

（4）子类

```c++
class CChild : public CBase 
{ 
public: 
	CChild(void); 
	~CChild(void); 

	virtual void test();
private: 
	int b; 
}; 
```

sizeof(CChild)=16

可见子类的大小是本身成员变量的大小加上父类的大小。//其中有一部分是虚拟函数表的原因，一定要知道父类子类共享一个虚函数指针。

（5）总结

空的类是会占用内存空间的，而且大小是1，原因是C++要求每个实例在内存中都有独一无二的地址。

（一）类内部的成员变量：

- 普通的变量：是要占用内存的，但是要注意对齐原则（这点和struct类型很相似）。
- static修饰的静态变量：不占用内容，原因是编译器将其放在全局变量区。

（二）类内部的成员函数：

- 普通函数：不占用内存。
- 虚函数：要占用4个字节，用来指定虚函数的虚拟函数表的入口地址。所以一个类的虚函数所占用的地址是不变的，和虚函数的个数是没有关系的

### C++空类有哪些成员函数?

- 首先，空类大小为1字节。

- 默认函数有：
  - 缺省的构造函数
  - 拷贝构造函数
  - 析构函数
  - 赋值运算符
  - 两个取址运算符
  
  ```c++
  #include <iostream>
  using namespace std;
  /*
  class A
  {}; 该空类的等价写法如下：
  */
  class A
  {
  public:
      A(){};                                       // 缺省构造函数
      A(const A &tmp){};                           // 拷贝构造函数
      ~A(){};                                      // 析构函数
      A &operator=(const A &tmp){};                // 赋值运算符
      A *operator&() { return this; };             // 取址运算符
      const A *operator&() const { return this; }; // 取址运算符（const 版本）
  };
  
  int main()
  {
      A *p = new A(); 
      cout << "sizeof(A):" << sizeof(A) << endl; // sizeof(A):1
      delete p;       
      return 0;
  }
  ```

### 构造函数能否为虚函数，析构函数呢？

- 析构函数：
  - 析构函数可以为虚函数，并且一般情况下基类析构函数要定义为虚函数。

  - 将可能被继承的父类的析构函数设置为虚函数，可以保证当我们new一个子类，然后使用积累指针指向该子类对象，释放基类指针时可以释放掉子类的空间，防止内存泄漏。

  - 析构函数可以是纯虚函数，含有纯虚函数的类是抽象类，此时不能被实例化。但派生类中可以根据自身需求重新改写基类中的纯虚函数。


注意：默认的析构函数并不是虚函数，是因为虚函数需要额外的虚函数表和虚函数指针，占用额外的内存。而对于不会被继承的类来说，其析构函数如果是虚函数，就会浪费内存。因此C++默认的析构函数不是虚函数，只有当需要当做父类时，设置为虚函数。

**构造函数不能是虚函数的原因**

1、构造一个对象的时候，必须知道对象的实际类型，而虚函数行为是在运行期间确定实际类型的。而在构造一个对象时，由于对象还未构造成功。编译器无法知道对象的实际类型。

2、**虚函数的执行依赖于虚函数表。**虚函数表指针在对象创建时期初始化，而此时对象还没有创建，也就没有内存空间，所以无法指向虚函数表，所以构造函数不能是虚函数。

### 构造函数调用顺序，析构函数呢？

构造函数的调用顺序：当建立一个对象时，首先调用基类的构造函数，然后调用下一个派生类的构造函数，依次类推，直至到达最底层的目标派生类的构造函数为止。

析构函数的调用顺序：当删除一个对象时，首先调用该派生类的析构函数，然后调用上一层基类的析构函数，依次类推，直到到达最顶层的基类的析构函数为止。

### 拷贝构造函数中深拷贝和浅拷贝区别？

- 深拷贝时，当被拷贝对象存在动态分配的存储空间时，需要先动态申请一块存储空间，然后逐字节拷贝内容。
- 浅拷贝仅仅是拷贝指针字面值。
- 当使用浅拷贝时，如果原来的对象调用析构函数释放掉指针所指向的数据，则会产生空悬指针。因为所指向的内存空间已经被释放了。浅拷贝造成内存的重复释放。

拷贝构造函数和赋值运算符重载的区别？

- 拷贝构造函数是函数，赋值运算符是运算符重载。

- 拷贝构造函数会生成新的类对象，赋值运算符不能。

- 拷贝构造函数是直接构造一个新的类对象，所以在初始化对象前不需要检查源对象和新建对象是否相同；赋值运算符需要上述操作并提供两套不同的复制策略，另外赋值运算符中如果原来的对象有内存分配则需要先把内存释放掉。

- 形参传递是调用拷贝构造函数（调用的被赋值对象的拷贝构造函数），但并不是所有出现"="的地方都是使用赋值运算符，如下：

```c
Student s;
Student s1 = s;    // 调用拷贝构造函数
Student s2;
s2 = s;    // 赋值运算符操作
```

注意拷贝构造函数不能是值传递，而是引用传递。因为在值传递时，会进行形参的复制，然后再次调用形参的拷贝构造函数，这样又会进行值传递，这是一个死循环。

### 虚函数和纯虚函数区别？

- 虚函数是为了实现动态编联产生的，目的是通过基类类型的指针指向不同对象时，自动调用相应的、和基类同名的函数（使用同一种调用形式，既能调用派生类又能调用基类的同名函数）。虚函数需要在基类中加上virtual修饰符修饰，因为virtual会被隐式继承，所以子类中相同函数都是虚函数。当一个成员函数被声明为虚函数之后，其派生类中同名函数自动成为虚函数，在派生类中重新定义此函数时要求函数名、返回值类型、参数个数和类型全部与基类函数相同。
- 纯虚函数只是相当于一个接口名，但含有纯虚函数的类不能够实例化。

### 覆盖、重载和隐藏的区别？

 - 覆盖是派生类中重新定义的函数，其函数名、参数列表（个数、类型和顺序）、返回值类型和父类完全相同，只有函数体有区别。派生类虽然继承了基类的同名函数，但用派生类对象调用该函数时会根据对象类型调用相应的函数。覆盖只能发生在类的成员函数中。
 - 隐藏是指派生类函数屏蔽了与其同名的函数，这里仅要求基类和派生类**函数同名即可**。其他状态同覆盖。可以说隐藏比覆盖涵盖的范围更宽泛，毕竟参数不加限定。
 - 重载是具有相同函数名但参数列表不同（个数、类型或顺序）的两个函数（不关心返回值），当调用函数时根据传递的参数列表来确定具体调用哪个函数。重载可以是同一个类的成员函数也可以是类外函数。

**重写和重载的区别：**

范围区别：对于类中函数的重载或者重写而言，重载发生在同一个类的内部，重写发生在不同的类之间（子类和父类之间）。
参数区别：重载的函数需要与原函数有相同的函数名、不同的参数列表，不关注函数的返回值类型；重写的函数的函数名、参数列表和返回值类型都需要和原函数相同，父类中被重写的函数需要有 virtual 修饰。
virtual 关键字：重写的函数基类中必须有 virtual关键字的修饰，重载的函数可以有 virtual 关键字的修饰也可以没有。

**隐藏和重写，重载的区别：**

范围区别：隐藏与重载范围不同，隐藏发生在不同类中。
参数区别：隐藏函数和被隐藏函数参数列表可以相同，也可以不同，但函数名一定相同；当参数不同时，无论基类中的函数是否被 virtual 修饰，基类函数都是被隐藏，而不是重写。

**重载：**

```c++
class A
{
public:
    void fun(int tmp);
    void fun(float tmp);        // 重载 参数类型不同（相对于上一个函数）
    void fun(int tmp, float tmp1); // 重载 参数个数不同（相对于上一个函数）
    void fun(float tmp, int tmp1); // 重载 参数顺序不同（相对于上一个函数）
    int fun(int tmp);            // error: 'int A::fun(int)' cannot be overloaded 错误：注意重载不关心函数返回类型
};
```

**隐藏**：

```c++
#include <iostream>
using namespace std;

class Base
{
public:
    void fun(int tmp, float tmp1) { cout << "Base::fun(int tmp, float tmp1)" << endl; }
};

class Derive : public Base
{
public:
    void fun(int tmp) { cout << "Derive::fun(int tmp)" << endl; } // 隐藏基类中的同名函数
};

int main()
{
    Derive ex;
    ex.fun(1);       // Derive::fun(int tmp)
    ex.fun(1, 0.01); // error: candidate expects 1 argument, 2 provided
    return 0;
}
```

**覆盖：**

```c++
#include <iostream>
using namespace std;

class Base
{
public:
    virtual void fun(int tmp) { cout << "Base::fun(int tmp) : " << tmp << endl; }
};

class Derived : public Base
{
public:
    virtual void fun(int tmp) { cout << "Derived::fun(int tmp) : " << tmp << endl; } // 重写基类中的 fun 函数
};
int main()
{
    Base *p = new Derived();
    p->fun(3); // Derived::fun(int) : 3
    return 0;
}
```

### 在main执行之前执行的代码可能是什么？

- 全局对象的构造函数

### 什么是虚指针？

- 虚指针或虚函数指针是虚函数的实现细节。
- 虚指针指向虚表结构。

### 重载和函数模板的区别？

- 重载需要多个函数，这些函数彼此之间函数名相同，但参数列表中参数数量和类型不同。在区分各个重载函数时我们并不关心函数体。
- 模板函数是一个通用函数，函数的类型和形参不直接指定而用虚拟类型来代表。但只适用于参个数相同而类型不同的函数。

### this指针是什么？

- this指针是类的指针，指向对象的首地址。
- this指针只能在成员函数中使用，在全局函数、静态成员函数中都不能用this。
- this指针只有在成员函数中才有定义，且存储位置会因编译器不同有不同存储位置。

### 类模板是什么？

- 用于解决多个功能相同、数据类型不同的类需要重复定义的问题。
- 在建立类时候使用template及任意类型标识符T，之后在建立类对象时，会指定实际的类型，这样才会是一个实际的对象。
- 类模板是对一批仅数据成员类型不同的类的抽象，只要为这一批类创建一个类模板，即给出一套程序代码，就可以用来生成具体的类。

### 构造函数和析构函数调用时机？

- 全局范围中的对象：构造函数在所有函数调用之前执行，在主函数执行完调用析构函数。
- 局部自动对象：建立对象时调用构造函数，离开作用域时调用析构函数。
- 动态分配的对象：建立对象时调用构造函数，调用释放时调用析构函数。
- 静态局部变量对象：建立时调用一次构造函数，主函数结束时调用析构函数。

### 虚函数指针创建时机

虚函数表指针：**虚函数表指针随对象走，它发生在对象运行期，当对象创建的时候，虚函数表表指针位于该对象所在内存的最前面。**使用虚函数是，虚函数表指针指向虚函数表中的函数地址即可实现多态。

虚函数表：虚函数表是在编译期间就已经确定，且虚函数表存放虚函数的地址也是在创建时被确定。

虚函数表属于类，类的所有对象共享这个类的虚函数表。虚函数表由编译器在编译时生成，保存在常量区的只读数据段。

### 智能指针

#### 分类

智能指针是为了解决动态内存分配时带来的内存泄漏以及多次释放同一块内存空间而提出的。C++11 中封装在了 <memory> 头文件中。

C++11 中智能指针包括以下三种：

共享指针（shared_ptr）：资源可以被多个指针共享，使用计数机制表明资源被几个指针共享。通过 use_count() 查看资源的所有者的个数，可以通过 unique_ptr、weak_ptr 来构造，调用 release() 释放资源的所有权，计数减一，当计数减为 0 时，会自动释放内存空间，从而避免了内存泄漏。
独占指针（unique_ptr）：独享所有权的智能指针，资源只能被一个指针占有，该指针不能拷贝构造和赋值。但可以进行移动构造和移动赋值构造（调用 move() 函数），即一个 unique_ptr 对象赋值给另一个 unique_ptr 对象，可以通过该方法进行赋值。
弱指针（weak_ptr）：指向 share_ptr 指向的对象，能够解决由shared_ptr带来的循环引用问题。

#### 实现原理

计数原理

```c++
#include <iostream>
#include <memory>

template <typename T>
class SmartPtr
{
private : 
	T *_ptr;
	size_t *_count;

public:
	SmartPtr(T *ptr = nullptr) : _ptr(ptr)
	{
		if (_ptr)
		{
			_count = new size_t(1);
		}
		else
		{
			_count = new size_t(0);
		}
	}

	~SmartPtr()
	{
		(*this->_count)--;
		if (*this->_count == 0)
		{
			delete this->_ptr;
			delete this->_count;
		}
	}

	SmartPtr(const SmartPtr &ptr) // 拷贝构造：计数 +1
	{
		if (this != &ptr)
		{
			this->_ptr = ptr._ptr;
			this->_count = ptr._count;
			(*this->_count)++;
		}
	}

	SmartPtr &operator=(const SmartPtr &ptr) // 赋值运算符重载 
	{
		if (this->_ptr == ptr._ptr)
		{
			return *this;
		}
		if (this->_ptr) // 将当前的 ptr 指向的原来的空间的计数 -1
		{
			(*this->_count)--;
			if (this->_count == 0)
			{
				delete this->_ptr;
				delete this->_count;
			}
		}
		this->_ptr = ptr._ptr;
		this->_count = ptr._count;
		(*this->_count)++; // 此时 ptr 指向了新赋值的空间，该空间的计数 +1
		return *this;
	}

	T &operator*()
	{
		assert(this->_ptr == nullptr);
		return *(this->_ptr);
	}

	T *operator->()
	{
		assert(this->_ptr == nullptr);
		return this->_ptr;
	}

	size_t use_count()
	{
		return *this->count;
	}
};
```

### unique_ptr赋值

借助 `std::move()` 可以实现将一个 `unique_ptr` 对象赋值给另一个 `unique_ptr` 对象，其目的是实现所有权的转移。

```c++
// A 作为一个类 
std::unique_ptr<A> ptr1(new A());
std::unique_ptr<A> ptr2 = std::move(ptr1);
```

### 使用智能指针会出现什么问题？怎么解决？

智能指针可能出现的问题：循环引用
在如下例子中定义了两个类 Parent、Child，在两个类中分别定义另一个类的对象的共享指针，由于在程序结束后，两个指针相互指向对方的内存空间，导致内存无法释放。

```c++ 
#include <iostream>
#include <memory>

using namespace std;

class Child;
class Parent;

class Parent {
private:
    shared_ptr<Child> ChildPtr;
public:
    void setChild(shared_ptr<Child> child) {
        this->ChildPtr = child;
    }

    void doSomething() {
        if (this->ChildPtr.use_count()) {

        }
    }

    ~Parent() {
    }
};

class Child {
private:
    shared_ptr<Parent> ParentPtr;
public:
    void setPartent(shared_ptr<Parent> parent) {
        this->ParentPtr = parent;
    }
    void doSomething() {
        if (this->ParentPtr.use_count()) {

        }
    }
    ~Child() {
    }
};

int main() {
    weak_ptr<Parent> wpp;
    weak_ptr<Child> wpc;
    {
        shared_ptr<Parent> p(new Parent);
        shared_ptr<Child> c(new Child);
        p->setChild(c);
        c->setPartent(p);
        wpp = p;
        wpc = c;
        cout << p.use_count() << endl; // 2
        cout << c.use_count() << endl; // 2
    }
    cout << wpp.use_count() << endl;  // 1
    cout << wpc.use_count() << endl;  // 1
    return 0;
}
```

循环引用的解决方法： weak_ptr

循环引用：该被调用的析构函数没有被调用，从而出现了内存泄漏。

- weak_ptr 对被 shared_ptr 管理的对象存在 非拥有性（弱）引用，在访问所引用的对象前必须先转化为 shared_ptr；
- weak_ptr 用来打断 shared_ptr 所管理对象的循环引用问题，若这种环被孤立（没有指向环中的外部共享指针），shared_ptr 引用计数无法抵达 0，内存被泄露；令环中的指针之一为弱指针可以避免该情况；
- weak_ptr 用来表达临时所有权的概念，当某个对象只有存在时才需要被访问，而且随时可能被他人删除，可以用 weak_ptr 跟踪该对象；需要获得所有权时将其转化为 shared_ptr，此时如果原来的 shared_ptr 被销毁，则该对象的生命期被延长至这个临时的 shared_ptr 同样被销毁。

```c++ 
#include <iostream>
#include <memory>

using namespace std;

class Child;
class Parent;

class Parent {
private:
    //shared_ptr<Child> ChildPtr;
    weak_ptr<Child> ChildPtr;
public:
    void setChild(shared_ptr<Child> child) {
        this->ChildPtr = child;
    }

    void doSomething() {
        //new shared_ptr
        if (this->ChildPtr.lock()) {

        }
    }

    ~Parent() {
    }
};

class Child {
private:
    shared_ptr<Parent> ParentPtr;
public:
    void setPartent(shared_ptr<Parent> parent) {
        this->ParentPtr = parent;
    }
    void doSomething() {
        if (this->ParentPtr.use_count()) {

        }
    }
    ~Child() {
    }
};

int main() {
    weak_ptr<Parent> wpp;
    weak_ptr<Child> wpc;
    {
        shared_ptr<Parent> p(new Parent);
        shared_ptr<Child> c(new Child);
        p->setChild(c);
        c->setPartent(p);
        wpp = p;
        wpc = c;
        cout << p.use_count() << endl; // 2
        cout << c.use_count() << endl; // 1
    }
    cout << wpp.use_count() << endl;  // 0
    cout << wpc.use_count() << endl;  // 0
    return 0;
}
```



### string 和vector的区别

1、string内置find函数，而vector没有find函数。string内置的find函数，查找时使用的是int类型的pos,不是iterator。

2、string 中的insert使用的是int类型，而vector使用的是iterator类型

3、string 和 vector中的erase都是使用的iterator，但是string中可以使用erase(pos, n )删除pos位置后的n个字符



### string

#### find

* `int find(const char* s, int pos = 0) const; `                     //查找s第一次出现位置,从pos开始查找
* `int find(const char* s, int pos, int n) const; `               //从pos位置查找s的前n个字符第一次位置
* `int find(const char c, int pos = 0) const; `                       //查找字符c第一次出现位置

除此之外还有

find_first_of：从前往后查找何处出现另一个字符串中包含的字符

find_last_of：从后往前查找何处出现另一个字符串中包含的字符

find_first_not_of：从前往后查找何处出现另一个字符串中没有包含的字符。

find_last_not_of：从后往前查找何处出现另一个字符串中没有包含的字符

这些函数的用法和find函数的用法类似

当没有找到时返回npos

#### insert

* `string& insert(int pos, const char* s);  `                //插入字符串
* `string& insert(int pos, const string& str); `        //插入字符串
* `string& insert(int pos, int n, char c);`                //在指定位置插入n个字符c

#### erase

* `string& erase(int pos, int n = npos);`                    //删除从Pos开始的n个字符 
* iterator erase (iterator p);      //删除指定位置的字符
* iterator erase (iterator first, iterator last);   //删除范围内的所有元素

删除之后，字符串即变为删除后的样子

#### substr

substr有2种用法：
假设：string s = "0123456789";

string sub1 = s.substr(5); //只有一个数字5表示从下标为5开始一直到结尾：sub1 = "56789"

string sub2 = s.substr(5, 3); //从下标为5开始截取长度为3位：sub2 = "567"

### vector

#### 插入删除

​    push_back(ele);                                      //先拷贝，在插入；尾部插入元素ele

​	emplace_back(ele);										//直接插入，无需拷贝

​	pop_back();                                              //删除最后一个元素

#### insert

​	iterator insert( iterator pos, const TYPE &val ); //在指定位置pos前插入值为val的元素,返回指向这个元素的迭代器

​	void insert( iterator pos, size_type num, const TYPE &val );	//在指定位置pos前插入num个值为val的元素

​	void insert( iterator pos, input_iterator start, input_iterator end ); //在指定位置pos前插入区间[start, end)的所有元素 .

#### erase

* `erase(const_iterator pos);`                     //删除迭代器指向的元素
* `erase(const_iterator start, const_iterator end);`//删除迭代器从start到end之间的元素

### map

#### 用法

```c++
    定义：
        map<T_key, T_value> mymap;

    插入元素：
        mymap.insert(pair<T_key, T_value>(key, value));    // 同key不插入
        mymap.insert(map<T_key, T_value>::value_type(key, value));    // 同key不插入
        mymap[key] = value;    // 同key覆盖

    删除元素：
        mymap.erase(key);    // 按值删
        mymap.erase(iterator);    // 按迭代器删

    修改元素：
        mymap[key] = new_value;

    遍历容器：
          for(auto it = mymap.begin(); it != mymap.end(); ++it) {
            cout << it->first << " => " << it->second << '\n';
          }
```

#### 实现

红黑树

- 树状结构，RBTree实现。
  - 插入删除不需要数据复制。

  - 操作复杂度仅跟树高有关。

- RBTree本身也是二叉排序树的一种，key值有序，且唯一。
  - 必须保证key可排序。

基于红黑树实现的map结构（实际上是map, set, multimap，multiset底层均是红黑树），不仅增删数据时不需要移动数据，其所有操作都可以在O(logn)时间范围内完成。另外，基于红黑树的map在通过迭代器遍历时，得到的是key按序排列后的结果，这点特性在很多操作中非常方便。

#### 红黑树相关概念

1. 它是二叉排序树（继承二叉排序树特显）：
   - 若左子树不空，则左子树上所有结点的值均小于或等于它的根结点的值。

   - 若右子树不空，则右子树上所有结点的值均大于或等于它的根结点的值。

   - 左、右子树也分别为二叉排序树。

2. 它满足如下几点要求：
   - 树中所有节点非红即黑。

   - 根节点必为黑节点。

   - 红节点的子节点必为黑（黑节点子节点可为黑）。

   - 从根到NULL的任何路径上黑结点数相同。

3. 查找时间一定可以控制在O(logn)。

从红黑树的定义来看，红黑树从根到NULL的每条路径拥有相同的黑节点数（假设为n），所以最短的路径长度为n（全为黑节点情况）。因为红节点不能连续出现，所以路径最长的情况就是插入最多的红色节点，在黑节点数一致的情况下，最可观的情况就是黑红黑红排列......最长路径不会大于2n，这里路径长就是树高。

### 什么是面向对象？面向对象的三大特性

面向对象：对象是指具体的某一个事物，这些事物的抽象就是类，类中包含数据（成员变量）和动作（成员方法）。

面向对象编程将数据成员和成员函数封装到一个类中，并声明数据成员和成员函数的访问级别（public、private、protected），以便控制类对象对数据成员和函数的访问，对数据成员起到一定的保护作用。而且在类的对象调用成员函数时，只需知道成员函数的名、参数列表以及返回值类型即可，无需了解其函数的实现原理。当类内部的数据成员或者成员函数发生改变时，不影响类外部的代码。

面向对象的三大特性：

封装：将具体的实现过程和数据封装成一个函数，只能通过接口进行访问，降低耦合性。
继承：子类继承父类的特征和行为，子类有父类的非 private 方法或成员变量，子类可以对父类的方法进行重写，增强了类之间的耦合性，但是当父类中的成员变量、成员函数或者类本身被 final 关键字修饰时，修饰的类不能继承，修饰的成员不能重写或修改。
多态：多态就是不同继承类的对象，对同一消息做出不同的响应，基类的指针指向或绑定到派生类的对象，使得基类指针呈现不同的表现方式。

### 什么是多态？多态如何实现？

多态：多态就是不同继承类的对象，对同一消息做出不同的响应，基类的指针指向或绑定到派生类的对象，使得基类指针呈现不同的表现方式。在基类的函数前加上 virtual 关键字，在派生类中重写该函数，运行时将会根据对象的实际类型来调用相应的函数。如果对象类型是派生类，就调用派生类的函数；如果对象类型是基类，就调用基类的函数。
实现方法：多态是通过虚函数实现的，虚函数的地址保存在虚函数表中，虚函数表的地址保存在含有虚函数的类的实例对象的内存空间中。

实现过程：

在类中用 virtual 关键字声明的函数叫做虚函数；
存在虚函数的类都有一个虚函数表，当创建一个该类的对象时，该对象有一个指向虚函数表的虚表指针（虚函数表和类对应的，虚表指针是和对象对应）；
当基类指针指向派生类对象，基类指针调用虚函数时，基类指针指向派生类的虚表指针，由于该虚表指针指向派生类虚函数表，通过遍历虚表，寻找相应的虚函数。

```c++
#include <iostream>
using namespace std;

class Base
{
public:
	virtual void fun() { cout << "Base::fun()" << endl; }

	virtual void fun1() { cout << "Base::fun1()" << endl; }

	virtual void fun2() { cout << "Base::fun2()" << endl; }
};
class Derive : public Base
{
public:
	void fun() { cout << "Derive::fun()" << endl; }

	virtual void D_fun1() { cout << "Derive::D_fun1()" << endl; }

	virtual void D_fun2() { cout << "Derive::D_fun2()" << endl; }
};
int main()
{
	Base *p = new Derive();
	p->fun(); // Derive::fun() 调用派生类中的虚函数
	return 0;
}
```

### 什么是模板？如何实现？

模板：创建类或者函数的蓝图或者公式，分为函数模板和类模板。
实现方式：模板定义以关键字 template 开始，后跟一个模板参数列表。

模板参数列表不能为空；
模板类型参数前必须使用关键字 class 或者 typename，在模板参数列表中这两个关键字含义相同，可互换使用。

模板格式：

```c++
template <typename T, typename U, ...>
```

**函数模板**

函数模板：通过定义一个函数模板，可以避免为每一种类型定义一个新函数。

对于函数模板而言，模板类型参数可以用来指定返回类型或函数的参数类型，以及在函数体内用于变量声明或类型转换。
函数模板实例化：当调用一个模板时，编译器用函数实参来推断模板实参，从而使用实参的类型来确定绑定到模板参数的类型。

编译器会对函数模板进行两次编译（1）在声明的位置对模板代码进行编译；（2）在调用的位置对参数替换后的代码进行编译。

```c++
#include<iostream>

using namespace std;

template <typename T>
T add_fun(const T & tmp1, const T & tmp2){
    return tmp1 + tmp2;
}

int main(){
    int var1, var2;
    cin >> var1 >> var2;
    cout << add_fun(var1, var2);

    double var3, var4;
    cin >> var3 >> var4;
    cout << add_fun(var3, var4);
    return 0;
}
```

**类模板**

类似函数模板，类模板以关键字 template 开始，后跟模板参数列表。但是，编译器不能为类模板推断模板参数类型，需要在使用该类模板时，在模板名后面的尖括号中指明类型。

```c++
#include <iostream>

using namespace std;

template <typename T>
class Complex
{
public:
    //构造函数
    Complex(T a, T b)
    {
        this->a = a;
        this->b = b;
    }

    //运算符重载
    Complex<T> operator+(Complex &c)
    {
        Complex<T> tmp(this->a + c.a, this->b + c.b);
        cout << tmp.a << " " << tmp.b << endl;
        return tmp;
    }

private:
    T a;
    T b;
};

int main()
{
    Complex<int> a(10, 20);
    Complex<int> b(20, 30);
    Complex<int> c = a + b;

    return 0;
}
```

### 函数模板和类模板的区别？

实例化方式不同：函数模板实例化由编译程序在处理函数调用时自动完成，类模板实例化需要在程序中显示指定。
实例化的结果不同：函数模板实例化后是一个函数，类模板实例化后是一个类。
默认参数：类模板在模板参数列表中可以有默认参数。
特化：函数模板只能全特化；而类模板可以全特化，也可以偏特化。
调用方式不同：函数模板可以隐式调用，也可以显示调用；类模板只能显示调用。

函数模板调用方式举例：

```c++
#include<iostream>

using namespace std;

template <typename T>
T add_fun(const T & tmp1, const T & tmp2){
    return tmp1 + tmp2;
}

int main(){
    int var1, var2;
    cin >> var1 >> var2;
    cout << add_fun<int>(var1, var2); // 显示调用

    double var3, var4;
    cin >> var3 >> var4;
    cout << add_fun(var3, var4); // 隐式调用
    return 0;
}
```

全特化：特化其实就是特殊化的意思，在模板类里，所有的类型都是模板（template<class T>），而一旦我们将所有的模板类型T都明确化，并且写了一个类名与主模板类名相同的类，那么这个类就叫做全特化类。C++模板全特化之后已经失去了Template的属性了。

```c++
//模板函数
template<typename T, class N> void func(T num1, N num2)
{
    //cout << "num1:" << num1 << ", num2:" << num2 <<endl;
}
 
//模板类
template<typename T, class N> class Test_Class
{
public:
    static bool comp(T num1, N num2)
    {
        return (num1<num2)?true:false;
    }
};
 
//全特化，模板函数
template<> void func(int num1, double num2)
{
    cout << "num1:" << num1 << ", num2:" << num2 <<endl;
}
 
//全特化，模板类
template<> class Test_Class<int, double>
{
public:
    static bool comp(int num1, double num2)
    {
        return (num1<num2)?true:false;
    }
};

//调用
func<int, double>(1, 2.0);
Test_Class<int, double>::comp(1, 2.0);
```

偏特化：模板中的模板参数只确定了一部分，剩余部分需要在编译器编译时确定。

```c++
//模板函数
template<typename T, class N> void func(T num1, N num2)
{
    //cout << "num1:" << num1 << ", num2:" << num2 <<endl;
}
 
//模板类
template<typename T, class N> class Test_Class
{
public:
    static bool comp(T num1, N num2)
    {
        return (num1<num2)?true:false;
    }
};
 
//偏特化，模板函数
template<class N> void func(int num1, N num2)
{
    cout << "num1:" << num1 << ", num2:" << num2 <<endl;
}
 
//偏特化，模板类
template<class N> class Test_Class<int, N>
{
public:
    static bool comp(int num1, double num2)
    {
        return (num1<num2)?true:false;
    }
};

//调用
func<int, double>(1, 2.0);
Test_Class<int, double>::comp(1, 2.0);
```

对主版本模板类、全特化类、偏特化类的调用优先级从高到低进行排序是：***\*全特化类>偏特化类>主版本模板类\****。这样的优先级顺序对性能也是最好的。

### 程序编译过程

#### 预处理

处理以 # 开头的指令

- **展开所有的宏定义，完成字符常量替换。**
- **处理条件编译语句，通过是否具有某个宏来决定过滤掉哪些代码。**
- **处理#include指令，将被包含的文件插入到该指令所在位置。**
- **过滤掉所有注释语句。**
- 添加行号和文件名标识。
- 保留所有#pragma编译器指令。

#### 编译

将源码 .cpp 文件翻译成 .s 汇编代码；

- 代码规范性
- 语法分析。
- 语义分析。
- 中间语言生成。
- 目标代码生成与优化。

#### 汇编

汇编阶段把*.s文件翻译成二进制机器指令文件*.o，汇编阶段是把编译阶段生成的”.s”文件转成目标文件。

#### 链接	

汇编程序生成的目标文件，即 .o 文件，并不会立即执行，因为可能会出现：.cpp 文件中的函数引用了另一个 .cpp 文件中定义的符号或者调用了某个库文件中的函数。那链接的目的就是将这些文件对应的目标文件连接成一个整体，从而生成可执行的程序 .exe 文件。

- 静态链接

  **静态链接最简单的情况就是在编译时和静态库链接在一起成为完整的可执行程序。**这里所说的静态库就是对多个目标文件（.o）文件的打包，通常静态链接的包名为lib****.a，静态链接所有被用到的目标文件都会复制到最终生成的可执行目标文件中。这种方式的好处是在运行时，可执行目标文件已经完全装载完毕，只要按指令序执行即可，速度比较快，但缺点也有很多，在讲动态链接时会比较一下。

  既然静态链接是对目标文件的打包，这里介绍些打包命令。

      gcc -c test1.c    // 生成test1.o
      gcc -c test2.c    // 生成test2.c
      ar cr libtest.a test1.o test2.o

  首先编译得到test1.o和test2.o两个目标文件，之后通过ar命令将这两个文件打包为.a文件，文件名格式为lib + 静态库名 + .a后缀。在生成可执行文件需要使用到它的时候只需要在编译时加上即可。需要注意的是，使用静态库时加在最后的名字不是libtest.a，而是l + 静态库名。

      gcc -o main main.c -ltest

- 动态链接

  静态链接发生于编译阶段，加载至内存前已经完整，但缺点是如果多个程序都需要使用某个静态库，则该静态库会在每个程序中都拷贝一份，非常浪费内存资源，所以出现了动态链接的方式来解决这个问题。

  

  动态链接：代码被放到动态链接库或共享对象的某个目标文件中，链接程序只是在最终的可执行程序中记录了共享对象的名字等一些信息。在程序执行时，动态链接库的全部内容会被映射到运行时相应进行的虚拟地址的空间。

  

  动态链接在形式上倒是和静态链接非常相似，首先也是需要打包，打包成动态库，不过文件名格式为lib + 动态库名 + .so后缀。不过动态库的打包不需要使用ar命令，gcc就可以完成，但要注意在编译时要加上-fPIC选项，打包时加上-shared选项。

      gcc -fPIC -c test1.c 
      gcc -fPIC -c test2.c
      gcc -shared test1.o test2.o -o libtest.so

  使用动态链接的用法也和静态链接相同。

      gcc -o main main.c -ltest

如果仅仅像上面的步骤是没有办法正常使用库的，我们可以通过加-Lpath指定搜索库文件的目录（-L.表示当前目录），默认情况下会到环境变量LD_LIBRARY_PATH指定的目录下搜索库文件，默认情况是/usr/lib，我们可以将库文件拷贝到那个目录下再链接。

#### 动态库和静态库

区别

- 命名方式不同：静态库libxxx.a：库名前加”lib”，后缀用”.a”，“xxx”为静态库名；动态库libxxx.so：库名前加”lib”，后缀变为“.so”。
- 链接时间不同：静态库的代码是在编译过程中被载入程序中。动态库的代码是当程序运行到相关函数才调用动态库的相应函数。
- 链接方式不同：静态库的链接是将整个函数库的所有数据在编译时都整合进了目标代码。动态库的链接是程序执行到哪个函数链接哪个函数的库。**（用哪个链接哪个）**

优缺点：

静态库：

- 优点是，在编译后的执行程序不再需要外部的函数库支持，运行速度相对快些；
- 缺点是，如果所使用的静态库发生更新改变，你的程序必须重新编译。

动态库 ：

- 优点是，动态库的改变并不影响你的程序，所以动态函数库升级比较方便；
- 缺点是，因为函数库并没有整合进程序，所以程序的运行环境必须提供相应的库。

### C++内存管理

C++ 内存分区：栈、堆、全局/静态存储区、常量存储区、代码区。

栈：存放函数的局部变量、函数参数、返回地址等，由编译器自动分配和释放。                                                                    堆：动态申请的内存空间，就是由 malloc 分配的内存块，由程序员控制它的分配和释放，如果程序执行结束还没有释放，操作系统会自动回收。                                                                                                                                                                       全局区/静态存储区（.bss 段和 .data 段）：存放全局变量和静态变量，程序运行结束操作系统自动释放，在 C 语言中，未初始化的放在 .bss 段中，初始化的放在 .data 段中，C++ 中不再区分了。
常量存储区（.rodata 段）：存放的是常量，不允许修改，程序运行结束自动释放。
代码区（.text 段）：存放代码，不允许修改，但可以执行。编译后的二进制文件存放在这里。

### 可变参数模板

可变参数模板：接受可变数目参数的模板函数或模板类。将可变数目的参数被称为参数包，包括模板参数包和函数参数包。

模板参数包：表示零个或多个模板参数；
函数参数包：表示零个或多个函数参数。
用省略号来指出一个模板参数或函数参数表示一个包，在模板参数列表中，class... 或 typename... 指出接下来的参数表示零个或多个类型的列表；一个类型名后面跟一个省略号表示零个或多个给定类型的非类型参数的列表。当需要知道包中有多少元素时，可以使用 sizeof... 运算符。

```c++
template <typename T, typename... Args> // Args 是模板参数包
void foo(const T &t, const Args&... rest); // 可变参数模板，rest 是函数参数包
```

实例：

```c++
#include <iostream>

using namespace std;

template <typename T>
void print_fun(const T &t)
{
    cout << t << endl; // 最后一个元素
}

template <typename T, typename... Args>
void print_fun(const T &t, const Args &...args)
{
    cout << t << " ";
    print_fun(args...);
}

int main()
{
    print_fun("Hello", "wolrd", "!");
    return 0;
}
/*运行结果：
Hello wolrd !

*/
```

说明：可变参数函数通常是递归的，第一个版本的 print_fun 负责终止递归并打印初始调用中的最后一个实参。第二个版本的 print_fun 是可变参数版本，打印绑定到 t 的实参，并用来调用自身来打印函数参数包中的剩余值。

### include " " 和 <> 的区别

include<文件名> 和 #include"文件名" 的区别:

查找文件的位置：include<文件名> 在标准库头文件所在的目录中查找，如果没有，再到当前源文件所在目录下查找；#include"文件名" 在当前源文件所在目录中进行查找，如果没有；再到系统目录中查找。

使用习惯：对于标准库中的头文件常用 include<文件名>，对于自己定义的头文件，常用 #include"文件名"

### 迭代器的作用

迭代器：一种抽象的设计概念，在设计模式中有迭代器模式，即提供一种方法，使之能够依序寻访某个容器所含的各个元素，而无需暴露该容器的内部表述方式。

作用：在无需知道容器底层原理的情况下，遍历容器中的元素。

## 语言对比

### C++11新特性

尽量列举一些常用的并且熟悉的特性，尽可能的掌握相关原理

#### auto 类型推导

`auto` 关键字：自动类型推导，编译器会在 **编译期间** 通过**初始值**推导出变量的类型，通过 `auto` 定义的变量必须有初始值。

注意：编译器推导出来的类型和初始值的类型并不完全一样，编译器会适当地改变结果类型使其更符合初始化规则。

#### decltype类型推导

decltype 关键字：decltype 是“declare type”的缩写，译为“声明类型”。和 auto 的功能一样，都用来在编译时期进行自动类型推导。如果希望从表达式中推断出要定义的变量的类型，但是不想用该表达式的值初始化变量，这时就不能再用 auto。decltype 作用是选择并返回操作数的数据类型。

区别

```c++
auto var = val1 + val2; 
decltype(val1 + val2) var1 = 0; 
```

- auto 根据 = 右边的初始值 val1 + val2 推导出变量的类型，并将该初始值赋值给变量 var；decltype 根据 val1 + val2 表达式推导出变量的类型，变量的初始值和与表达式的值无关。
- auto 要求变量必须初始化，因为它是根据初始化的值推导出变量的类型，而 decltype 不要求，定义变量的时候可初始化也可以不初始化。

#### lambda表达式

定义

```c++
[capture list] (parameter list) -> return type
{
   function body;
};
```

其中：

capture list：捕获列表，指 lambda 所在函数中定义的局部变量的列表，通常为空。
return type、parameter list、function body：分别表示返回值类型、参数列表、函数体，和普通函数一样。

```c++
#include <iostream>
#include <algorithm>
using namespace std;

int main()
{
    int arr[4] = {4, 2, 3, 1};
    //对 a 数组中的元素进行升序排序
    sort(arr, arr+4, [=](int x, int y) -> bool{ return x < y; } );
    for(int n : arr){
        cout << n << " ";
    }
    return 0;
}
```

#### 范围for语句

```c++
for (declaration : expression){
    statement
}
```

参数的含义：

expression：必须是一个序列，例如用花括号括起来的初始值列表、数组、vector ，string 等，这些类型的共同特点是拥有能返回迭代器的 beign、end 成员。
declaration：此处定义一个变量，序列中的每一个元素都能转化成该变量的类型，常用 auto 类型说明符。

```c++
#include <iostream>
#include <vector>
using namespace std;
int main() {
    char arr[] = "hello world!";
    for (char c : arr) {
        cout << c;
    }  
    return 0;
}
/*
程序执行结果为：
hello world!
*/
```

#### 智能指针

#### 右值引用

右值引用是 C++11 中引入的新特性 , 它实现了转移语义和精确传递。它的主要目的有两个方面：
1.	消除两个对象交互时不必要的对象拷贝，节省运算存储资源，提高效率。
2.	能够更简洁明确地定义泛型函数。

左值和右值的概念：
左值：能对表达式取地址、或具名对象/变量。一般指表达式结束后依然存在的持久对象。
右值：不能对表达式取地址，或匿名对象。一般指表达式结束就不再存在的临时对象。

右值引用和左值引用的区别：
1.	左值可以寻址，而右值不可以。
2.	左值可以被赋值，右值不可以被赋值，可以用来给左值赋值。
3. 左值可变,右值不可变（仅对基础类型适用，用户自定义类型右值引用可以通过成员函数改变）。

```
    int a =10;
    int&  lr = a;       // a 是个变量
    int&& rv = 10;      // 10 是常量
```

结论：<font color=red>无论左值引用还是右值引用，都是左值</font>，即上面的lr和rv是左值、是个变量，只是左值引用lr指向的是变量a，而右值引用rv指向常量10。

#### 移动语义 

在面向对象中，有的类是可以拷贝的，例如车、房等他们的属性是可以复制的，可以调用拷贝构造函数，有点类的对象则是独一无二的，或者类的资源是独一无二的，比如 IO 、 std::unique_ptr等，他们不可以复制，但是可以把资源交出所有权给新的对象，称为可以移动的。
C++11最重要的一个改进之一就是引入了move语义，这样在一些对象的构造时可以获取到已有的资源（如内存）而不需要通过拷贝，申请新的内存，这样移动而非拷贝将会大幅度提升性能。例如有些右值即将消亡析构，这个时候我们用移动构造函数可以接管他们的资源。

NOTE：拷贝构造函数中是对传进来的对象进行了实实在在的拷贝工作；而移动构造函数中只是对传进来的对象进行了所有权的转让，即掏空传进来的对象，然后把所有权转给当前对象

`std::move`的实现即使**将一个对象强制转型为右值引用类型的对象而已**，并不做任何移动工作。

```c++
template<typename T>
typename remove_reference<T>::type && move(T&& t)
{
    return static_cast<typename remove_reference<T>::type &&>(t);
}
```

说明：引用折叠原理

右值传递给上述函数的形参 T&& 依然是右值，即 T&& && 相当于 T&&。
左值传递给上述函数的形参 T&& 依然是左值，即 T&& & 相当于 T&。

通过引用折叠原理可以知道，`move()` 函数的形参既可以是左值也可以是右值。

move函数说明：

1、利用引用折叠原理将右值经过 T&& 传递类型保持不变还是右值，而左值经过 T&& 变为普通的左值引用，以保证模板可以传递任意实参，且保持类型不变；

2、然后通过 remove_refrence 移除引用，得到具体的类型 T；

3、最后通过 static_cast<> 进行强制类型转换，返回 T&& 右值引用。

#### 完美转发

完美转发是指在函数模板中，完全按照模板的参数的类型，将参数传递给函数模板中调用的另一个函数，即传入转发函数的是左值对象，目标函数就能获得左值对象，转发函数是右值对象，目标函数就能获得右值对象，而不产生额外的开销。

因此转发函数和目标函数参数一般采用引用类型，从而避免拷贝的开销。其次，由于目标函数可能需要既能接受左值的引用，又能接受右值引用，所以考虑转发也需要兼容这两种类型。

C++11采用引用折叠的规则，结合新的模板推导规则实现完美转发。其引用折叠规则如下：

X& + & => X&
X&& + & => X&
X& + && => X&
X&& + && => X&&

折叠的实现是类型转换。static_cast

因此，转发函数和目标函数的参数都设置成右值引用类型。

std::forward只有在它的参数绑定到一个右值上的时候，它才转换它的参数到一个右值。

```c++
template<class T>
T&& forward(remove_reference_t<T>& t) {
    return static_cast<T&&>(t);
}
```

#### delete函数和default函数

- `delete` 函数：`= delete` 表示该函数不能被调用。
- `default` 函数：`= default` 表示编译器生成默认的函数，例如：生成默认的构造函数。

```c++
#include <iostream>
using namespace std;

class A
{
public:
	A() = default; // 表示使用默认的构造函数
	~A() = default;	// 表示使用默认的析构函数
	A(const A &) = delete; // 表示类的对象禁止拷贝构造
	A &operator=(const A &) = delete; // 表示类的对象禁止拷贝赋值
};
int main()
{
	A ex1;
	A ex2 = ex1; // error: use of deleted function 'A::A(const A&)'
	A ex3;
	ex3 = ex1; // error: use of deleted function 'A& A::operator=(const A&)'
	return 0;
}
```

#### nullptr

NULL：NULL在C++中被明确定义为整数0。

nullptr：nullptr关键字用于标识空指针，是std::nullptr_t类型的（constexpr）变量。它可以转换成任何指针类型和bool布尔类型（主要是为了兼容普通指针可以作为条件判断语句的写法），但是不能被转换为整数。

**为什么用nullptr代替NULL？**

由于C++函数重载机制。由于我们经常使用NULL表示空指针，所以从程序员的角度来看，Func（NULL）应该调用的是Func（char *）但实际上NULL的值是0，所以调用了Func（int）。nullptr关键字真是为了解决这个问题而引入的。

```c++
void Func(char *);
void Func(int);

int main()
{
    Func(NULL);
}
```

**另外我们还有注意到NULL只是一个宏定义，而nullptr是一个C++关键字。**

nullptr关键字用于标识空指针，是std::nullptr_t类型的（constexpr）变量。它可以转换成任何指针类型和bool布尔类型（主要是为了兼容普通指针可以作为条件判断语句的写法），但是不能被转换为整数。

```c++ 
char *p1 = nullptr;     // 正确
int  *p2 = nullptr;     // 正确
bool b = nullptr;       // 正确. if(b)判断为false
int a = nullptr;        // error
```

#### constexpr

关键字 constexpr 是C++11中引入的关键字，声明为**constexpr类型的变量**，编译器会验证该变量的值是否是一个常量表达式。

声明为constexpr的变量一定是一个常量，而且必须用常量表达式初始化：

```c++
constexpr int mf = 0; // 0 是常量表达式
constexpr int limit = mf + 1; // mf + 1 是常量表达式
constexpr int sz = size(); // 只有当 size() 是一个constexpr函数时才是一条正确的声明语句
```

const 和 constexpr 变量之间的主要区别在于：const 变量的初始化可以延迟到运行时，而 constexpr 变量必须在编译时进行初始化。所有 constexpr 变量均为常量，因此必须使用常量表达式初始化。

#### overrid 和 final关键字

在C++ 11之前，没有一个强制的机制来标识虚函数在派生类中正确被重写。

- override：表示当前函数重写了基类的虚函数。
- final：阻止类的进一步派生 和 虚函数的进一步重写。

![image-20210604164504002](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210604164504002.png)

要确认派生类中的成员函数覆盖基类中的虚成员函数，可以在派生类的函数原型（如果函数以内联方式写入，则在函数头）后面加上 override 关键字。override 关键字告诉编译器，该函数应覆盖基类中的函数。如果该函数实际上没有覆盖任何函数，则会导致编译器错误。

![image-20210604164542485](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210604164542485.png)

**final**

（1）基类的虚函数上加上final关键字，表明基类后续的派生类不能重写该虚函数

（2）在派生类的虚函数加上关键final说明该派生类后续的派生类不能重写该函数！

（3）在派生类类的虚函数加上关键字override和final，说明派生类后续的派生类不能重写该函数，但是当前派生类可以重写当前基类的虚函数！

（4）C++11中允许将类标记为final，方法时直接在类名称后面使用关键字final，如此，意味着继承该类会导致编译错误。

### C和C++的区别

区别和联系：

语言自身：C 语言是面向过程的编程，它最重要的特点是函数，通过 main 函数来调用各个子函数。程序运行的顺序都是程序员事先决定好的。C++ 是面向对象的编程，类是它的主要特点，在程序执行过程中，先由主 main 函数进入，定义一些类，根据需要执行类的成员函数，过程的概念被淡化了（实际上过程还是有的，就是主函数的那些语句。），以类驱动程序运行，类就是对象，所以我们称之为面向对象程序设计。面向对象在分析和解决问题的时候，将涉及到的数据和数据的操作封装在类中，通过类可以创建对象，以事件或消息来驱动对象执行处理。

应用领域：C 语言主要用于嵌入式领域，驱动开发等与硬件直接打交道的领域，C++ 可以用于应用层开发，用户界面开发等与操作系统打交道的领域。

C++ 既继承了 C 强大的底层操作特性，又被赋予了面向对象机制。它特性繁多，面向对象语言的多继承，对值传递与引用传递的区分以及 const 关键字，等等。

C++ 对 C 的“增强”，表现在以下几个方面：类型检查更为严格。增加了面向对象的机制、泛型编程的机制（Template）、异常处理、运算符重载、标准模板库（STL）、命名空间（避免全局命名冲突）。

### Java和C++的区别

二者在语言特性上有很大的区别：

指针：C++ 可以直接操作指针，容易产生内存泄漏以及非法指针引用的问题；Java 并不是没有指针，虚拟机（JVM）内部还是使用了指针，只是编程人员不能直接使用指针，不能通过指针来直接访问内存，并且 Java 增加了内存管理机制。
多重继承：C++ 支持多重继承，允许多个父类派生一个类，虽然功能很强大，但是如果使用的不当会造成很多问题，例如：菱形继承；Java 不支持多重继承，但允许一个类可以继承多个接口，可以实现 C++ 多重继承的功能，但又避免了多重继承带来的许多不便。
数据类型和类：Java 是完全面向对象的语言，所有函数和变量部必须是类的一部分。除了基本数据类型之外，其余的都作为类对象，包括数组。对象将数据和方法结合起来，把它们封装在类中，这样每个对象都可实现自己的特点和行为。而 C++ 允许将函数和变量定义为全局的。

**垃圾回收：**

Java 语言一个显著的特点就是垃圾回收机制，编程人员无需考虑内存管理的问题，可以有效的防止内存泄漏，有效的使用空闲的内存。
Java 所有的对象都是用 new 操作符建立在内存堆栈上，类似于 C++ 中的 new 操作符，但是当要释放该申请的内存空间时，Java 自动进行内存回收操作，C++ 需要程序员自己释放内存空间，并且 Java 中的内存回收是以线程的方式在后台运行的，利用空闲时间。

**应用场景：**

Java 运行在虚拟机上，和开发平台无关，C++ 直接编译成可执行文件，是否跨平台在于用到的编译器的特性是否有多平台的支持。
C++ 可以直接编译成可执行文件，运行效率比 Java 高。
Java 主要用来开发 Web 应用。
C++ 主要用在嵌入式开发、网络、并发编程的方面。

### Python 和 C++ 的区别

语言自身：Python 为脚本语言，解释执行，不需要经过编译；C++ 是一种需要编译后才能运行的语言，在特定的机器上编译后运行。
运行效率：C++ 运行效率高，安全稳定。原因：Python 代码和 C++ 最终都会变成 CPU 指令来跑，但一般情况下，比如反转和合并两个字符串，Python 最终转换出来的 CPU 指令会比 C++ 多很多。首先，Python 中涉及的内容比 C++ 多，经过了更多层，Python 中甚至连数字都是 object ；其次，Python 是解释执行的，和物理机 CPU 之间多了解释器这层，而 C++ 是编译执行的，直接就是机器码，编译的时候编译器又可以进行一些优化。
开发效率：Python 开发效率高。原因：Python 一两句代码就能实现的功能，C++ 往往需要更多的代码才能实现。
书写格式和语法不同：Python 的语法格式不同于其 C++ 定义声明才能使用，而且极其灵活，完全面向更上层的开发者。

## 类相关

### 什么是虚函数？什么是纯虚函数？

**虚函数**：被 `virtual` 关键字修饰的成员函数，就是虚函数。

纯虚函数：

- 纯虚函数在类中声明时，加上 =0；
- 含有纯虚函数的类称为抽象类（只要含有纯虚函数这个类就是抽象类），类中只有接口，没有具体的实现方法
- 继承纯虚函数的派生类，如果没有完全实现基类纯虚函数，依然是抽象类，不能实例化对象。

说明：

- 抽象类对象不能作为函数的参数，不能创建对象，不能作为函数返回类型；
- 可以声明抽象类指针，可以声明抽象类的引用；
- 子类必须继承父类的纯虚函数，并全部实现后，才能创建子类的对象。

### 虚函数和纯虚函数的区别？

- 虚函数和纯虚函数可以出现在同一个类中，该类称为抽象基类。（含有纯虚函数的类称为抽象基类）
- 使用方式不同：虚函数可以直接使用，纯虚函数必须在派生类中实现后才能使用；
- 定义形式不同：虚函数在定义时在普通函数的基础上加上 virtual 关键字，纯虚函数定义时除了加上virtual 关键字还需要加上 =0;
- 虚函数必须实现，否则编译器会报错；
- 对于实现纯虚函数的派生类，该纯虚函数在派生类中被称为虚函数，虚函数和纯虚函数都可以在派生类中重写；
- 析构函数最好定义为虚函数，特别是对于含有继承关系的类；析构函数可以定义为纯虚函数，此时，其所在的类为抽象基类，不能创建实例化对象。

### 虚函数的实现机制

实现机制：虚函数通过虚函数表来实现。虚函数的地址保存在虚函数表中，在类的对象所在的内存空间中，保存了指向虚函数表的指针（称为“虚表指针”），通过虚表指针可以找到类对应的虚函数表。虚函数表解决了基类和派生类的继承问题和类中成员函数的覆盖问题，当用基类的指针来操作一个派生类的时候，这张虚函数表就指明了实际应该调用的函数。

虚函数表相关知识点：

虚函数表存放的内容：类的虚函数的地址。
虚函数表建立的时间：编译阶段，即程序的编译过程中会将虚函数的地址放在虚函数表中。
虚表指针保存的位置：虚表指针存放在对象的内存空间中最前面的位置，这是为了保证正确取到虚函数的偏移量。
注：虚函数表和类绑定，虚表指针和对象绑定。即类的不同的对象的虚函数表是一样的，但是每个对象都有自己的虚表指针，来指向类的虚函数表。

实例：

无虚函数覆盖的情况：

```c
#include <iostream>
using namespace std;

class Base
{
public:
    virtual void B_fun1() { cout << "Base::B_fun1()" << endl; }
    virtual void B_fun2() { cout << "Base::B_fun2()" << endl; }
    virtual void B_fun3() { cout << "Base::B_fun3()" << endl; }
};

class Derive : public Base
{
public:
    virtual void D_fun1() { cout << "Derive::D_fun1()" << endl; }
    virtual void D_fun2() { cout << "Derive::D_fun2()" << endl; }
    virtual void D_fun3() { cout << "Derive::D_fun3()" << endl; }
};
int main()
{
    Base *p = new Derive();
    p->B_fun1(); // Base::B_fun1()
    return 0;
}
```

![image-20210607152117892](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210607152117892.png)

![image-20210607152129773](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210607152129773.png)

主函数中基类的指针 `p` 指向了派生类的对象，当调用函数 `B_fun1()` 时，通过派生类的虚函数表找到该函数的地址，从而完成调用。

### 单继承和多继承的虚函数表结构

编译器处理虚函数表：

- 编译器将虚函数表的指针放在类的实例对象的内存空间中，该对象调用该类的虚函数时，通过指针找到虚函数表，根据虚函数表中存放的虚函数的地址找到对应的虚函数。
- 如果派生类没有重新定义基类的虚函数 A，则派生类的虚函数表中保存的是基类的虚函数 A 的地址，也就是说基类和派生类的虚函数 A 的地址是一样的。
- 如果派生类重写了基类的某个虚函数 B，则派生的虚函数表中保存的是重写后的虚函数 B 的地址，也就是说虚函数 B 有两个版本，分别存放在基类和派生类的虚函数表中。
- 如果派生类重新定义了新的虚函数 C，派生类的虚函数表保存新的虚函数 C 的地址。

1、单继承无虚函数覆盖的情况：

```c++
#include <iostream>
using namespace std;

class Base
{
public:
    virtual void B_fun1() { cout << "Base::B_fun1()" << endl; }
    virtual void B_fun2() { cout << "Base::B_fun2()" << endl; }
    virtual void B_fun3() { cout << "Base::B_fun3()" << endl; }
};

class Derive : public Base
{
public:
    virtual void D_fun1() { cout << "Derive::D_fun1()" << endl; }
    virtual void D_fun2() { cout << "Derive::D_fun2()" << endl; }
    virtual void D_fun3() { cout << "Derive::D_fun3()" << endl; }
};
int main()
{
    Base *p = new Derive();
    p->B_fun1(); // Base::B_fun1()
    return 0;
}
```

![image-20210607152520473](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210607152520473.png)

![image-20210607152558788](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210607152558788.png)

2、单继承有虚函数覆盖的情况：

```c++
#include <iostream>
using namespace std;

class Base
{
public:
    virtual void fun1() { cout << "Base::fun1()" << endl; }
    virtual void B_fun2() { cout << "Base::B_fun2()" << endl; }
    virtual void B_fun3() { cout << "Base::B_fun3()" << endl; }
};

class Derive : public Base
{
public:
    virtual void fun1() { cout << "Derive::fun1()" << endl; }
    virtual void D_fun2() { cout << "Derive::D_fun2()" << endl; }
    virtual void D_fun3() { cout << "Derive::D_fun3()" << endl; }
};
int main()
{
    Base *p = new Derive();
    p->fun1(); // Derive::fun1()
    return 0;
}
```

![image-20210607152653951](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210607152653951.png)

3、多继承无虚函数覆盖的情况

```c++
#include <iostream>
using namespace std;

class Base1
{
public:
    virtual void B1_fun1() { cout << "Base1::B1_fun1()" << endl; }
    virtual void B1_fun2() { cout << "Base1::B1_fun2()" << endl; }
    virtual void B1_fun3() { cout << "Base1::B1_fun3()" << endl; }
};
class Base2
{
public:
    virtual void B2_fun1() { cout << "Base2::B2_fun1()" << endl; }
    virtual void B2_fun2() { cout << "Base2::B2_fun2()" << endl; }
    virtual void B2_fun3() { cout << "Base2::B2_fun3()" << endl; }
};
class Base3
{
public:
    virtual void B3_fun1() { cout << "Base3::B3_fun1()" << endl; }
    virtual void B3_fun2() { cout << "Base3::B3_fun2()" << endl; }
    virtual void B3_fun3() { cout << "Base3::B3_fun3()" << endl; }
};

class Derive : public Base1, public Base2, public Base3
{
public:
    virtual void D_fun1() { cout << "Derive::D_fun1()" << endl; }
    virtual void D_fun2() { cout << "Derive::D_fun2()" << endl; }
    virtual void D_fun3() { cout << "Derive::D_fun3()" << endl; }
};

int main(){
    Base1 *p = new Derive();
    p->B1_fun1(); // Base1::B1_fun1()
    return 0;
}
```

![image-20210607152808284](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210607152808284.png)

![image-20210607152820951](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210607152820951.png)

4、多继承有虚函数覆盖的情况

```c++
#include <iostream>
using namespace std;

class Base1
{
public:
    virtual void fun1() { cout << "Base1::fun1()" << endl; }
    virtual void B1_fun2() { cout << "Base1::B1_fun2()" << endl; }
    virtual void B1_fun3() { cout << "Base1::B1_fun3()" << endl; }
};
class Base2
{
public:
    virtual void fun1() { cout << "Base2::fun1()" << endl; }
    virtual void B2_fun2() { cout << "Base2::B2_fun2()" << endl; }
    virtual void B2_fun3() { cout << "Base2::B2_fun3()" << endl; }
};
class Base3
{
public:
    virtual void fun1() { cout << "Base3::fun1()" << endl; }
    virtual void B3_fun2() { cout << "Base3::B3_fun2()" << endl; }
    virtual void B3_fun3() { cout << "Base3::B3_fun3()" << endl; }
};

class Derive : public Base1, public Base2, public Base3
{
public:
    virtual void fun1() { cout << "Derive::fun1()" << endl; }
    virtual void D_fun2() { cout << "Derive::D_fun2()" << endl; }
    virtual void D_fun3() { cout << "Derive::D_fun3()" << endl; }
};

int main(){
    Base1 *p1 = new Derive();
    Base2 *p2 = new Derive();
    Base3 *p3 = new Derive();
    p1->fun1(); // Derive::fun1()
    p2->fun1(); // Derive::fun1()
    p3->fun1(); // Derive::fun1()
    return 0;
}
```

![image-20210607152906374](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210607152906374.png)

![image-20210607152917582](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210607152917582.png)

### 如何禁止构造函数的使用？

为类的构造函数增加 `= delete` 修饰符，可以达到虽然声明了构造函数但禁止使用的目的。

```c++
#include <iostream>

using namespace std;

class A {
public:
    int var1, var2;
    A(){
        var1 = 10;
        var2 = 20;
    }
    A(int tmp1, int tmp2) = delete;
};

int main()
{
    A ex1;    
    A ex2(12,13); // error: use of deleted function 'A::A(int, int)'
    return 0;
}
```

说明：上述代码中，使用了已经删除 `delete` 的构造函数，程序出现错误。

### 构造函数、析构函数是否需要定义成虚函数？为什么？

构造函数一般不定义为虚函数，原因：

1. 从存储空间的角度考虑：构造函数是在实例化对象的时候进行调用，如果此时将构造函数定义成虚函数，需要通过访问该对象所在的内存空间才能进行虚函数的调用（因为需要通过指向虚函数表的指针调用虚函数表，虽然**虚函数表在编译时就有了**，但是没有虚函数的指针，虚函数的指针只有在创建了对象才有），但是此时该对象还未创建，便无法进行虚函数的调用。所以构造函数不能定义成虚函数。
4. 从类型上考虑：在创建对象时需要明确其类型。

析构函数一般定义成虚函数，原因：

析构函数定义成虚函数是为了防止内存泄漏，因为当基类的指针或者引用指向或绑定到派生类的对象时，如果未将基类的析构函数定义成虚函数，会调用基类的析构函数，那么只能将基类的成员所占的空间释放掉，派生类中特有的就会无法释放内存空间导致内存泄漏。

### 如何减少构造函数开销？

在构造函数中使用类初始化列表，会减少调用默认的构造函数产生的开销，具体原因可以参考本章“为什么用成员初始化列表会快些？”这个问题。

```c++
class A
{
private:
    int val;
public:
    A()
    {
        cout << "A()" << endl;
    }
    A(int tmp)
    {
        val = tmp;
        cout << "A(int " << val << ")" << endl;
    }
};
class Test1
{
private:
    A ex;

public:
    Test1() : ex(1) // 成员列表初始化方式
    {
    }
};
```

### 多重继承(菱形继承)时会出现什么状况？如何解决？

多重继承（多继承）：是指从多个直接基类中产生派生类。

多重继承容易出现的问题：命名冲突和数据冗余问题。

举例:

```c++
#include <iostream>
using namespace std;

// 间接基类
class Base1
{
public:
    int var1;
};

// 直接基类
class Base2 : public Base1
{
public:
    int var2;
};

// 直接基类
class Base3 : public Base1
{
public:
    int var3;
};

// 派生类
class Derive : public Base2, public Base3
{
public:
    void set_var1(int tmp) { var1 = tmp; } // error: reference to 'var1' is ambiguous. 命名冲突
    void set_var2(int tmp) { var2 = tmp; }
    void set_var3(int tmp) { var3 = tmp; }
    void set_var4(int tmp) { var4 = tmp; }

private:
    int var4;
};

int main()
{
    Derive d;
    return 0;
}
```

![image-20210607160318754](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210607160318754.png)

上述代码中存的问题：
对于派生类 Derive 上述代码中存在直接继承关系和间接继承关系。

直接继承：Base2 、Base3
间接继承：Base1
对于派生类中继承的的成员变量 var1 ，从继承关系来看，实际上保存了两份，一份是来自基类 Base2，一份来自基类 Base3。因此，出现了命名冲突。

**解决方法 1：** 声明出现冲突的成员变量来源于哪个类

```c++
#include <iostream>
using namespace std;

// 间接基类
class Base1
{
public:
    int var1;
};

// 直接基类
class Base2 : public Base1
{
public:
    int var2;
};

// 直接基类
class Base3 : public Base1
{
public:
    int var3;
};

// 派生类 
class Derive : public Base2, public Base3
{
public:
    void set_var1(int tmp) { Base2::var1 = tmp; } // 这里声明成员变量来源于类 Base2，当然也可以声明来源于类 Base3
    void set_var2(int tmp) { var2 = tmp; }
    void set_var3(int tmp) { var3 = tmp; }
    void set_var4(int tmp) { var4 = tmp; }

private:
    int var4;
};

int main()
{
    Derive d;
    return 0;
}
```

**解决方法 2：** 虚继承

使用虚继承的目的：保证存在命名冲突的成员变量在派生类中只保留一份，即使间接基类中的成员在派生类中只保留一份。在菱形继承关系中，间接基类称为虚基类，直接基类和间接基类之间的继承关系称为虚继承。

实现方式：在继承方式前面加上 `virtual` 关键字。

```c++
#include <iostream>
using namespace std;

// 间接基类，即虚基类
class Base1
{
public:
    int var1;
};

// 直接基类 
class Base2 : virtual public Base1 // 虚继承
{
public:
    int var2;
};

// 直接基类 
class Base3 : virtual public Base1 // 虚继承
{
public:
    int var3;
};

// 派生类
class Derive : public Base2, public Base3
{
public:
    void set_var1(int tmp) { var1 = tmp; } 
    void set_var2(int tmp) { var2 = tmp; }
    void set_var3(int tmp) { var3 = tmp; }
    void set_var4(int tmp) { var4 = tmp; }

private:
    int var4;
};

int main()
{
    Derive d;
    return 0;
}
```

**虚基类**

虚继承的目的是让某个类做出声明，承诺愿意共享它的基类。其中，这个被共享的基类就称为虚基类（Virtual Base Class），本例中的 A 就是一个虚基类。在这种机制下，不论虚基类在继承体系中出现了多少次，在派生类中都只包含一份虚基类的成员。

![image-20210607165054647](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210607165054647.png)

**虚基类成员的可见性**

因为在虚继承的最终派生类中只保留了一份虚基类的成员，所以该成员可以被直接访问，不会产生二义性。此外，如果虚基类的成员只被一条派生路径覆盖，那么仍然可以直接访问这个被覆盖的成员。但是如果该成员被两条或多条路径覆盖了，那就不能直接访问了，此时必须指明该成员属于哪个类。

以图中的菱形继承为例，假设 A 定义了一个名为 x 的成员变量，当我们在 D 中直接访问 x 时，会有三种可能性：

- 如果 B 和 C 中都没有 x 的定义，那么 x 将被解析为 A 的成员，此时不存在二义性。
- 如果 B 或 C 其中的一个类定义了 x，也不会有二义性，**派生类的 x 比虚基类的 x 优先级更高**。
- 如果 B 和 C 中都定义了 x，那么直接访问 x 将产生二义性问题。

### C++虚继承时的构造函数

在虚继承中，虚基类是由最终的派生类初始化的，换句话说，最终派生类的构造函数必须要调用虚基类的构造函数。对最终的派生类来说，虚基类是间接基类，而不是直接基类。这跟普通继承不同，在普通继承中，派生类构造函数中只能调用直接基类的构造函数，不能调用间接基类的。

下面我们以菱形继承为例来演示构造函数的调用：

```c++
#include <iostream>
using namespace std;
//虚基类A
class A{
public:
    A(int a);
protected:
    int m_a;
};
A::A(int a): m_a(a){ }
//直接派生类B
class B: virtual public A{
public:
    B(int a, int b);
public:
    void display();
protected:
    int m_b;
};
B::B(int a, int b): A(a), m_b(b){ }
void B::display(){
    cout<<"m_a="<<m_a<<", m_b="<<m_b<<endl;
}
//直接派生类C
class C: virtual public A{
public:
    C(int a, int c);
public:
    void display();
protected:
    int m_c;
};
C::C(int a, int c): A(a), m_c(c){ }
void C::display(){
    cout<<"m_a="<<m_a<<", m_c="<<m_c<<endl;
}
//间接派生类D
class D: public B, public C{
public:
    D(int a, int b, int c, int d);
public:
    void display();
private:
    int m_d;
};
D::D(int a, int b, int c, int d): A(a), B(90, b), C(100, c), m_d(d){ }
void D::display(){
    cout<<"m_a="<<m_a<<", m_b="<<m_b<<", m_c="<<m_c<<", m_d="<<m_d<<endl;
}
int main(){
    B b(10, 20);
    b.display();
   
    C c(30, 40);
    c.display();
    D d(50, 60, 70, 80);
    d.display();
    return 0;
}
/*
运行结果：
m_a=10, m_b=20
m_a=30, m_c=40
m_a=50, m_b=60, m_c=70, m_d=80
*/
```

需要关注的是构造函数的执行顺序。虚继承时构造函数的执行顺序与普通继承时不同：在最终派生类的构造函数调用列表中，不管各个构造函数出现的顺序如何，编译器总是先调用虚基类的构造函数，再按照出现的顺序调用其他的构造函数；而对于普通继承，就是按照构造函数出现的顺序依次调用的。

修改本例中第 50 行代码，改变构造函数出现的顺序：

```
D::D(int a, int b, int c, int d): B(90, b), C(100, c), A(a), m_d(d){ }
```

虽然我们将 A() 放在了最后，但是编译器仍然会先调用 A()，然后再调用 B()、C()，因为 A() 是虚基类的构造函数，比其他构造函数优先级高。如果没有使用虚继承的话，那么编译器将按照出现的顺序依次调用 B()、C()、A()。

### C++ 虚继承下的内存模型

 对于普通继承，基类子对象始终位于派生类对象的前面（也即基类成员变量始终在派生类成员变量的前面），而且不管继承层次有多深，它相对于派生类对象顶部的偏移量是固定的。请看下面的例子：

```c++
class {
protected:
    int m_a1;
    int m_a2;
};

class B: public A{
protected:
    int b1;
    int b2;
};

class C: public B{
protected:
    int c1;
    int c2;
};

class D: public C{
protected:
    int d1;
    int d2;
};

int main(){
    A obj_a;
    B obj_b;
    C obj_c;
    D obj_d;

    return 0;
}
```

`obj_a`、`obj_b`、`obj_c`、`obj_d` 的内存模型如下所示：

![image-20210607172122842](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210607172122842.png)

A 是最顶层的基类，在派生类 B、C、D 的对象中，A 类子对象始终位于最前面，偏移量是固定的，为 0。`b1`、`b2` 是派生类 B 的新增成员变量，它们的偏移量也是固定的，分别为 8 和 12。`c1`、`c2`、`d1`、`d2` 也是同样的道理。

编译器在知道对象首地址的情况下，通过计算偏移来存取成员变量。**对于普通继承，基类成员变量的偏移是固定的，不会随着继承层级的增加而改变，存取起来非常方便**。

 而对于虚继承，恰恰和普通继承相反，**大部分编译器会把基类成员变量放在派生类成员变量的后面，这样随着继承层级的增加，基类成员变量的偏移就会改变，就得通过其他方案来计算偏移量**。

下面我们来一步一步地分析虚继承时的对象内存模型。

1) 修改上面的代码，使得 A 是 B 的虚基类：

```
class B: virtual public A
```

此时 obj_b、obj_c、obj_d 的内存模型就会发生变化，如下图所示：

![image-20210607172630862](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210607172630862.png)

2) 再假设 A 是 B 的虚基类，B 又是 C 的虚基类，那么各个对象的内存模型如下图所示：

![image-20210607172708168](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210607172708168.png)

- 不带阴影的一部分偏移量固定，不会随着继承层次的增加而改变，称为固定部分；
- 带有阴影的一部分是虚基类的子对象，偏移量会随着继承层次的增加而改变，称为共享部分。

当要访问对象的成员变量时，需要知道对象的首地址和变量的偏移，对象的首地址很好获得，关键是变量的偏移。对于固定部分，偏移是不变的，很好计算；而对于共享部分，偏移会随着继承层次的增加而改变，这就需要设计一种方案，在偏移不断变化的过程中准确地计算偏移。各个编译器正是在设计这一方案时出现了分歧，不同的编译器设计了不同的方案来计算共享部分的偏移。

对于虚继承，将派生类分为**固定部分**和**共享部分**，并把共享部分放在最后，几乎所有的编译器都在这一点上达成了共识。主要的分歧就是如何计算共享部分的偏移，可谓是百花齐放，没有统一标准。

**`cfront`解决方案**

早期的 `cfront` 编译器会在派生类对象中安插一些指针，每个指针指向一个虚基类的子对象，要存取继承来的成员变量，可以使用指针间接完成。

1) 如果 A 是 B 的虚基类，那么各个对象的实际内存模型如下所示：

![image-20210607172952586](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210607172952586.png)

编译器会在直接派生类的对象 `obj_b` 中安插一个指针，指向虚基类 A 的起始位置，并且这个指针的偏移是固定的，不会随着继承层次的增加而改变。当要访问 `a1`、`a2` 时，要先通过对象指针找到 pa，再通过 pa 找到 `a1`、`a2`，这样一来就比没有虚继承时多了一层间接。

2) 如果 A 是 B 的虚基类，同时 B 也是 C 的虚基类，那么各个对象的实际内存模型如下所示：

![image-20210607173044315](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210607173044315.png)

 当要访问 `a1`、`a2` 时，要先通过对象指针找到 `pb`，再通过 `pb` 找到 `pa`，最后才能通过 pa 找到 `a1`、`a2`，这样一来就比没有虚继承时多了两层间接。

 通过上面的分析可以发现，这种方案的一个缺点就是，随着虚继承层次的增加，访问顶层基类需要的间接转换会越来越多，效率越来越低。

 这种方案另外的一个缺点是：**当有多个虚基类时，派生类要为每个虚基类都安插一个指针，会增加对象的体积**。

**`VC`解决方案**

`cfront` 的后来者 `VC` 尝试对上面的方案进行了改进，一定程度上弥补了它的不足。

`VC` 引入了**虚基类表**，如果某个派生类有一个或多个虚基类，编译器就会在派生类对象中安插一个指针，指向虚基类表。虚基类表其实就是一个数组，数组中的元素存放的是各个虚基类的偏移。

 假设 A 是 B 的虚基类，那么各对象的内存模型如下图所示：

![image-20210607173305607](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210607173305607.png)

假设 A 是 B 的虚基类，同时 B 又是 C 的虚基类，那么各对象的内存模型如下图所示：

![image-20210607173517479](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210607173517479.png)

 虚继承表中保存的是所有虚基类（包括直接继承和间接继承到的）相对于当前对象的偏移，这样通过派生类指针访问虚基类的成员变量时，不管继承层次都多深，只需要一次间接转换就可以。

 另外，这种方案还可以避免有多个虚基类时让派生类对象额外背负过多的指针。例如，假设 A、B、C、D 类的继承关系为：

![image-20210607173833764](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210607173833764.png)

那么 obj_d 的内存模型如下图所示： 

![image-20210607173846698](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210607173846698.png)

如此一来，D 类虽然有三个虚基类，但它的对象 obj_d 只需要额外背负一个指针。

### 为什么拷贝构造函数必须为引用（为什么拷贝构造函数不是值传递）

原因：避免拷贝构造函数无限制的递归，最终导致栈溢出。

举例说明：

```c++
#include <iostream>
using namespace std;

class A
{
private:
    int val;

public:
    A(int tmp) : val(tmp) // 带参数构造函数
    {
        cout << "A(int tmp)" << endl;
    }

    A(const A &tmp) // 拷贝构造函数
    {
        cout << "A(const A &tmp)" << endl;
        val = tmp.val;
    }

    A &operator=(const A &tmp) // 赋值函数（赋值运算符重载）
    {
        cout << "A &operator=(const A &tmp)" << endl;
        val = tmp.val;
        return *this;
    }

    void fun(A tmp)
    {
    }
};

int main()
{
    A ex1(1);
    A ex2(2);
    A ex3 = ex1;
    ex2 = ex1;
    ex2.fun(ex1);
    return 0;
}
/*
运行结果：
A(int tmp)
A(int tmp)
A(const A &tmp)
A &operator=(const A &tmp)
A(const A &tmp)
*/
```

说明 1：ex2 = ex1; 和 A ex3 = ex1; 为什么调用的函数不一样？
对象 ex2 已经实例化了，不需要构造，此时只是将 ex1 赋值给 ex2，只会调用赋值函数；但是 ex3 还没有实例化，因此调用的是拷贝构造函数，构造出 ex3，而不是赋值函数，这里涉及到构造函数的隐式调用。

说明 2：如果拷贝构造函数中形参不是引用类型，A ex3 = ex1;会出现什么问题？
构造 ex3，实质上是 ex3.A(ex1);，假如拷贝构造函数参数不是引用类型，那么将使得 ex3.A(ex1); 相当于 ex1 作为函数 A(const A tmp)的形参，在参数传递时相当于 A tmp = ex1，因为 tmp 没有被初始化，所以在 A tmp = ex1 中继续调用拷贝构造函数，接下来的是构造 tmp，也就是 tmp.A(ex1) ，必然又会有 ex1 作为函数 A(const A tmp); 的形参，在参数传递时相当于即 A tmp = ex1，那么又会触发拷贝构造函数，就这下永远的递归下去。

说明 3：为什么 ex2.fun(ex1); 会调用拷贝构造函数？
ex1 作为参数传递给 fun 函数， 即 A tmp = ex1;，这个过程会调用拷贝构造函数进行初始化。

### C++类对象的初始化顺序

构造函数调用顺序：

- 按照派生类继承基类的顺序，即派生列表中声明的顺序，依次调用基类的构造函数；
- 按照派生类中成员变量的声名顺序，依次调用派生类中成员变量所属类的构造函数；
- 执行派生类自身的构造函数。

综上可以得出，类对象的初始化顺序：基类构造函数–>派生类成员变量的构造函数–>自身构造函数
注：

- 基类构造函数的调用顺序与派生类的派生列表中的顺序有关；
- 成员变量的初始化顺序与声明顺序有关；
- 析构顺序和构造顺序相反

```c++
#include <iostream>
using namespace std;

class A
{
public:
    A() { cout << "A()" << endl; }
    ~A() { cout << "~A()" << endl; }
};

class B
{
public:
    B() { cout << "B()" << endl; }
    ~B() { cout << "~B()" << endl; }
};

class Test : public A, public B // 派生列表
{
public:
    Test() { cout << "Test()" << endl; }
    ~Test() { cout << "~Test()" << endl; }

private:
    B ex1;
    A ex2;
};

int main()
{
    Test ex;
    return 0;
}
/*
运行结果：
A()
B()
B()
A()
Test()
~Test()
~A()
~B()
~B()
~A()
*/
```

程序运行结果分析：

- 首先调用基类 A 和 B 的构造函数，按照派生列表 public A, public B 的顺序构造；
- 然后调用派生类 Test 的成员变量 ex1 和 ex2 的构造函数，按照派生类中成员变量声明的顺序构造；
- 最后调用派生类的构造函数；
- 接下来调用析构函数，和构造函数调用的顺序相反。

### 如何禁止一个类被实例化

方法一：

- 在类中定义一个纯虚函数，使该类成为抽象基类，因为不能创建抽象基类的实例化对象；

方法二：

- 将类的构造函数声明为私有 `private`

### 为什么用成员初始化列表会快一些？

说明：数据类型可分为内置类型和用户自定义类型（类类型），对于用户自定义类型，利用成员初始化列表效率高。

原因：用户自定义类型如果使用类初始化列表，**直接调用该成员变量对应的构造函数即完成初始化**；如果在构造函数中初始化，因为 C++ 规定，**对象的成员变量的初始化动作发生在进入构造函数本体之前，**那么<font color = red>在执行构造函数的函数体之前首先调用默认的构造函数为成员变量设初值，在进入函数体之后，调用该成员变量对应的构造函数</font>。因此，使用列表初始化会减少调用默认的构造函数的过程，效率高。

```c++
#include <iostream>
using namespace std;
class A
{
private:
    int val;
public:
    A()
    {
        cout << "A()" << endl;
    }
    A(int tmp)
    {
        val = tmp;
        cout << "A(int " << val << ")" << endl;
    }
};

class Test1
{
private:
    A ex;

public:
    Test1() : ex(1) // 成员列表初始化方式
    {
    }
};

class Test2
{
private:
    A ex;

public:
    Test2() // 函数体中赋值的方式
    {
        ex = A(2);
    }
};
int main()
{
    Test1 ex1;
    cout << endl;
    Test2 ex2;
    return 0;
}
/*
运行结果：
A(int 1)

A()
A(int 2)
*/
```

说明：
从程序运行结果可以看出，使用成员列表初始化的方式会省去调用默认的构造函数的过程。

### 实例化一个对象需要哪几个阶段

1. 分配空间
   创建类对象首先要为该对象分配内存空间。不同的对象，为其分配空间的时机未必相同。全局对象、静态对象、分配在栈区域内的对象，在编译阶段进行内存分配；存储在堆空间的对象，是在运行阶段进行内存分配。

2. 初始化
   首先明确一点：初始化不同于赋值。初始化发生在赋值之前，初始化随对象的创建而进行，而赋值是在对象创建好后，为其赋上相应的值。这一点可以联想下上一个问题中提到：初始化列表先于构造函数体内的代码执行，初始化列表执行的是数据成员的初始化过程，这个可以从成员对象的构造函数被调用看的出来。

3. 赋值
   对象初始化完成后，可以对其进行赋值。对于一个类的对象，其成员变量的赋值过程发生在类的构造函数的函数体中。当执行完该函数体，也就意味着类对象的实例化过程完成了。（总结：构造函数实现了对象的初始化和赋值两个过程，对象的初始化是通过初始化列表来完成，而对象的赋值则才是通过构造函数的函数体来实现。

4. 注：对于拥有虚函数的类的对象，还需要给虚表指针赋值。

   没有继承关系的类，分配完内存后，首先给虚表指针赋值，然后再列表初始化以及执行构造函数的函数体，即上述中的初始化和赋值操作。

   有继承关系的类，分配内存之后，首先进行基类的构造过程，然后给该派生类的虚表指针赋值，最后再列表初始化以及执行构造函数的函数体，即上述中的初始化和赋值操作。

### 友元函数的作用及使用场景

作用：友元提供了不同类的成员函数之间、类的成员函数与一般函数之间进行数据共享的机制。通过友元，一个不同函数或另一个类中的成员函数可以访问类中的私有成员和保护成员。

使用场景：

1、普通函数定义为友元函数，使普通函数能够访问类的私有成员。

2、友元类：类之间共享数据

```c++
#include <iostream>

using namespace std;

class A
{
    friend class B;

public:
    A() : var(10){}
    A(int tmp) : var(tmp) {}
    void fun()
    {
        cout << "fun():" << var << endl;
    }

private:
    int var;
};

class B
{
public:
    B() {}
    void fun()
    {
        cout << "fun():" << ex.var << endl; // 访问类 A 中的私有成员
    }

private:
    A ex;
};

int main()
{
    B ex;
    ex.fun(); // fun():10
    return 0;
}
```

### 深拷贝和浅拷贝

如果一个类拥有资源，该类的对象进行复制时，如果资源重新分配，就是深拷贝，否则就是浅拷贝。

深拷贝：该对象和原对象占用不同的内存空间，既拷贝存储在栈空间中的内容，又拷贝存储在堆空间中的内容。

浅拷贝：该对象和原对象占用同一块内存空间，仅拷贝类中位于栈空间中的内容。

当类的成员变量中有指针变量时，最好使用深拷贝。因为当两个对象指向同一块内存空间，如果使用浅拷贝，当其中一个对象的删除后，该块内存空间就会被释放，另外一个对象指向的就是垃圾内存。

**浅拷贝实例**

```c++
#include <iostream>

using namespace std;

class Test
{
private:
	int *p;

public:
	Test(int tmp)
	{
		this->p = new int(tmp);
		cout << "Test(int tmp)" << endl;
	}
	~Test()
	{
		if (p != NULL)
		{
			delete p;
		}
		cout << "~Test()" << endl;
	}
};

int main()
{
	Test ex1(10);	
	Test ex2 = ex1; 
	return 0;
}
/*
运行结果：
Test(int tmp)
~Test()
*/
```

说明：上述代码中，类对象 ex1、ex2 实际上是指向同一块内存空间，对象析构时，ex2 先将内存释放了一次，之后 析构对象 ex1 时又将这块已经被释放过的内存再释放一次。对同一块内存空间释放了两次，会导致程序崩溃。

**深拷贝实例：**

```c++
#include <iostream>

using namespace std;

class Test
{
private:
	int *p;

public:
	Test(int tmp)
	{
		p = new int(tmp);
		cout << "Test(int tmp)" << endl;
	}
	~Test()
	{
		if (p != NULL)
		{
			delete p;
		}
		cout << "~Test()" << endl;
	}
	Test(const Test &tmp) // 定义拷贝构造函数
	{
		p = new int(*tmp.p);
		cout << "Test(const Test &tmp)" << endl;
	}

};

int main()
{
	Test ex1(10);	
	Test ex2 = ex1; 
	return 0;
}
/*
Test(int tmp)
Test(const Test &tmp)
~Test()
~Test()
*/
```

### 编译时多态(静态多态)和运行时多态（动态多态）的区别

编译时多态：在程序编译过程中出现，发生在模板和函数重载中（泛型编程）。
运行时多态：在程序运行过程中出现，发生在继承体系中，是指通过基类的指针或引用访问派生类中的虚函数。

编译时多态和运行时多态的区别：

时期不同：编译时多态发生在程序编译过程中，运行时多态发生在程序的运行过程中；
实现方式不同：编译时多态运用泛型编程来实现，运行时多态借助虚函数来实现。

**静态类型和动态类型：**

静态类型：变量在声明时的类型，是在编译阶段确定的。静态类型不能更改。
动态类型：目前所指对象的类型，是在运行阶段确定的。动态类型可以更改。

**静态绑定和动态绑定：**

静态绑定是指程序在 编译阶段 确定对象的类型（静态类型）。
动态绑定是指程序在 运行阶段 确定对象的类型（动态类型）

### 实现一个类成员函数，要求不允许修改类的成员变量？

如果想达到一个类的成员函数不能修改类的成员变量，只需用 `const` 关键字来修饰该函数即可。

```c++
#include <iostream>

using namespace std;

class A
{
public:
    int var1, var2;
    A()
    {
        var1 = 10;
        var2 = 20;
    }
    void fun() const // 不能在 const 修饰的成员函数中修改成员变量的值，除非该成员变量用 mutable 修饰
    {
        var1 = 100; // error: assignment of member 'A::var1' in read-only object
    }
};

int main()
{
    A ex1;
    return 0;
}
```

### 如何让类不能被继承？

解决方法一：借助 `final` 关键字，用该关键字修饰的类不能被继承。

```c++
#include <iostream>

using namespace std;

class Base final
{
};

class Derive: public Base{ // error: cannot derive from 'final' base 'Base' in derived type 'Derive'

};

int main()
{
    Derive ex;
    return 0;
}
```

解决方法二：借助友元、虚继承和私有构造函数来实现

```c++
#include <iostream>
using namespace std;

template <typename T>
class Base{
    friend T;
private:
    Base(){
        cout << "base" << endl;
    }
    ~Base(){}
};

class B:virtual public Base<B>{   //一定注意 必须是虚继承
public:
    B(){
        cout << "B" << endl;
    }
};

class C:public B{
public:
    C(){}     // error: 'Base<T>::Base() [with T = B]' is private within this context
};


int main(){
    B b;  
    return 0;
}
```

说明：在上述代码中 B 类是不能被继承的类。

具体原因：

- 虽然 Base 类构造函数和析构函数被声明为私有 private，在 B 类中，由于 B 是 Base 的友元，因此可以访问 Base 类构造函数，从而正常创建 B 类的对象；
- B 类继承 Base 类采用虚继承的方式，创建 C 类的对象时，C 类的构造函数要负责 Base 类的构造，但是 Base 类的构造函数私有化了，C 类没有权限访问。因此，无法创建 C 类的对象， B 类是不能被继承的类。

注意：<font color = green>**在继承体系中，友元关系不能被继承**</font>，虽然 C 类继承了 B 类，B 类是 Base 类的友元，但是 C 类和 Base 类没有友元关系。

# 设计模式

设计模式分为三类：

创造型模式：单例模式、工厂模式、建造者模式、原型模式
结构型模式：适配器模式、桥接模式、外观模式、组合模式、装饰模式、享元模式、代理模式
行为型模式：责任链模式、命令模式、解释器模式、迭代器模式、中介者模式、备忘录模式、观察者模式、状态模式、策略模式、模板方法模式、访问者模式
下面介绍常见的几种设计模式：

单例模式：保证一个类仅有一个实例，并提供一个访问它的全局访问点。
工厂模式：包括简单工厂模式、抽象工厂模式、工厂方法模式
简单工厂模式：主要用于创建对象。用一个工厂来根据输入的条件产生不同的类，然后根据不同类的虚函数得到不同的结果。
工厂方法模式：修正了简单工厂模式中不遵守开放封闭原则。把选择判断移到了客户端去实现，如果想添加新功能就不用修改原来的类，直接修改客户端即可。
抽象工厂模式：定义了一个创建一系列相关或相互依赖的接口，而无需指定他们的具体类。
观察者模式：定义了一种一对多的关系，让多个观察对象同时监听一个主题对象，主题对象发生变化时，会通知所有的观察者，使他们能够更新自己。
装饰模式：动态地给一个对象添加一些额外的职责，就增加功能来说，装饰模式比生成派生类更为灵活。

### 什么是单例模式？如何实现？应用场景？

单例模式：保证类的实例化对象仅有一个，并且提供一个访问他的全局访问点。

应用场景：

- 表示文件系统的类，一个操作系统一定是只有一个文件系统，因此文件系统的类的实例有且仅有一个。

- 打印机打印程序的实例，一台计算机可以连接好几台打印机，但是计算机上的打印程序只有一个，就可以通过单例模式来避免两个打印作业同时输出到打印机。

实现方式：

单例模式可以通过全局或者静态变量的形式实现，这样比较简单，但是这样会影响封装性，难以保证别的代码不会对全局变量造成影响。

- 默认的构造函数、拷贝构造函数、赋值构造函数声明为私有的，这样禁止在类的外部创建该对象；
- 全局访问点也要定义成 静态类型的成员函数，没有参数，返回该类的指针类型。因为使用实例化对象的时候是通过类直接调用该函数，并不是先创建一个该类的对象，通过对象调用。

不安全的实现方式：
原因：考虑当两个线程同时调用 getInstance 方法，并且同时检测到 instance 是 NULL，两个线程会同时实例化对象，不符合单例模式的要求。

```c++
class Singleton{
private:
    static Singleton * instance;
    Singleton(){}
    Singleton(const Singleton& tmp){}
    Singleton& operator=(const Singleton& tmp){}
public:
    static Singleton* getInstance(){
        if(instance == NULL){
            instance = new Singleton();
        }
        return instance;
    }
};
Singleton* Singleton::instance = NULL;
```

分类：

- 懒汉模式：直到第一次用到类的实例时才去实例化，上面是懒汉实现。
- 饿汉模式：类定义的时候就实例化。

线程安全的懒汉模式实现：
  方法：加锁
  存在的问题：每次判断实例对象是否为空，都要被锁定，如果是多线程的话，就会造成大量线程阻塞。

```c++
class Singleton{
private:
    static pthread_mutex_t mutex;
    static Singleton * instance;
    Singleton(){
        pthread_mutex_init(&mutex, NULL); 
    }
    Singleton(const Singleton& tmp){}
    Singleton& operator=(const Singleton& tmp){}
public:
    static Singleton* getInstance(){
        pthread_mutex_lock(&mutex);
        if(instance == NULL){            
            instance = new Singleton();            
        }
        pthread_mutex_unlock(&mutex);
        return instance;
    }
};
Singleton* Singleton::instance = NULL;
pthread_mutex_t Singleton::mutex;
```

方法：**内部静态变量**，在全局访问点 `getInstance` 中定义静态实例。

```c++
class Singleton{
private:
    static pthread_mutex_t mutex;
    Singleton(){
        pthread_mutex_init(&mutex, NULL);
    }
    Singleton(const Singleton& temp){}
    Singleton& operator=(const Singleton& temp){}
public:
    static Singleton* getInstance(){ 
        static Singleton instance;
        return &instance;
    }
};
pthread_mutex_t Singleton::mutex; 
```

饿汉模式的实现：
饿汉模式本身就是线程安全的不用加锁。

```c++
class Singleton{
private:
    static Singleton* instance;
    Singleton(const Singleton& temp){}
    Singleton& operator=(const Singleton& temp){}
protected:
	 Singleton(){} 
public:
    static Singleton* getInstance(){ 
        return instance;    
    }
};
Singleton* Singleton::instance = new Singleton();
```

### 什么是工厂模式？如何实现？应用场景？

工厂模式：包括简单工厂模式、抽象工厂模式、工厂方法模式

简单工厂模式：主要用于创建对象。用一个工厂来根据输入的条件产生不同的类，然后根据不同类的虚函数得到不同的结果。

工厂方法模式：修正了简单工厂模式中不遵守开放封闭原则。把选择判断移到了客户端去实现，如果想添加新功能就不用修改原来的类，直接修改客户端即可。

抽象工厂模式：定义了一个创建一系列相关或相互依赖的接口，而无需指定他们的具体类。

1、简单工厂模式

主要用于创建对象。用一个工厂来根据输入的条件产生不同的类，然后根据不同类的虚函数得到不同的结果。

**应用场景**：

- 适用于针对不同情况创建不同类时，只需传入工厂类的参数即可，无需了解具体实现方法。例如：计算器中对于同样的输入，执行不同的操作：加、减、乘、除。

实现方式：

```c++
#include <iostream>
#include <vector>
using namespace std;

// Here is the product class
class Operation
{
public:
    int var1, var2;
    virtual double GetResult()
    {
        double res = 0;
        return res;
    }
};

class Add_Operation : public Operation
{
public:
    virtual double GetResult()
    {
        return var1 + var2;
    }
};

class Sub_Operation : public Operation
{
public:
    virtual double GetResult()
    {
        return var1 - var2;
    }
};

class Mul_Operation : public Operation
{
public:
    virtual double GetResult()
    {
        return var1 * var2;
    }
};

class Div_Operation : public Operation
{
public:
    virtual double GetResult()
    {
        return var1 / var2;
    }
};

// Here is the Factory class
class Factory
{
public:
    static Operation *CreateProduct(char op)
    {
        switch (op)
        {
        case '+':
            return new Add_Operation();

        case '-':
            return new Sub_Operation();

        case '*':
            return new Mul_Operation();

        case '/':
            return new Div_Operation();

        default:
            return new Add_Operation();
        }
    }
};

int main()
{
    int a, b;
    cin >> a >> b;
    Operation *p = Factory::CreateProduct('+');
    p->var1 = a;
    p->var2 = b;
    cout << p->GetResult() << endl;

    p = Factory::CreateProduct('*');
    p->var1 = a;
    p->var2 = b;
    cout << p->GetResult() << endl;

    return 0;
}
```

2、工厂方法模式

修正了简单工厂模式中不遵守开放封闭原则。把选择判断移到了客户端去实现，如果想添加新功能就不用修改原来的类，直接修改客户端即可。

应用场景：

- 一个类不知道它所需要的对象的类：在工厂方法模式中，客户端不需要知道具体产品类的类名，只需要知道所对应的工厂即可，具体的产品对象由具体工厂类创建；客户端需要知道创建具体产品的工厂类。
- 一个类通过其派生类来指定创建哪个对象：在工厂方法模式中，对于抽象工厂类只需要提供一个创建产品的接口，而由其派生类来确定具体要创建的对象，利用面向对象的多态性和里氏代换原则，在程序运行时，派生类对象将覆盖父类对象，从而使得系统更容易扩展。
- 将创建对象的任务委托给多个工厂派生类中的某一个，客户端在使用时可以无须关心是哪一个工厂派生类创建产品派生类，需要时再动态指定，可将具体工厂类的类名存储在配置文件或数据库中。

实现方式：

```c++
#include <iostream>
#include <vector>
using namespace std;

// Here is the product class
class Operation
{
public:
    int var1, var2;
    virtual double GetResult()
    {
        double res = 0;
        return res;
    }
};

class Add_Operation : public Operation
{
public:
    virtual double GetResult()
    {
        return var1 + var2;
    }
};

class Sub_Operation : public Operation
{
public:
    virtual double GetResult()
    {
        return var1 - var2;
    }
};

class Mul_Operation : public Operation
{
public:
    virtual double GetResult()
    {
        return var1 * var2;
    }
};

class Div_Operation : public Operation
{
public:
    virtual double GetResult()
    {
        return var1 / var2;
    }
};

class Factory
{
public:
    virtual Operation *CreateProduct() = 0;
};

class Add_Factory : public Factory
{
public:
    Operation *CreateProduct()
    {
        return new Add_Operation();
    }
};

class Sub_Factory : public Factory
{
public:
    Operation *CreateProduct()
    {
        return new Sub_Operation();
    }
};

class Mul_Factory : public Factory
{
public:
    Operation *CreateProduct()
    {
        return new Mul_Operation();
    }
};

class Div_Factory : public Factory
{
public:
    Operation *CreateProduct()
    {
        return new Div_Operation();
    }
};

int main()
{
    int a, b;
    cin >> a >> b;
    Add_Factory *p_fac = new Add_Factory();
    Operation *p_pro = p_fac->CreateProduct();
    p_pro->var1 = a;
    p_pro->var2 = b;
    cout << p_pro->GetResult() << endl;

    Mul_Factory *p_fac1 = new Mul_Factory();
    Operation *p_pro1 = p_fac1->CreateProduct();
    p_pro1->var1 = a;
    p_pro1->var2 = b;
    cout << p_pro1->GetResult() << endl;

    return 0;
}
```

3、抽象工厂模式

定义了一个创建一系列相关或相互依赖的接口，而无需指定他们的具体类。

**应用场景：**

- 一个系统不应当依赖于产品类实例如何被创建、组合和表达的细节，这对于所有类型的工厂模式都是重要的。
- 系统中有多于一个的产品族，而每次只使用其中某一产品族。
- 属于同一个产品族的产品将在一起使用，这一约束必须在系统的设计中体现出来。
- 产品等级结构稳定，设计完成之后，不会向系统中增加新的产品等级结构或者删除已有的产品等级结构。

**实现方式：**

```c++
#include <iostream>
#include <vector>
using namespace std;

// Here is the product class
class Operation_Pos
{
public:
    int var1, var2;
    virtual double GetResult()
    {
        double res = 0;
        return res;
    }
};

class Add_Operation_Pos : public Operation_Pos
{
public:
    virtual double GetResult()
    {
        return var1 + var2;
    }
};

class Sub_Operation_Pos : public Operation_Pos
{
public:
    virtual double GetResult()
    {
        return var1 - var2;
    }
};

class Mul_Operation_Pos : public Operation_Pos
{
public:
    virtual double GetResult()
    {
        return var1 * var2;
    }
};

class Div_Operation_Pos : public Operation_Pos
{
public:
    virtual double GetResult()
    {
        return var1 / var2;
    }
};
/*********************************************************************************/
class Operation_Neg
{
public:
    int var1, var2;
    virtual double GetResult()
    {
        double res = 0;
        return res;
    }
};

class Add_Operation_Neg : public Operation_Neg
{
public:
    virtual double GetResult()
    {
        return -(var1 + var2);
    }
};

class Sub_Operation_Neg : public Operation_Neg
{
public:
    virtual double GetResult()
    {
        return -(var1 - var2);
    }
};

class Mul_Operation_Neg : public Operation_Neg
{
public:
    virtual double GetResult()
    {
        return -(var1 * var2);
    }
};

class Div_Operation_Neg : public Operation_Neg
{
public:
    virtual double GetResult()
    {
        return -(var1 / var2);
    }
};
/*****************************************************************************************************/

// Here is the Factory class
class Factory
{
public:
    virtual Operation_Pos *CreateProduct_Pos() = 0;
    virtual Operation_Neg *CreateProduct_Neg() = 0;
};

class Add_Factory : public Factory
{
public:
    Operation_Pos *CreateProduct_Pos()
    {
        return new Add_Operation_Pos();
    }
    Operation_Neg *CreateProduct_Neg()
    {
        return new Add_Operation_Neg();
    }
};

class Sub_Factory : public Factory
{
public:
    Operation_Pos *CreateProduct_Pos()
    {
        return new Sub_Operation_Pos();
    }
    Operation_Neg *CreateProduct_Neg()
    {
        return new Sub_Operation_Neg();
    }
};

class Mul_Factory : public Factory
{
public:
    Operation_Pos *CreateProduct_Pos()
    {
        return new Mul_Operation_Pos();
    }
    Operation_Neg *CreateProduct_Neg()
    {
        return new Mul_Operation_Neg();
    }
};

class Div_Factory : public Factory
{
public:
    Operation_Pos *CreateProduct_Pos()
    {
        return new Div_Operation_Pos();
    }
    Operation_Neg *CreateProduct_Neg()
    {
        return new Div_Operation_Neg();
    }
};

int main()
{
    int a, b;
    cin >> a >> b;
    Add_Factory *p_fac = new Add_Factory();
    Operation_Pos *p_pro = p_fac->CreateProduct_Pos();
    p_pro->var1 = a;
    p_pro->var2 = b;
    cout << p_pro->GetResult() << endl;

    Add_Factory *p_fac1 = new Add_Factory();
    Operation_Neg *p_pro1 = p_fac1->CreateProduct_Neg();
    p_pro1->var1 = a;
    p_pro1->var2 = b;
    cout << p_pro1->GetResult() << endl;

    return 0;
}
```

### 什么是观察者模式？如何实现？应用场景？

观察者模式：定义一种一（被观察类）对多（观察类）的关系，让多个观察对象同时监听一个被观察对象，被观察对象状态发生变化时，会通知所有的观察对象，使他们能够更新自己的状态。

观察者模式中存在两种角色：

观察者：内部包含被观察者对象，当被观察者对象的状态发生变化时，更新自己的状态。（接收通知更新状态）
被观察者：内部包含了所有观察者对象，当状态发生变化时通知所有的观察者更新自己的状态。（发送通知）

应用场景：

当一个对象的改变需要同时改变其他对象，且不知道具体有多少对象有待改变时，应该考虑使用观察者模式；
一个抽象模型有两个方面，其中一方面依赖于另一方面，这时可以用观察者模式将这两者封装在独立的对象中使它们各自独立地改变和复用。

```c++
#include <iostream>
#include <string>
#include <list>
using namespace std;

class Subject;
//观察者 基类 （内部实例化了被观察者的对象sub）
class Observer
{
protected:
    string name;
    Subject *sub;

public:
    Observer(string name, Subject *sub)
    {
        this->name = name;
        this->sub = sub;
    }
    virtual void update() = 0;
};

class StockObserver : public Observer
{
public:
    StockObserver(string name, Subject *sub) : Observer(name, sub)
    {
    }
    void update();
};

class NBAObserver : public Observer
{
public:
    NBAObserver(string name, Subject *sub) : Observer(name, sub)
    {
    }
    void update();
};
//被观察者 基类 （内部存放了所有的观察者对象，以便状态发生变化时，给观察者发通知）
class Subject
{
protected:
    list<Observer *> observers;

public:
    string action; //被观察者对象的状态
    virtual void attach(Observer *) = 0;
    virtual void detach(Observer *) = 0;
    virtual void notify() = 0;
};

class Secretary : public Subject
{
    void attach(Observer *observer)
    {
        observers.push_back(observer);
    }
    void detach(Observer *observer)
    {
        list<Observer *>::iterator iter = observers.begin();
        while (iter != observers.end())
        {
            if ((*iter) == observer)
            {
                observers.erase(iter);
                return;
            }
            ++iter;
        }
    }
    void notify()
    {
        list<Observer *>::iterator iter = observers.begin();
        while (iter != observers.end())
        {
            (*iter)->update();
            ++iter;
        }
    }
};

void StockObserver::update()
{
    cout << name << " 收到消息：" << sub->action << endl;
    if (sub->action == "梁所长来了!")
    {
        cout << "我马上关闭股票，装做很认真工作的样子！" << endl;
    }
}

void NBAObserver::update()
{
    cout << name << " 收到消息：" << sub->action << endl;
    if (sub->action == "梁所长来了!")
    {
        cout << "我马上关闭NBA，装做很认真工作的样子！" << endl;
    }
}

int main()
{
    Subject *dwq = new Secretary();
    Observer *xs = new NBAObserver("xiaoshuai", dwq);
    Observer *zy = new NBAObserver("zouyue", dwq);
    Observer *lm = new StockObserver("limin", dwq);

    dwq->attach(xs);
    dwq->attach(zy);
    dwq->attach(lm);

    dwq->action = "去吃饭了！";
    dwq->notify();
    cout << endl;
    dwq->action = "梁所长来了!";
    dwq->notify();
    return 0;
}
```

# 计算机网络

## 协议层以及它们的服务类型

### OSI七层模型

![image-20210609144722870](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210609144722870.png)

① 应用层

应用层位于 OSI 参考模型的第七层，其作用是通过应用程序间的交互来完成特定的网络应用。该层协议定义了应用进程之间的交互规则，通过不同的应用层协议为不同的网络应用提供服务。例如域名系统 DNS，支持万维网应用的 HTTP 协议，电子邮件系统采用的 SMTP 协议等。在应用层交互的数据单元我们称之为报文。

② 表示层

表示层的作用是使通信的应用程序能够解释交换数据的含义，其位于 OSI 参考模型的第六层，向上为应用层提供服务，向下接收来自会话层的服务。该层提供的服务主要包括数据压缩，数据加密以及数据描述。这使得应用程序不必担心在各台计算机中表示和存储的内部格式差异。

③ 会话层

会话层就是负责建立、管理和终止表示层实体之间的通信会话。该层提供了数据交换的定界和同步功能，包括了建立检查点和恢复方案的方法。

④ 传输层

传输层的主要任务是为两台主机进程之间的通信提供服务。应用程序利用该服务传送应用层报文。该服务并不针对某一特定的应用，多种应用可以使用同一个传输层服务。由于一台主机可同时运行多个线程，因此传输层有复用和分用的功能。所谓复用就是指多个应用层进程可同时使用下面传输层的服务，分用和复用相反，是传输层把收到的信息分别交付上面应用层中的相应进程。

⑤ 网络层

两台计算机之间传送数据时其通信链路往往不止一条，所传输的信息甚至可能经过很多通信子网。网络层的主要任务就是选择合适的网间路由和交换节点，确保数据按时成功传送。在发送数据时，网络层把传输层产生的报文或用户数据报封装成分组和包向下传输到数据链路层。在网络层使用的协议是无连接的网际协议（Internet Protocol）和许多路由协议，因此我们通常把该层简单地称为 IP 层。路由器

⑥ 数据链路层

数据链路层通常也叫做链路层，在物理层和网络层之间。两台主机之间的数据传输，总是在一段一段的链路上传送的，这就需要使用专门的链路层协议。在两个相邻节点之间传送数据时，数据链路层将网络层交下来的 IP 数据报组装成帧，在两个相邻节点间的链路上传送帧。每一帧包括数据和必要的控制信息。通过控制信息我们可以知道一个帧的起止比特位置，此外，也能使接收端检测出所收到的帧有无差错，如果发现差错，数据链路层能够简单的丢弃掉这个帧，以避免继续占用网络资源。网卡、交换机

⑦ 物理层

作为 OSI 参考模型中最低的一层，物理层的作用是实现计算机节点之间比特流的透明传送，尽可能屏蔽掉具体传输介质和物理设备的差异。使其上面的数据链路层不必考虑网络的具体传输介质是什么。该层的主要任务是确定与传输媒体的接口的一些特性（机械特性、电气特性、功能特性，过程特性）。

### OSI 模型和 TCP/IP 模型异同比较

相同点

① OSI 参考模型与 TCP/IP 参考模型都采用了层次结构。

② 都能够提供面向连接和无连接两种通信服务机制。

不同点

① OSI 采用的七层模型； TCP/IP 是四层结构。

② TCP/IP 参考模型没有对网络接口层进行细分，只是一些概念性的描述； OSI 参考模型对服务和协议做了明确的区分。

③ OSI 先有模型，后有协议规范，适合于描述各种网络；TCP/IP 是先有协议集然后建立模型，不适用于非 TCP/IP 网络。

④ TCP/IP 一开始就提出面向连接和无连接服务，而 OSI 一开始只强调面向连接服务，直到很晚才开始制定无连接的服务标准。

⑤ OSI 参考模型虽然被看好，但将网络划分为七层，实现起来较困难；相反，TCP/IP 参考模型虽然有许多不尽人意的地方，但作为一种简化的分层结构还是比较成功的。

### OSI 和 TCP/IP 协议之间的对应关系

![image-20210609145037510](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210609145037510.png)

### 为什么 TCP/IP 去除了表示层和会话层

OSI 参考模型在提出时，他们的理想是非常好的，但实际上，由于会话层、表示层、应用层都是在应用程序内部实现的，最终产出的是一个应用数据包，而应用程序之间是几乎无法实现代码的抽象共享的，这也就造成 OSI 设想中的应用程序维度的分层是无法实现的，例如，我们几乎不会认为数据的压缩、加密算法算是一种协议，而会话的概念则更为抽象，难以用协议来进行描述，所以在后来的 TCP/IP 协议框架的设计中，便将表示层和会话层与应用层整合在一起，让整个过程更为清晰明了。

### 数据的封装过程

在发送主机端，一个应用层报文被传送到运输层。在最简单的情况下，运输层收取到报文并附上附加信息，该首部将被接收端的运输层使用。应用层报文和运输层首部信息一道构成了运输层报文段。附加的信息可能包括：允许接收端运输层向上向适当的应用程序交付报文的信息以及差错检测位信息。该信息让接收端能够判断报文中的比特是否在途中已被改变。运输层则向网络层传递该报文段，网络层增加了如源和目的端系统地址等网络层首部信息，生成了网络层数据报。该数据报接下来被传递给链路层，在数据链路层数据包添加发送端 MAC 地址和接收端 MAC 地址后被封装成数据帧，在物理层数据帧被封装成比特流，之后通过传输介质传送到对端。

## 应用层

### HTTP协议特点

1.支持客户/服务器模式。

2.简单快速：客户向服务器请求服务时，只需传送请求方法和路径。请求方法常用的有GET、HEAD、POST。每种方法规定了客户与服务器联系的类型不同。由于HTTP协议简单，使得HTTP服务器的程序规模小，因而通信速度很快。

3.灵活：HTTP允许传输任意类型的数据对象。正在传输的类型由Content-Type加以标记。

4.无连接：无连接的含义是限制每次连接只处理一个请求。服务器处理完客户的请求，并收到客户的应答后，即断开连接。采用这种方式可以节省传输时间。

5.无状态：HTTP协议是无状态协议。**无状态是指协议对于事务处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息，则它必须重传，**这样可能导致每次连接传送的数据量增大。另一方面，在服务器不需要先前信息时它的应答就较快。

### HTTP 头部包含哪些信息

**通用头部**

![image-20210609150100676](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210609150100676.png)

**请求头部**

![image-20210609150145866](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210609150145866.png)

**响应头部**

![image-20210609150212802](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210609150212802.png)

**实体头部**

![image-20210609150231059](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210609150231059.png)

Referer:首部字段Referer会告知服务器请求的原始资源的URI。

### HTTP  Keep-Alive的作用

在早期的 HTTP/1.0 中，浏览器每次 发起 HTTP 请求都要与服务器创建一个新的 TCP 连接，服务器完成请求处理后立即断开 TCP 连接，服务器不跟踪每个客户也不记录过去的请求。然而创建和关闭连接的过程需要消耗资源和时间，为了减少资源消耗，缩短响应时间，就需要重用连接。在 HTTP/1.1 版本中默认使用持久连接，在此之前的 HTTP 版本的默认连接都是使用非持久连接，如果想要在旧版本的 HTTP 协议上维持持久连接，则需要指定 connection 的首部字段的值为 Keep-Alive 来告诉对方这个请求响应完成后不要关闭，下一次咱们还用这个tcp连接请求继续交流，我们用一个示意图来更加生动的表示两者的区别：

![image-20210609150558989](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210609150558989.png)

对于非 Keep=Alive 来说，必须为每一个请求的对象建立和维护一个全新的连接。对于每一个这样的连接，客户机和服务器都要分配 TCP 的缓冲区和变量，这给服务器带来的严重的负担，因为一台 Web 服务器可能同时服务于数以百计的客户机请求。在 Keep-Alive 方式下，服务器在响应后保持该 TCP 连接打开，在同一个客户机与服务器之间的后续请求和响应报文可通过相同的连接进行传送。甚至位于同一台服务器的多个 Web 页面在从该服务器发送给同一个客户机时，可以在单个持久 TCP 连接上进行。

然而，Keep-Alive 并不是没有缺点的，当长时间的保持 TCP 连接时容易导致系统资源被无效占用，若对 Keep-Alive 模式配置不当，将有可能比非 Keep-Alive 模式带来的损失更大。因此，我们需要正确地设置 keep-alive timeout 参数，当 TCP 连接在传送完最后一个 HTTP 响应，该连接会保持 keepalive_timeout 秒，之后就开始关闭这个链接。

### HTTP 长连接短连接使用场景是什么

长连接：多用于操作频繁，点对点的通讯，而且客户端连接数目较少的情况。例如即时通讯、网络游戏等。

短连接：用户数目较多的Web网站的 HTTP 服务一般用短连接。例如京东，淘宝这样的大型网站一般客户端数量达到千万级甚至上亿，若采用长连接势必会使得服务端大量的资源被无效占用，所以一般使用的是短连接。

### 怎么知道 HTTP 的报文长度

当响应消息中存在 Content-Length 字段时，我们可以直接根据这个值来判断数据是否接收完成，例如客户端向服务器请求一个静态页面或者一张图片时，服务器能够很清楚的知道请求内容的大小，因此可以通过消息首部字段 Content- Length 来告诉客户端需要接收多少数据，**但是如果服务器预先不知道请求内容的大小，例如加载动态页面的时候，就需要使用 Transfer-Encoding: chunked 的方式来代替 Content-Length。**

分块传输编码（Chunked transfer encoding）是 HTTP/1.1 中引入的一种数据传输机制，其允许 HTTP 由服务器发送给客户端的数据可以分成多个部分，当数据分解成一系列数据块发送时，服务器就可以发送数据而不需要预先知道发送内容的总大小，每一个分块包含十六进制的长度值和数据，最后一个分块长度值为0，表示实体结束，客户机可以以此为标志确认数据已经接收完毕。

### HTTP 方法了解哪些

HTTP/1.0 定义了三种请求方法：GET, POST 和 HEAD 方法。

HTTP/1.1 增加了六种请求方法：OPTIONS, PUT, PATCH, DELETE, TRACE 和 CONNECT 方法

![image-20210609153242084](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210609153242084.png)

### GET 和 POST 的区别

get 提交的数据会放在 URL 之后，并且请求参数会被完整的保留在浏览器的记录里，由于参数直接暴露在 URL 中，可能会存在安全问题，因此往往用于获取资源信息。而 post 参数放在请求主体中，并且参数不会被保留，相比 get 方法，post 方法更安全，主要用于修改服务器上的资源。
get 请求只支持 URL 编码，post 请求支持多种编码格式。
get 只支持 ASCII 字符格式的参数，而 post 方法没有限制。
get 提交的数据大小有限制（这里所说的限制是针对浏览器而言的），而 post 方法提交的数据没限制
get 方法产生一个 TCP 数据包，post 方法产生两个（并不是所有的浏览器中都产生两个）。

对于GET方式的请求，浏览器会把http header和data一并发送出去，服务端响应200，请求成功。

对于POST方式的请求，浏览器会先发送http header给服务端，告诉服务端等一下会有数据过来，服务端响应100 continue，告诉浏览器我已经准备接收数据，浏览器再post发送一个data给服务端，服务端响应200，请求成功。

### GET 的长度限制是多少

HTTP 中的 GET 方法是通过 URL 传递数据的，而 URL 本身并没有对数据的长度进行限制，真正限制 GET 长度的是浏览器，例如 IE 浏览器对 URL 的最大限制为 2000多个字符，大概 2KB左右，像 Chrome, FireFox 等浏览器能支持的 URL 字符数更多，其中 FireFox 中 URL 最大长度限制为 65536 个字符，Chrome 浏览器中 URL 最大长度限制为 8182 个字符。并且这个长度不是只针对数据部分，而是针对整个 URL 而言，在这之中，不同的服务器同样影响 URL 的最大长度限制。因此对于特定的浏览器，GET的长度限制不同。

由于 POST 方法请求参数在请求主体中，理论上讲，post 方法是没有大小限制的，而真正起限制作用的是服务器处理程序的处理能力。

### HTTP 与 HTTPs 的工作方式【建立连接的过程】

HTTP

HTTP（Hyper Text Transfer Protocol: 超文本传输协议） 是一种简单的请求 - 响应协议，被用于在 Web 浏览器和网站服务器之间传递消息。HTTP 使用 TCP（而不是 UDP）作为它的支撑运输层协议。其默认工作在 TCP 协议 80 端口，HTTP 客户机发起一个与服务器的 TCP 连接，一旦连接建立，浏览器和服务器进程就可以通过套接字接口访问 TCP。客户机从套接字接口发送 HTTP 请求报文和接收 HTTP 响应报文。类似地，服务器也是从套接字接口接收 HTTP 请求报文和发送 HTTP 响应报文。其通信内容以明文的方式发送，不通过任何方式的数据加密。当通信结束时，客户端与服务器关闭连接。

HTTPS

HTTPS（Hyper Text Transfer Protocol over Secure Socket Layer）是以安全为目标的 HTTP 协议，在 HTTP 的基础上通过传输加密和身份认证的方式保证了传输过程的安全性。其工作流程如下：

① 客户端发起一个 HTTPS 请求，并连接到服务器的 443 端口，发送的信息主要包括自身所支持的算法列表和密钥长度等；

② 服务端将自身所支持的所有加密算法与客户端的算法列表进行对比并选择一种支持的加密算法，然后将它和其它密钥组件一同发送给客户端。

③ 服务器向客户端发送一个包含数字证书的报文，该数字证书中包含证书的颁发机构、过期时间、服务端的公钥等信息。

④ 最后服务端发送一个完成报文通知客户端 SSL 的第一阶段已经协商完成。

⑤ SSL 第一次协商完成后，客户端发送一个回应报文，报文中包含一个客户端生成的随机密码串，称为 pre_master_secre，并且该报文是经过证书中的公钥加密过的。

⑥ 紧接着客户端会发送一个报文提示服务端在此之后的报文是采用pre_master_secre 加密的。

⑦ 客户端向服务端发送一个 finish 报文，这次握手中包含第一次握手至今所有报文的整体校验值，最终协商是否完成取决于服务端能否成功解密。

⑧ 服务端同样发送与第 ⑥ 步中相同作用的报文，已让客户端进行确认，最后发送 finish 报文告诉客户端自己能够正确解密报文。

当服务端和客户端的 finish 报文交换完成之后，SSL 连接就算建立完成了，之后就进行和 HTTP 相同的通信过程，唯一不同的是在 HTTP 通信过程中并不是采用明文传输，而是采用对称加密的方式，其中对称密钥已经在 SSL 的建立过程中协商好了。

### HTTPS 和 HTTP 的区别

HTTP 协议以明文方式发送内容，数据都是未加密的，安全性较差。HTTPS 数据传输过程是加密的，安全性较好。
HTTP 和 HTTPS 使用的是完全不同的连接方式，用的端口也不一样，前者是 80 端口，后者是 443 端口。
HTTPS 协议需要到数字认证机构（Certificate Authority, CA）申请证书，一般需要一定的费用。
HTTP 页面响应比 HTTPS 快，主要因为 HTTP 使用 3 次握手建立连接，客户端和服务器需要握手 3 次，而 HTTPS 除了 TCP 的 3 次握手，还需要经历一个 SSL 协商过程。

### HTTPS的加密方式

**单向认证**

![img](https://upload-images.jianshu.io/upload_images/8525388-f4425aaa05ea3097.png?imageMogr2/auto-orient/strip|imageView2/2/w/1182/format/webp)

**双向认证**

![img](https://upload-images.jianshu.io/upload_images/8525388-020ed32bc59172dc.png?imageMogr2/auto-orient/strip|imageView2/2/w/1120/format/webp)

TLS：(Transport Layer Security，传输层安全协议)，用于两个应用程序之间提供保密性和数据完整性。

SSL：（Secure Socket Layer，安全套接字层），位于可靠的面向连接的网络层协议和应用层协议之间的一种协议层。SSL通过互相认证、使用数字签名确保完整性、使用加密确保私密性，以实现客户端和服务器之间的安全通讯。

SSL协议即用到了对称加密也用到了非对称加密(公钥加密)，在建立传输链路时，SSL首先对对称加密的密钥使用公钥进行非对称加密，链路建立好之后，SSL对传输内容使用对称加密。

HTTPS 采用对称加密和非对称加密相结合的方式（客户端使用公钥对对称加密的密钥进行加密，然后传递给服务端，服务端使用私钥进行解密确认密钥，开始传输数据。），首先使用 SSL/TLS 协议进行加密传输，为了弥补非对称加密的缺点（接收方并不确定所接受的公钥一定是对方的），HTTPS 采用证书来进一步加强非对称加密的安全性，通过非对称加密，客户端和服务端协商好之后进行通信传输的对称密钥，后续的所有信息都通过该对称秘钥进行加密解密，完成整个 HTTPS 的流程。

### 对称加密和非对称加密

对称加密
对称加密是指双方持有相同的密钥进行通信，加密速度快，可加密内容较大，用来加密会话过程中的消息。密钥容易被黑客拦截。

非对称加密

非对称加密使用了一对密钥，公钥和私钥。私钥由解密方安全保管，公钥可以发给任何请求它的人。数据使用公钥加密，私钥解密。因为私钥不通过网络发送出去，所以非对称加密的安全性很高。加密速度较慢，但能提供更好的身份认证技术，用来加密对称加密的密钥

### 客户端为什么信任第三方证书

一个证书中含有三个部分:"证书内容，散列算法，加密密文"，证书内容会被散列算法hash计算出hash值，然后使用CA机构提供的私钥进行RSA加密。当客户端发起请求时，服务器将该数字证书发送给客户端，客户端通过CA机构提供的公钥对加密密文进行解密获得散列值（数字签名），同时将证书内容使用相同的散列算法进行Hash得到另一个散列值，比对两个散列值，如果两者相等则说明证书没问题。

上述过程说明证书无法被篡改，我们考虑更严重的情况，例如中间人拿到了 CA 机构认证的证书，它想窃取网站 A 发送给客户端的信息，于是它成为中间人拦截到了 A 传给客户端的证书，然后将其替换为自己的证书。此时客户端浏览器收到的是被中间人掉包后的证书，但由于证书里包含了客户端请求的网站信息，因此客户端浏览器只需要把证书里的域名与自己请求的域名比对一下就知道证书有没有被掉包了。

### 证书和CA

数字证书是由权威的CA（Certificate Authority）机构给服务端进行颁发，CA机构通过服务端提供的相关信息生成证书，证书内容包含了持有人的相关信息，服务器的公钥，签署者签名信息（数字签名）等，最重要的是公钥在数字证书中。
 数字证书是如何保证公钥来自请求的服务器呢？数字证书上有持有人的相关信息，通过这点可以确定其不是一个中间人。

### 数字签名

信息传输的途中，我们的信息很有可能被第三方劫持篡改，所以我们需要保证信息的完整性，通用方法是使用散列算法如SHA1，MD5将传输内容hash一次获得hash值，即摘要。客户端使用服务端的公钥对摘要和信息内容进行加密，然后传输给服务端，服务端使用私钥进行解密获得原始内容和摘要值，这时服务端使用相同的hash算法对原始内容进行hash，然后与摘要值比对，如果一致，说明信息是完整的。

### HTTP 是不保存状态的协议,如何保存用户状态

① 基于 Session 实现的会话保持

在客户端第一次向服务器发送 HTTP 请求后，服务器会创建一个 Session 对象并**将客户端的身份信息以键值对的形式**存储下来，然后分配一个会话标识（SessionId）给客户端，这个会话标识一般保存在客户端 Cookie 中，之后每次该浏览器发送 HTTP 请求都会带上 Cookie 中的 SessionId 到服务器，服务器根据会话标识就可以将之前的状态信息与会话联系起来，从而实现会话保持。

优点：安全性高，因为状态信息保存在服务器端。

缺点：由于大型网站往往采用的是分布式服务器，浏览器发送的 HTTP 请求一般要先通过负载均衡器才能到达具体的后台服务器，倘若同一个浏览器两次 HTTP 请求分别落在不同的服务器上时，基于 Session 的方法就不能实现会话保持了。

【解决方法：采用中间件，例如 Redis，我们通过将 Session 的信息存储在 Redis 中，使得每个服务器都可以访问到之前的状态信息】

② 基于 Cookie 实现的会话保持

当服务器发送响应消息时，在 HTTP 响应头中设置 Set-Cookie 字段，用来存储客户端的状态信息。客户端解析出 HTTP 响应头中的字段信息，并根据其生命周期创建不同的 Cookie，这样一来每次浏览器发送 HTTP 请求的时候都会带上 Cookie 字段，从而实现状态保持。基于 Cookie 的会话保持与基于 Session 实现的会话保持最主要的区别是前者完全将会话状态信息存储在浏览器 Cookie 中。

优点：服务器不用保存状态信息， 减轻服务器存储压力，同时便于服务端做水平拓展。

缺点：该方式不够安全，因为状态信息存储在客户端，这意味着不能在会话中保存机密数据。除此之外，浏览器每次发起 HTTP 请求时都需要发送额外的 Cookie 到服务器端，会占用更多带宽。

拓展：Cookie被禁用了怎么办？

若遇到 Cookie 被禁用的情况，则可以通过重写 URL 的方式将会话标识放在 URL 的参数里，也可以实现会话保持。

### session 和 cookie的区别

**1.cooie和session特点**

Cookie特点：

cookie是将数据保存在浏览器端，是一门浏览器端的技术。由于数据保存在浏览器端，所以可以被任意的查看，安全性较低，但是可以长时间存储数据。cookie善于存储安全性要求较低，但是存储时间较长的数据。

Session特点：

session是将数据保存在服务器端，是一门服务器端的技术，数据保存在服务器端相对安全，但是服务器无法保留大量session对象，所以不能够长时间存储数据。服务器善于存储安全性要求较高，但是存储时间较短的数据。

区别：

cookie数据存放在客户的浏览器（客户端）上，session数据放在服务器上，但是服务端的session的实现对客户端的cookie有依赖关系的；

cookie不是很安全，别人可以分析存放在本地的COOKIE并进行COOKIE欺骗，考虑到安全应当使用session；

session会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能。考虑到减轻服务器性能方面，应当使用COOKIE；

单个cookie在客户端的限制是3K，就是说一个站点在客户端存放的COOKIE不能超过3K；

### 会话cookie和持久cookie的区别

如果不设置过期时间，则表示这个cookie生命周期为浏览器会话期间，只要关闭浏览器窗口，cookie就消失了。这种生命期为浏览会话期的cookie被称为会话cookie。会话cookie一般不保存在硬盘上而是保存在内存里。

如果设置了过期时间(setMaxAge(60*60*24))，浏览器就会把cookie保存到硬盘上，关闭后再次打开浏览器，这些cookie依然有效直到超过设定的过期时间。存储在硬盘上的cookie可以在不同的浏览器进程间共享，比如两个IE窗口。

### HTTP状态码

![image-20210609175355099](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210609175355099.png)

200 请求成功
204 请求成功但无内容返回
206 范围请求成功

301 永久重定向； 30(2|3|7)临时重定向，语义和实现有略微区别；
304 带if-modified-since 请求首部的条件请求，条件没有满足

400 语法错误（前端挨打）
401 需要认证信息
403 拒绝访问
404 找不到资源
412 除if-modified-since 以外的条件请求，条件未满足

500 服务器错误（后端挨打）
503 服务器宕机了（DevOps or IT 挨打）

### HTTP重定向

http 1.0规范中有2个重定向——301和302，在http 1.1规范中存在4个重定向——301、302、303和307。

![image-20210610104745639](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210610104745639.png)

![image-20210610104812600](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210610104812600.png)

![image-20210610104821716](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210610104821716.png)

① 状态码 301 和 302 的区别？

301：永久移动。请求的资源已被永久的移动到新的URI，旧的地址已经被永久的删除了。返回信息会包括新的URI，浏览器会自动定向到新的URI。今后新的请求都应使用新的URI代替。

302：临时移动。与301类似，客户端拿到服务端的响应消息后会跳转到一个新的 URL 地址。但资源只是临时被移动，旧的地址还在，客户端应继续使用原有URI。

### HTTP/1.1 和 HTTP/1.0 的区别

主要区别如下：

缓存处理：在 HTTP/1.0 中主要使用 header 里的 if-modified-Since, Expries 来做缓存判断的标准。而 HTTP/1.1 请求头中添加了更多与缓存相关的字段，从而支持更为灵活的缓存策略，例如 Entity-tag, If-Unmodified-Since, If-Match, If-None-Match 等可供选择的缓存头来控制缓存策略。

节约带宽： 当客户端请求某个资源时，HTTP/1.0 默认将该资源相关的整个对象传送给请求方，但很多时候可能客户端并不需要对象的所有信息。而在 HTTP/1.1 的请求头中引入了 range 头域，它允许只请求部分资源，其使得开发者可以多线程请求某一资源，从而充分的利用带宽资源，实现高效并发。

错误通知的管理：HTTP/1.1 在 1.0 的基础上新增了 24 个错误状态响应码，例如 414 表示客户端请求中所包含的 URL 地址太长，以至于服务器无法处理；410 表示所请求的资源已经被永久删除。

Host 请求头：早期 HTTP/1.0 中认为每台服务器都绑定一个唯一的 IP 地址并提供单一的服务，请求消息中的 URL 并没有传递主机名。而随着虚拟主机的出现，一台物理服务器上可以存在多个虚拟主机，并且它们共享同一个 IP 地址。为了支持虚拟主机，HTTP/1.1 中添加了 host 请求头，请求消息和响应消息中应声明这个字段，若请求消息中缺少该字段时服务端会响应一个 404 错误状态码。

长连接：HTTP/1.0 默认浏览器和服务器之间保持短暂连接，浏览器的每次请求都需要与服务器建立一个 TCP 连接，服务器完成后立即断开 TCP 连接。HTTP/1.1 默认使用的是持久连接，其支持在同一个 TCP 请求中传送多个 HTTP 请求和响应。此之前的 HTTP 版本的默认连接都是使用非持久连接，如果想要在旧版本的 HTTP 协议上维持持久连接，则需要指定 Connection 的首部字段的值为 Keep-Alive。

### HTTP1.1缓存机制

#### Last-Modified

`Last-Modified`和`If-Modified-Since`是一对的。

当浏览器第一次请求一个url时，服务器端的返回状态码为200，同时`HTTP响应头`会有一个Last-Modified标记着文件在服务器端最后被修改的时间。

浏览器第二次请求上次请求过的url时，浏览器会在`HTTP请求头`添加一个If-Modified-Since的标记，用来询问服务器该时间之后文件是否被修改过。

但是`Last-Modified`是http1.0的产物，有两个缺点：

- 只能精确到秒级别
- 内容完全没改变的资源文件，无法识别出来（只要修改时间变了，就算变动）。

所有就有了`ETag`。

If-Modified-Since用于确认代理或客户端拥有的本地资源的有效性。获取资源的更新日期时间，可通过确认首部字段Last-Modified来确定。

#### ETag

`ETag`解决了`Last-Modified`的缺点，http1.1的字段，优先级高于`Last-Modified`。

理论上是ETag优先于Last-Modified，但是Nginx一会把这两个一起开启一起验证，而且Nginx ETag的计算方式把最后修改时间也算进去了（所有这个算是弱ETag）。

> Nginx ETag计算方式：计算页面文件的最后修改时间，将文件最后修改时间的秒级Unix时间戳转为16进制作为etag的第一部分 计算页面文件的大小，将大小字节数转为16进制作为etag的第二部分。

ETag有两种类型：

- 强ETag

  强ETag值，不论实体发生多么细微的变化都会改变其值。

  强ETag表示形式：`"22FAA065-2664-4197-9C5E-C92EA03D0A16"`。

- 弱ETag

  弱 ETag 值只用于提示资源是否相同。只有资源发生了根本改变产 生差异时才会改变 ETag 。这时，会在字段值最开始处附加 `W/`。

  弱ETag表现形式：`W/"22FAA065-2664-4197-9C5E-C92EA03D0A16"`。

ETag和If-None-Match是一对:

当浏览器第一次请求一个url时，服务器端的返回状态码为200，同时HTTP响应头会有一个Etag，存放着服务器端生成的一个序列值。

浏览器第二次请求上次请求过的url时，浏览器会在HTTP请求头添加一个If-None-Match的标记，用来询问服务器该文件有没有被修改。

一般网站都会把`Last-Modified`和`ETag`一起用，同时对对比，两个条件都满足了才会返回304。

原理是这样的，当浏览器请求服务器的某项资源(A)时, 服务器根据A算出一个哈希值(3f80f-1b6-3e1cb03b)并通过 ETag 返回给浏览器，浏览器把"3f80f-1b6-3e1cb03b" 和 A 同时缓存在本地，当下次再次向服务器请求A时，会通过类似 `If-None-Match: "3f80f-1b6-3e1cb03b"` 的请求头把ETag发送给服务器，服务器再次计算A的哈希值并和浏览器返回的值做比较，如果发现A发生了变化就把A返回给浏览器(200)，如果发现A没有变化就给浏览器返回一个304未修改。这样通过控制浏览器端的缓存，可以节省服务器的带宽，因为服务器不需要每次都把全量数据返回给客户端。

### HTTP/1.X和HTTP/2.0的区别

![image-20210601215722931](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210601215722931.png)

![image-20210601215834081](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210601215834081.png)

（1）多路复用

​	HTTP2.0使用了多路复用的技术，做到同一个连接并发处理多个请求，而且并发请求的数量比HTTP1.1大了好几个数量级。HTTP1.1也可以多建立几个TCP连接，来支持处理更多并发的请求，但是创建TCP连接本身也是有开销的。

（2）头部压缩

HTTP1.1不支持header数据的压缩，HTTP2.0使用HPACK算法对header的数据进行压缩，这样数据体积小了，在网络上传输就会更快。

（3）服务器推送

 服务端推送是一种在客户端请求之前发送数据的机制。网页使用了许多资源：HTML、样式表、脚本、图片等等。在HTTP1.1中这些资源每一个都必须明确地请求。这是一个很慢的过程。浏览器从获取HTML开始，然后在它解析和评估页面的时候，增量地获取更多的资源。因为服务器必须等待浏览器做每一个请求，网络经常是空闲的和未充分使用的。

 为了改善延迟，HTTP2.0引入了server push，它允许服务端推送资源给浏览器，在浏览器明确地请求之前，免得客户端再次创建连接发送请求到服务器端获取。这样客户端可以直接从本地加载这些资源，不用再通过网络。

（4）强化安全

http2.0一般都跑在https上

### DNS 的作用和原理

DNS

DNS（Domain Name System）是域名系统的英文缩写，是一种组织成域层次结构的计算机和网络服务命名系统，用于 TCP/IP 网络。

DNS 的作用

通常我们有两种方式识别主机：通过主机名或者 IP 地址。人们喜欢便于记忆的主机名表示，而路由器则喜欢定长的、有着层次结构的 IP 地址。为了满足这些不同的偏好，我们就需要一种能够进行主机名到 IP 地址转换的目录服务，域名系统作为将域名和 IP 地址相互映射的一个分布式数据库，能够使人更方便地访问互联网。

DNS 域名解析原理

DNS 采用了分布式的设计方案，其域名空间采用一种树形的层次结构：

![image-20210610150337268](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210610150337268.png)

上图展示了 DNS 服务器的部分层次结构，从上到下依次为根域名服务器、顶级域名服务器和权威域名服务器。其实根域名服务器在因特网上有13个，大部分位于北美洲。第二层为顶级域服务器，这些服务器负责顶级域名（如 com、org、net、edu）和所有国家的顶级域名（如uk、fr、ca 和 jp）。在第三层为权威 DNS 服务器，因特网上具有公共可访问主机（例如 Web 服务器和邮件服务器）的每个组织机构必须提供公共可访问的 DNS 记录，这些记录由组织机构的权威 DNS 服务器负责保存，这些记录将这些主机的名称映射为 IP 地址。

我们以一个例子来了解 DNS 的工作原理，假设主机 A（IP 地址为 abc.xyz.edu） 想知道主机 B 的 IP 地址 （def.mn.edu），如下图所示，主机 A 首先向它的本地 DNS 服务器发送一个 DNS 查询报文。该查询报文含有被转换的主机名 def.mn.edu。本地 DNS 服务器将该报文转发到根 DNS 服务器，根 DNS 服务器注意到查询的 IP 地址前缀为 edu 后向本地 DNS 服务器返回负责 edu 的顶级域名服务器的 IP 地址列表。该本地 DNS 服务器则再次向这些 顶级域名服务器发送查询报文。该顶级域名服务器注意到 mn.edu 的前缀，并用权威域名服务器的 IP 地址进行响应。通常情况下，顶级域名服务器并不总是知道每台主机的权威 DNS 服务器的 IP 地址，而只知道中间的某个服务器，该中间 DNS 服务器依次能找到用于相应主机的 IP 地址，我们假设中间经历了权威服务器 ① 和 ②，最后找到了负责 def.mn.edu 的权威 DNS 服务器 ③，之后，本地 DNS 服务器直接向该服务器发送查询报文从而获得主机 B 的IP 地址。

![DNS.png](https://pic.leetcode-cn.com/1612458718-QcTlwM-DNS.png)

在上图中，IP 地址的查询其实经历了两种查询方式，分别是递归查询和迭代查询。

拓展：域名解析查询的两种方式

递归查询：如果主机所询问的本地域名服务器不知道被查询域名的 IP 地址，那么本地域名服务器就以 DNS 客户端的身份，向其他根域名服务器继续发出查询请求报文，即替主机继续查询，而不是让主机自己进行下一步查询，如上图步骤（1）和（10）。
迭代查询：当根域名服务器收到本地域名服务器发出的迭代查询请求报文时，要么给出所要查询的 IP 地址，要么告诉本地服务器下一步应该找哪个域名服务器进行查询，然后让本地服务器进行后续的查询，如上图步骤（2）~（9）。

### DNS 为什么用 UDP

更正确的答案是 DNS 既使用 TCP 又使用 UDP。

当进行区域传送（主域名服务器向辅助域名服务器传送变化的那部分数据）时会使用 TCP，因为数据同步传送的数据量比一个请求和应答的数据量要多，而 TCP 允许的报文长度更长，因此为了保证数据的正确性，会使用基于可靠连接的 TCP。

当客户端向 DNS 服务器查询域名 ( 域名解析) 的时候，一般返回的内容不会超过 UDP 报文的最大长度，即 512 字节。用 UDP 传输时，不需要经过 TCP 三次握手的过程，从而大大提高了响应速度，但这要求域名解析器和域名服务器都必须自己处理超时和重传从而保证可靠性。

### socket() 套接字有哪些

套接字（Socket）是对网络中不同主机上的应用进程之间进行双向通信的端点的抽象，网络进程通信的一端就是一个套接字，不同主机上的进程便是通过套接字发送报文来进行通信。例如 TCP 用主机的 IP 地址 + 端口号作为 TCP 连接的端点，这个端点就叫做套接字。

套接字主要有以下三种类型：

流套接字（SOCK_STREAM）：流套接字基于 TCP 传输协议，主要用于提供面向连接、可靠的数据传输服务。由于 TCP 协议的特点，使用流套接字进行通信时能够保证数据无差错、无重复传送，并按顺序接收，通信双方不需要在程序中进行相应的处理。
数据报套接字（SOCK_DGRAM）：和流套接字不同，数据报套接字基于 UDP 传输协议，对应于无连接的 UDP 服务应用。该服务并不能保证数据传输的可靠性，也无法保证对端能够顺序接收到数据。此外，通信两端不需建立长时间的连接关系，当 UDP 客户端发送一个数据给服务器后，其可以通过同一个套接字给另一个服务器发送数据。当用 UDP 套接字时，丢包等问题需要在程序中进行处理。
原始套接字（SOCK_RAW）：由于流套接字和数据报套接字只能读取 TCP 和 UDP 协议的数据，当需要传送非传输层数据包（例如 Ping 命令时用的 ICMP 协议数据包）或者遇到操作系统无法处理的数据包时，此时就需要建立原始套接字来发送。

### URI（统一资源标识符）和 URL（统一资源定位符）之间的区别

URL，即统一资源定位符 (Uniform Resource Locator )，URL 其实就是我们平时上网时输入的网址，它标识一个互联网资源，并指定对其进行操作或获取该资源的方法。例如 https://leetcode-cn.com/problemset/all/ 这个 URL，标识一个特定资源并表示该资源的某种形式是可以通过 HTTP 协议从相应位置获得。

从定义即可看出，URL 是 URI 的一个子集，两者都定义了资源是什么，而 URL 还定义了如何能访问到该资源。URI 是一种语义上的抽象概念，可以是绝对的，也可以是相对的，而URL则必须提供足够的信息来定位，是绝对的。简单地说，只要能唯一标识资源的就是 URI，在 URI 的基础上给出其资源的访问方式的就是 URL。

换句话说：URI：资源是什么？URL：资源是什么，如何获取？

### 网页解析全过程【用户输入网址到显示对应页面的全过程】

![image-20210610152101414](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210610152101414.png)

① DNS 解析：当用户输入一个网址并按下回车键的时候，浏览器获得一个域名，而在实际通信过程中，我们需要的是一个 IP 地址，因此我们需要先把域名转换成相应 IP 地址。

② TCP 连接：浏览器通过 DNS 获取到 Web 服务器真正的 IP 地址后，便向 Web 服务器发起 TCP 连接请求，通过 TCP 三次握手建立好连接后，浏览器便可以将 HTTP 请求数据发送给服务器了。

③ 发送 HTTP 请求：浏览器向 Web 服务器发起一个 HTTP 请求，HTTP 协议是建立在 TCP 协议之上的应用层协议，其本质是在建立起的TCP连接中，按照HTTP协议标准发送一个索要网页的请求。在这一过程中，会涉及到负载均衡等操作。

拓展：什么是负载均衡？

负载均衡，英文名为 Load Balance，其含义是指将负载（工作任务）进行平衡、分摊到多个操作单元上进行运行，例如 FTP 服务器、Web 服务器、企业核心服务器和其他主要任务服务器等，从而协同完成工作任务。负载均衡建立在现有的网络之上，它提供了一种透明且廉价有效的方法扩展服务器和网络设备的带宽、增加吞吐量、加强网络处理能力并提高网络的灵活性和可用性。

负载均衡是分布式系统架构设计中必须考虑的因素之一，例如天猫、京东等大型用户网站中为了处理海量用户发起的请求，其往往采用分布式服务器，并通过引入反向代理等方式将用户请求均匀分发到每个服务器上，而这一过程所实现的就是负载均衡。

④ 处理请求并返回：服务器获取到客户端的 HTTP 请求后，会根据 HTTP 请求中的内容来决定如何获取相应的文件，并将文件发送给浏览器。

⑤ 浏览器渲染：浏览器根据响应开始显示页面，首先解析 HTML 文件构建 DOM 树，然后解析 CSS 文件构建渲染树，等到渲染树构建完成后，浏览器开始布局渲染树并将其绘制到屏幕上。

⑥ 断开连接：客户端和服务器通过四次挥手终止 TCP 连接。

## 传输层

### 三次握手和四次挥手机制

**三次握手**

![image-20210610153314595](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210610153314595.png)

三次握手是 TCP 连接的建立过程。在握手之前，主动打开连接的客户端结束 CLOSE 阶段，被动打开的服务器也结束 CLOSE 阶段，并进入 LISTEN 阶段。随后进入三次握手阶段：

① 首先客户端向服务器发送一个 SYN 包，并等待服务器确认，其中：

标志位为 SYN，表示请求建立连接；
序号为 Seq = x（x 一般为 1）；
随后客户端进入 SYN-SENT 阶段。
② 服务器接收到客户端发来的 SYN 包后，对该包进行确认后结束 LISTEN 阶段，并返回一段 TCP 报文，其中：

标志位为 SYN 和 ACK，表示确认客户端的报文 Seq 序号有效，服务器能正常接收客户端发送的数据，并同意创建新连接；
序号为 Seq = y；
确认号为 Ack = x + 1，表示收到客户端的序号 Seq 并将其值加 1 作为自己确认号 Ack 的值，随后服务器端进入 SYN-RECV 阶段。
③ 客户端接收到发送的 SYN + ACK 包后，明确了从客户端到服务器的数据传输是正常的，从而结束 SYN-SENT 阶段。并返回最后一段报文。其中：

标志位为 ACK，表示确认收到服务器端同意连接的信号；
序号为 Seq = x + 1，表示收到服务器端的确认号 Ack，并将其值作为自己的序号值；
确认号为 Ack= y + 1，表示收到服务器端序号 seq，并将其值加 1 作为自己的确认号 Ack 的值。
随后客户端进入 ESTABLISHED。
当服务器端收到来自客户端确认收到服务器数据的报文后，得知从服务器到客户端的数据传输是正常的，从而结束 SYN-RECV 阶段，进入 ESTABLISHED 阶段，从而完成三次握手。

**四次挥手：**

![image-20210610153447747](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210610153447747.png)

四次挥手即 TCP 连接的释放，这里假设客户端主动释放连接。在挥手之前主动释放连接的客户端结束 ESTABLISHED 阶段，随后开始四次挥手：

① 首先客户端向服务器发送一段 TCP 报文表明其想要释放 TCP 连接，其中：

标记位为 FIN，表示请求释放连接；
序号为 Seq = u；
随后客户端进入 FIN-WAIT-1 阶段，即半关闭阶段，并且停止向服务端发送通信数据。
② 服务器接收到客户端请求断开连接的 FIN 报文后，结束 ESTABLISHED 阶段，进入 CLOSE-WAIT 阶段并返回一段 TCP 报文，其中：

标记位为 ACK，表示接收到客户端释放连接的请求；
序号为 Seq = v；
确认号为 Ack = u + 1，表示是在收到客户端报文的基础上，将其序号值加 1 作为本段报文确认号 Ack 的值；
随后服务器开始准备释放服务器端到客户端方向上的连接。
客户端收到服务器发送过来的 TCP 报文后，确认服务器已经收到了客户端连接释放的请求，随后客户端结束 FIN-WAIT-1 阶段，进入 FIN-WAIT-2 阶段。

③ 服务器端在发出 ACK 确认报文后，服务器端会将遗留的待传数据传送给客户端，待传输完成后即经过 CLOSE-WAIT 阶段，便做好了释放服务器端到客户端的连接准备，再次向客户端发出一段 TCP 报文，其中：

标记位为 FIN 和 ACK，表示已经准备好释放连接了；
序号为 Seq = w；
确认号 Ack = u + 1，表示是在收到客户端报文的基础上，将其序号 Seq 的值加 1 作为本段报文确认号 Ack 的值。
随后服务器端结束 CLOSE-WAIT 阶段，进入 LAST-ACK 阶段。并且停止向客户端发送数据。

④ 客户端收到从服务器发来的 TCP 报文，确认了服务器已经做好释放连接的准备，于是结束 FIN-WAIT-2 阶段，进入 TIME-WAIT 阶段，并向服务器发送一段报文，其中：

标记位为 ACK，表示接收到服务器准备好释放连接的信号；
序号为 Seq= u + 1，表示是在已收到服务器报文的基础上，将其确认号 Ack 值作为本段序号的值；
确认号为 Ack= w + 1，表示是在收到了服务器报文的基础上，将其序号 Seq 的值作为本段报文确认号的值。
随后客户端开始在 TIME-WAIT 阶段等待 2 MSL。服务器端收到从客户端发出的 TCP 报文之后结束 LAST-ACK 阶段，进入 CLOSED 阶段。由此正式确认关闭服务器端到客户端方向上的连接。客户端等待完 2 MSL 之后，结束 TIME-WAIT 阶段，进入 CLOSED 阶段，由此完成「四次挥手」。

### 如果三次握手的时候每次握手信息对方没有收到会怎么样

若第一次握手服务器未接收到客户端请求建立连接的数据包时，服务器不会进行任何相应的动作，而客户端由于在一段时间内没有收到服务器发来的确认报文， 因此会等待一段时间后重新发送 SYN 同步报文，若仍然没有回应，则重复上述过程直到发送次数超过最大重传次数限制后，建立连接的系统调用会返回 -1。

若第二次握手客户端未接收到服务器回应的 ACK 报文时，客户端会采取第一次握手失败时的动作，这里不再重复，而服务器端此时将阻塞在 accept() 系统调用处等待 client 再次发送 ACK 报文。

若第三次握手服务器未接收到客户端发送过来的 ACK 报文，同样会采取类似于客户端的超时重传机制，若重传次数超过限制后仍然没有回应，则 accep() 系统调用返回 -1，服务器端连接建立失败。但此时客户端认为自己已经连接成功了，因此开始向服务器端发送数据，但是服务器端的 accept() 系统调用已返回，此时没有在监听状态。因此服务器端接收到来自客户端发送来的数据时会发送 RST 报文给 客户端，消除客户端单方面建立连接的状态。

### 为什么要进行三次握手？两次握手可以吗？

一种解释：三次握手的主要目的是确认自己和对方的发送和接收都是正常的，从而保证了双方能够进行可靠通信。若采用两次握手，当第二次握手后就建立连接的话，此时客户端知道服务器能够正常接收到自己发送的数据，而服务器并不知道客户端是否能够收到自己发送的数据。

另一种解释：防止已失效的连接请求又传送到服务器端，因而产生错误。例如：client发出的第一个连接请求报文段并没有丢失，而是在某个网络结点长时间的滞留了，以致延误到连接释放以后的某个时间才到达server。本来这是一个早已失效的报文段。但server收到此失效的连接请求报文段后，就误认为是client再次发出的一个新的连接请求。于是就向client发出确认报文段，同意建立连接。假设不采用“三次握手”，那么只要server发出确认，新的连接就建立了。由于现在client并没有发出建立连接的请求，因此不会理睬server的确认，也不会向server发送数据。但server却以为新的运输连接已经建立，并一直等待client发来数据。这样，server的很多资源就白白浪费掉了。采用“三次握手”的办法可以防止上述现象发生。

### 为什么不是四次握手？

服务器将SYN和ACK一起传回，所以不需要进行四次握手。另外三次握手就能保证可靠传输，四次握手时徒劳的。

### 什么是半连接队列和全连接队列

服务器第一次收到客户端的 SYN 之后，就会处于 SYN_RCVD 状态，此时双方还没有完全建立其连接，服务器会把此种状态下请求连接放在一个**队列**里，我们把这种队列称之为**半连接队列**。

当然还有一个**全连接队列**，就是已经完成三次握手，建立起连接的就会放在全连接队列中。如果队列满了就有可能会出现丢包现象。

这里在补充一点关于**SYN-ACK 重传次数**的问题：
服务器发送完SYN-ACK包，如果未收到客户确认包，服务器进行首次重传，等待一段时间仍未收到客户确认包，进行第二次重传。如果重传次数超过系统规定的最大重传次数，系统将该连接信息从半连接队列中删除。
注意，每次重传等待的时间不一定相同，一般会是指数增长，例如间隔时间为 1s，2s，4s，8s......

### 三次握手过程中可以携带数据吗？

其实第三次握手的时候，是可以携带数据的。但是，**第一次、第二次握手不可以携带数据**

为什么这样呢?大家可以想一个问题，假如第一次握手可以携带数据的话，如果有人要恶意攻击服务器，那他每次都在第一次握手中的 SYN 报文中放入大量的数据。因为攻击者根本就不理服务器的接收、发送能力是否正常，然后疯狂着重复发 SYN 报文的话，这会让服务器花费很多时间、内存空间来接收这些报文。

也就是说，**第一次握手不可以放数据，其中一个简单的原因就是会让服务器更加容易受到攻击了。而对于第三次的话，此时客户端已经处于 ESTABLISHED 状态。对于客户端来说，他已经建立起连接了，并且也已经知道服务器的接收、发送能力是正常的了，所以能携带数据也没啥毛病。**

### SYN攻击是什么？

**服务器端的资源分配是在二次握手时分配的，而客户端的资源是在完成三次握手时分配的**，所以服务器容易受到SYN洪泛攻击。SYN攻击就是Client在短时间内伪造大量不存在的IP地址，并向Server不断地发送SYN包，Server则回复确认包，并等待Client确认，由于源地址不存在，因此Server需要不断重发直至超时，这些伪造的SYN包将长时间占用半连接队列，导致正常的SYN请求因为队列满而被丢弃，从而引起网络拥塞甚至系统瘫痪。SYN 攻击是一种典型的 DoS/DDoS 攻击。

检测 SYN 攻击非常的方便，当你在服务器上看到大量的半连接状态时，特别是源IP地址是随机的，基本上可以断定这是一次SYN攻击。在 Linux/Unix 上可以使用系统自带的 netstats 命令来检测 SYN 攻击。

```shell
netstat -n -p TCP | grep SYN_RECV
```

常见的防御 SYN 攻击的方法有如下几种：

- 缩短超时（SYN Timeout）时间
- 增加最大半连接数
- 过滤网关防护
- SYN cookies技术：在SYN cookies中，服务器的初始序列号是通过对客户端IP地址、客户端端囗、服务器IP地址和服务器端囗以及其他一些安全数值等要素进行hash运算，加密得到的，称之为cookie。服务器回复cookie（回复包的 SYN序列号）给客户端，如果收到客户端的ACK包，服务器将客户端的ACK序列号减去1得到cookie比较值，并将上述要素进行一次hash运算，看看是否等于此 cookie。如果相等，直接完成三次握手。

### 为什么四次挥手

建立一个连接需要三次握手，而终止一个连接要经过四次挥手（也有将四次挥手叫做四次握手的）。这由TCP的**半关闭**（half-close）造成的。所谓的半关闭，其实就是TCP提供了连接的一端在结束它的发送后还能接收来自另一端数据的能力。

释放 TCP 连接时之所以需要四次挥手，是因为 FIN 释放连接报文和 ACK 确认接收报文是分别在两次握手中传输的。 当主动方在数据传送结束后发出连接释放的通知，由于被动方可能还有必要的数据要处理，所以会先返回 ACK 确认收到报文。当被动方也没有数据再发送的时候，则发出连接释放通知，对方确认后才完全关闭TCP连接。

### CLOSE-WAIT 和 TIME-WAIT 的状态和意义

在服务器收到客户端关闭连接的请求并告诉客户端自己已经成功收到了该请求之后，服务器进入了 CLOSE-WAIT 状态，然而此时有可能服务端还有一些数据没有传输完成，因此不能立即关闭连接，而 **CLOSE-WAIT 状态就是为了保证服务器在关闭连接之前将待发送的数据发送完成。**

TIME-WAIT 发生在第四次挥手，当客户端向服务端发送 ACK 确认报文后进入该状态，若取消该状态，即客户端在收到服务端的 FIN 报文后立即关闭连接，此时服务端相应的端口并没有关闭，若客户端在相同的端口立即建立新的连接，则有可能接收到上一次连接中残留的数据包，可能会导致不可预料的异常出现。除此之外，假设客户端最后一次发送的 ACK 包在传输的时候丢失了，由于 TCP 协议的超时重传机制，服务端将重发 FIN 报文，若客户端并没有维持 TIME-WAIT 状态而直接关闭的话，当收到服务端重新发送的 FIN 包时，客户端就会用 RST 包来响应服务端，这将会使得对方认为是有错误发生，然而其实只是正常的关闭连接过程，并没有出现异常情况。

### TIME_WAIT 状态会导致什么问题，怎么解决

我们考虑高并发短连接的业务场景，在高并发短连接的 TCP 服务器上，当服务器处理完请求后主动请求关闭连接，这样服务器上会有大量的连接处于 TIME_WAIT 状态，服务器维护每一个连接需要一个 socket，也就是每个连接会占用一个文件描述符，而文件描述符的使用是有上限的，如果持续高并发，会导致一些正常的 连接失败。

解决方案：修改配置或设置 SO_REUSEADDR 套接字，使得服务器处于 TIME-WAIT 状态下的端口能够快速回收和重用。

### TIME-WAIT 为什么是 2MSL

当客户端发出最后的 ACK 确认报文时，并不能确定服务器端能够收到该段报文。所以客户端在发送完 ACK 确认报文之后，会设置一个时长为 2 MSL 的计时器。MSL（Maximum Segment Lifetime），指一段 TCP 报文在传输过程中的最大生命周期。2 MSL 即是服务器端发出 FIN 报文和客户端发出的 ACK 确认报文所能保持有效的最大时长。

若服务器在 1 MSL 内没有收到客户端发出的 ACK 确认报文，再次向客户端发出 FIN 报文。如果客户端在 2 MSL 内收到了服务器再次发来的 FIN 报文，说明服务器由于一些原因并没有收到客户端发出的 ACK 确认报文。客户端将再次向服务器发出 ACK 确认报文，并重新开始 2 MSL 的计时。

若客户端在 2MSL 内没有再次收到服务器发送的 FIN 报文，则说明服务器正常接收到客户端 ACK 确认报文，客户端可以进入 CLOSE 阶段，即完成四次挥手。

所以客户端要经历 2 MSL 时长的 TIME-WAIT 阶段，为的是**确认服务器能否接收到客户端发出的 ACK 确认报文**。

### 有很多 TIME-WAIT 状态如何解决

服务器可以设置 SO_REUSEADDR 套接字选项来通知内核，如果端口被占用，但 TCP 连接位于 TIME_WAIT 状态时可以重用端口。如果你的服务器程序停止后想立即重启，而新的套接字依旧希望使用同一端口，此时 SO_REUSEADDR 选项就可以避免 TIME-WAIT 状态。

也可以采用长连接的方式减少 TCP 的连接与断开，在长连接的业务中往往不需要考虑 TIME-WAIT 状态，但其实在长连接的业务中并发量一般不会太高。

### 有很多 CLOSE-WAIT 怎么解决

首先检查是不是自己的代码问题（看是否服务端程序忘记关闭连接），如果是，则修改代码。
调整系统参数，包括句柄相关参数和 TCP/IP 的参数，一般一个 CLOSE_WAIT 会维持至少 2 个小时的时间，我们可以通过调整参数来缩短这个时间。

### TCP 和 UDP 的区别

![img](https://farm1.staticflickr.com/792/27194088468_4cb0141fc8_b.jpg)

![这里写图片描述](https://img-blog.csdn.net/20170921150026492?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFyeXdhbmc1Ng==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![image-20210610211444595](C:\Users\yang\AppData\Roaming\Typora\typora-user-images\image-20210610211444595.png)

### TCP 协议中的定时器

TCP中有七种计时器，分别为：

**建立连接定时器：**顾名思义，该定时器是在建立 TCP 连接的时候使用的，在 TCP 三次握手的过程中，发送方发送 SYN 时，会启动一个定时器（默认为 3 秒），若 SYN 包丢失了，那么 3 秒以后会重新发送 SYN 包，直到达到重传次数。

**重传定时器：**该计时器主要用于 TCP 超时重传机制中，当TCP 发送报文段时，就会创建特定报文的重传计时器，并可能出现两种情况：

① 若在计时器截止之前发送方收到了接收方的 ACK 报文，则撤销该计时器；

② 若计时器截止时间内并没有收到接收方的 ACK 报文，则发送方重传报文，并将计时器复位。

**坚持计时器**：我们知道 TCP 通过让接受方指明希望从发送方接收的数据字节数（窗口大小）来进行流量控制，当接收端的接收窗口满时，接收端会告诉发送端此时窗口已满，请停止发送数据。此时发送端和接收端的窗口大小均为0，直到窗口变为非0时，接收端将发送一个 确认 ACK 告诉发送端可以再次发送数据，但是该报文有可能在传输时丢失。若该 ACK 报文丢失，则双方可能会一直等待下去，为了避免这种死锁情况的发生，发送方使用一个坚持定时器来周期性地向接收方发送探测报文段，以查看接收方窗口是否变大。

**延迟应答计时器**：延迟应答也被称为捎带 ACK，这个定时器是在延迟应答的时候使用的，为了提高网络传输的效率，当服务器接收到客户端的数据后，不是立即回 ACK 给客户端，而是等一段时间，这样如果服务端有数据需要发送给客户端的话，就可以把数据和 ACK 一起发送给客户端了。

**保活定时器**：该定时器是在建立 TCP 连接时指定 SO_KEEPLIVE 时才会生效，当发送方和接收方长时间没有进行数据交互时，该定时器可以用于确定对端是否还活着。

**FIN_WAIT_2** 定时器：当主动请求关闭的一方发送 FIN 报文给接收端并且收到其对 FIN 的确认 ACK后进入 FIN_WAIT_2状态。如果这个时候因为网络突然断掉、被动关闭的一端宕机等原因，导致请求方没有收到接收方发来的 FIN，主动关闭的一方会一直等待。该定时器的作用就是为了避免这种情况的发生。当该定时器超时的时候，请求关闭方将不再等待，直接释放连接。

**TIME_WAIT** 定时器：我们知道在 TCP 四次挥手中，发送方在最后一次挥手之后会进入 TIME_WAIT 状态，不直接进入 CLOSE 状态的主要原因是被动关闭方万一在超时时间内没有收到最后一个 ACK，则会重发最后的 FIN，2 MSL（报文段最大生存时间）等待时间保证了重发的 FIN 会被主动关闭的一段收到且重新发送最后一个 ACK 。还有一个原因是在这 2 MSL 的时间段内任何迟到的报文段会被接收方丢弃，从而防止老的 TCP 连接的包在新的 TCP 连接里面出现。

### TCP 是如何保证可靠性的

数据分块：应用数据被分割成 TCP 认为最适合发送的数据块。
序列号和确认应答：TCP 给发送的每一个包进行编号，在传输的过程中，每次接收方收到数据后，都会对传输方进行确认应答，即发送 ACK 报文，这个 ACK 报文当中带有对应的确认序列号，告诉发送方成功接收了哪些数据以及下一次的数据从哪里开始发。除此之外，接收方可以根据序列号对数据包进行排序，把有序数据传送给应用层，并丢弃重复的数据。
校验和： TCP 将保持它首部和数据部分的检验和。这是一个端到端的检验和，目的是检测数据在传输过程中的任何变化。如果收到报文段的检验和有差错，TCP 将丢弃这个报文段并且不确认收到此报文段。
流量控制： TCP 连接的双方都有一个固定大小的缓冲空间，发送方发送的数据量不能超过接收端缓冲区的大小。当接收方来不及处理发送方的数据，会提示发送方降低发送的速率，防止产生丢包。TCP 通过滑动窗口协议来支持流量控制机制。
拥塞控制： 当网络某个节点发生拥塞时，减少数据的发送。
ARQ协议： 也是为了实现可靠传输的，它的基本原理就是每发完一个分组就停止发送，等待对方确认。在收到确认后再发下一个分组。
超时重传： 当 TCP 发出一个报文段后，它启动一个定时器，等待目的端确认收到这个报文段。如果超过某个时间还没有收到确认，将重发这个报文段。

### UDP 为什么是不可靠的？bind 和 connect 对于 UDP 的作用是什么

UDP 只有一个 socket 接收缓冲区，没有 socket 发送缓冲区，即只要有数据就发，不管对方是否可以正确接收。而在对方的 socket 接收缓冲区满了之后，新来的数据报无法进入到 socket 接受缓冲区，此数据报就会被丢弃，因此 UDP 不能保证数据能够到达目的地，此外，UDP 也没有流量控制和重传机制，故UDP的数据传输是不可靠的。

和 TCP 建立连接时采用三次握手不同，UDP 中调用 connect 只是把对端的 IP 和 端口号记录下来，并且 UDP 可多多次调用 connect 来指定一个新的 IP 和端口号，或者断开旧的 IP 和端口号（通过设置 connect 函数的第二个参数）。和普通的 UDP 相比，调用 connect 的 UDP 会提升效率，并且在高并发服务中会增加系统稳定性。

当 UDP 的发送端调用 bind 函数时，就会将这个套接字指定一个端口，若不调用 bind 函数，系统内核会随机分配一个端口给该套接字。当手动绑定时，能够避免内核来执行这一操作，从而在一定程度上提高性能。

### TCP 超时重传的原理

发送方在发送一次数据后就开启一个定时器，在一定时间内如果没有得到发送数据包的 ACK 报文，那么就重新发送数据，在达到一定次数还没有成功的话就放弃重传并发送一个复位信号。其中超时时间的计算是超时的核心，而定时时间的确定往往需要进行适当的权衡，因为当定时时间过长会造成网络利用率不高，定时太短会造成多次重传，使得网络阻塞。在 TCP 连接过程中，会参考当前的网络状况从而找到一个合适的超时时间。

### TCP 的停止等待协议是什么

停止等待协议是为了实现 TCP 可靠传输而提出的一种相对简单的协议，该协议指的是发送方每发完一组数据后，直到收到接收方的确认信号才继续发送下一组数据。

### TCP 最大连接数限制

Client 最大 TCP 连接数
client 在每次发起 TCP 连接请求时，如果自己并不指定端口的话，系统会随机选择一个本地端口（local port），该端口是独占的，不能和其他 TCP 连接共享。TCP 端口的数据类型是 unsigned short，因此本地端口个数最大只有 65536，除了端口 0不能使用外，其他端口在空闲时都可以正常使用，这样可用端口最多有 65535 个。

Server最大 TCP 连接数
server 通常固定在某个本地端口上监听，等待 client 的连接请求。不考虑地址重用（Unix 的 SO_REUSEADDR 选项）的情况下，即使 server 端有多个 IP，本地监听端口也是独占的，因此 server 端 TCP 连接 4 元组中只有客户端的 IP 地址和端口号是可变的，因此最大 TCP 连接为客户端 IP 数 × 客户端 port 数，对 IPV4，在不考虑 IP 地址分类的情况下，最大 TCP 连接数约为 2 的 32 次方（IP 数）× 2 的 16 次方（port 数），也就是 server 端单机最大 TCP 连接数约为 2 的 48 次方。

然而上面给出的是只是理论上的单机最大连接数，在实际环境中，受到明文规定（一些 IP 地址和端口具有特殊含义，没有对外开放）、机器资源、操作系统等的限制，特别是 sever 端，其最大并发 TCP 连接数远不能达到理论上限。对 server 端，通过增加内存、修改最大文件描述符个数等参数，单机最大并发 TCP 连接数超过 10 万 是没问题的。

### TCP 流量控制与拥塞控制

流量控制
所谓流量控制就是让发送方的发送速率不要太快，让接收方来得及接收。如果接收方来不及接收发送方发送的数据，那么就会有分组丢失。在 TCP 中利用可变长的滑动窗口机制可以很方便的在 TCP 连接上实现对发送方的流量控制。主要的方式是接收方返回的 ACK 中会包含自己的接收窗口大小，以控制发送方此次发送的数据量大小（发送窗口大小）。

拥塞控制
在实际的网络通信系统中，除了发送方和接收方外，还有路由器，交换机等复杂的网络传输线路，此时就需要拥塞控制。拥塞控制是作用于网络的，它是防止过多的数据注入到网络中，避免出现网络负载过大的情况。常用的解决方法有：慢开始和拥塞避免、快重传和快恢复。

拥塞控制和流量控制的区别
拥塞控制往往是一种全局的，防止过多的数据注入到网络之中，而TCP连接的端点只要不能收到对方的确认信息，猜想在网络中发生了拥塞，但并不知道发生在何处，因此，流量控制往往指点对点通信量的控制，是端到端的问题。

### 如果接收方滑动窗口满了，发送方会怎么做

基于 TCP 流量控制中的滑动窗口协议，我们知道接收方返回给发送方的 ACK 包中会包含自己的接收窗口大小，若接收窗口已满，此时接收方返回给发送方的接收窗口大小为 0，此时发送方会等待接收方发送的窗口大小直到变为非 0 为止，然而，接收方回应的 ACK 包是存在丢失的可能的，为了防止双方一直等待而出现死锁情况，此时就需要坚持计时器来辅助发送方周期性地向接收方查询，以便发现窗口是否变大【坚持计时器参考问题】，当发现窗口大小变为非零时，发送方便继续发送数据。

### TCP 拥塞控制采用的四种算法

慢开始
当发送方开始发送数据时，由于一开始不知道网络负荷情况，如果立即将大量的数据字节传输到网络中，那么就有可能引起网络拥塞。一个较好的方法是在一开始发送少量的数据先探测一下网络状况，即由小到大的增大发送窗口（拥塞窗口 cwnd）。慢开始的慢指的是初始时令 cwnd为 1，即一开始发送一个报文段。如果收到确认，则 cwnd = 2，之后每收到一个确认报文，就令 cwnd = cwnd* 2。

但是，为了防止拥塞窗口增长过大而引起网络拥塞，另外设置了一个慢开始门限 ssthresh。

① 当 cwnd < ssthresh 时，使用上述的慢开始算法；

② 当 cwnd > ssthresh 时，停止使用慢开始，转而使用拥塞避免算法；

③ 当 cwnd == ssthresh 时，两者均可。

拥塞避免
拥塞控制是为了让拥塞窗口 cwnd 缓慢地增大，即每经过一个往返时间 RTT （往返时间定义为发送方发送数据到收到确认报文所经历的时间）就把发送方的 cwnd 值加 1，通过让 cwnd 线性增长，防止很快就遇到网络拥塞状态。

当网络拥塞发生（超时重传）时，让新的慢开始门限值变为发生拥塞时候的值的一半,并将拥塞窗口置为 1 ,然后再次重复两种算法（慢开始和拥塞避免）,这时一瞬间会将网络中的数据量大量降低。

快重传
快重传算法要求接收方每收到一个失序的报文就立即发送重复确认，而不要等到自己发送数据时才捎带进行确认，假定发送方发送了 Msg 1 ~ Msg 4 这 4 个报文，已知接收方收到了 Msg 1，Msg 3 和 Msg 4 报文，此时因为接收到收到了失序的数据包，按照快重传的约定，接收方应立即向发送方发送 Msg 1 的重复确认。 于是在接收方收到 Msg 4 报文的时候，向发送方发送的仍然是 Msg 1 的重复确认。这样，发送方就收到了 3 次 Msg 1 的重复确认，于是立即重传对方未收到的 Msg 报文。由于发送方尽早重传未被确认的报文段，因此，快重传算法可以提高网络的吞吐量。

快恢复
快恢复算法是和快重传算法配合使用的，该算法主要有以下两个要点：

① 当发送方连续收到三个重复确认，执行乘法减小，慢开始门限 ssthresh 值减半；

② 由于发送方可能认为网络现在没有拥塞，因此与慢开始不同，把 cwnd 值设置为 ssthresh 减半之后的值，然后执行拥塞避免算法，线性增大 cwnd。

![image.png](https://pic.leetcode-cn.com/1618208921-kuMbas-image.png)

### TCP 粘包问题

为什么会发生TCP粘包和拆包?

① 发送方写入的数据大于套接字缓冲区的大小，此时将发生拆包。

② 发送方写入的数据小于套接字缓冲区大小，由于 TCP 默认使用 Nagle 算法，只有当收到一个确认后，才将分组发送给对端，当发送方收集了多个较小的分组，就会一起发送给对端，这将会发生粘包。

③ 进行 MSS （最大报文长度）大小的 TCP 分段，当 TCP 报文的数据部分大于 MSS 的时候将发生拆包。

④ 发送方发送的数据太快，接收方处理数据的速度赶不上发送端的速度，将发生粘包。

常见解决方法

① 在消息的头部添加消息长度字段，服务端获取消息头的时候解析消息长度，然后向后读取相应长度的内容。

② 固定消息数据的长度，服务端每次读取既定长度的内容作为一条完整消息，当消息不够长时，空位补上固定字符。但是该方法会浪费网络资源。

③ 设置消息边界，也可以理解为分隔符，服务端从数据流中按消息边界分离出消息内容，一般使用换行符。

什么时候需要处理粘包问题？

当接收端同时收到多个分组，并且这些分组之间毫无关系时，需要处理粘包；而当多个分组属于同一数据的不同部分时，并不需要处理粘包问题。

### 高并发服务器客户端主动关闭连接和服务端主动关闭连接的区别

服务端主动关闭连接
在高并发场景下，当服务端主动关闭连接时，此时服务器上就会有大量的连接处于 TIME-WAIT 状态。

客户端主动关闭连接
当客户端主动关闭连接时，我们并不需要关心 TIME-WAIT 状态过多造成的问题，但是需要关注服务端保持大量的 CLOSE-WAIT 状态时会产生的问题。

### TCP keepalive

**概念**：在使用TCP长连接（复用已建立TCP连接）的场景下，需要对TCP连接进行保活，避免被网关干掉连接。
在应用层，可以通过定时发送心跳包的方式实现。而Linux已提供的TCP KEEPALIVE，在应用层可不关心心跳包何时发送、发送什么内容，由OS管理：OS会在该TCP连接上定时发送探测包，探测包既起到**连接保活**的作用，也能自动检测连接的有效性，并**自动关闭无效连接**。

**原理：**建立TCP连接时，就有定时器与之绑定，其中的一些定时器就用于处理keepalive过程。当keepalive定时器到0的时候，便会给对端发送一个不包含数据部分的keepalive探测包（probe packet），如果收到了keepalive探测包的回复消息，那就可以断定连接依然是OK的。如果我们没有收到对端keepalive探测包的回复消息，我们便可以断定连接已经不可用，进而采取一些措施。但Keepalive会额外产生一些网络数据包外，这些包将加大网络流量，对路由器和防火墙造成一定的负担。

#### 长连接环境为什么要保活

TCP连接被中间设备断开，客户端和服务端对此是得不到通知的。长连接的环境下，如IM服务，若某连接长时间没有数据交换，可能被丢弃。那一旦有数据需要传递，且此时连接已经被中介设备断开，应用程序没有及时感知的话，那么就会导致在一个无效的数据链路层面发送业务数据，结果就是发送失败。

### HTTP keep-alive

在 HTTP 1.0 时期，每个 TCP 连接只会被一个 HTTP Transaction（请求加响应）使用，请求时建立，请求完成释放连接。当网页内容越来越复杂，包含大量图片、CSS 等资源之后，这种模式效率就显得太低了。所以，在 HTTP 1.1 中，引入了 HTTP persistent connection 的概念，也称为 HTTP keep-alive，目的是复用TCP连接，在一个TCP连接上进行多次的HTTP请求从而提高性能。

HTTP1.0中默认是关闭的，需要在HTTP头加入"Connection: Keep-Alive"，才能启用Keep-Alive；HTTP1.1中默认启用Keep-Alive，加入"Connection: close "，才关闭。

## 网络层

### IP 协议的定义和作用

IP 协议（Internet Protocol）又称互联网协议，是支持网间互联的数据包协议。该协议工作在网络层，主要目的就是为了提高网络的可扩展性，和传输层 TCP 相比，IP 协议提供一种无连接/不可靠、尽力而为的数据包传输服务，其与TCP协议（传输控制协议）一起构成了TCP/IP 协议族的核心。IP 协议主要有以下几个作用：

寻址和路由：在IP 数据包中会携带源 IP 地址和目的 IP 地址来标识该数据包的源主机和目的主机。IP 数据报在传输过程中，每个中间节点（IP 网关、路由器）只根据网络地址进行转发，如果中间节点是路由器，则路由器会根据路由表选择合适的路径。IP 协议根据路由选择协议提供的路由信息对 IP 数据报进行转发，直至抵达目的主机。
分段与重组：IP 数据包在传输过程中可能会经过不同的网络，在不同的网络中数据包的最大长度限制是不同的，IP 协议通过给每个 IP 数据包分配一个标识符以及分段与组装的相关信息，使得数据包在不同的网络中能够传输，被分段后的 IP 数据报可以独立地在网络中进行转发，在到达目的主机后由目的主机完成重组工作，恢复出原来的 IP 数据包。

### 域名和 IP 的关系，一个 IP 可以对应多个域名吗

IP 在同一个网络中是唯一的，用来标识每一个网络上的设备，其相当于一个人的身份证号；域名在同一个网络中也是唯一的，就像一个人的名字，绰号。假如你有多个不同的绰号，你的朋友可以用其中任何一个绰号叫你，但你的身份证号码却是唯一的。由此我们可以看出一个域名只能对应一个 IP 地址，是一对一的关系；而一个 IP 却可以对应多个域名，是一对多的关系。

### IPV4 地址不够如何解决

DHCP：动态主机配置协议。动态分配 IP 地址，只给接入网络的设备分配IP地址，因此同一个 MAC 地址的设备，每次接入互联网时，得到的IP地址不一定是相同的，该协议使得空闲的 IP 地址可以得到充分利用。
CIDR：无类别域间路由。CIDR 消除了传统的 A 类、B 类、C 类地址以及划分子网的概念，因而更加有效的分配 IPv4 的地址空间，但无法从根本上解决地址耗尽问题。
NAT：网络地址转换协议。我们知道属于不同局域网的主机可以使用相同的 IP 地址，从而一定程度上缓解了 IP 资源枯竭的问题。然而主机在局域网中使用的 IP 地址是不能在公网中使用的，当局域网主机想要与公网进行通信时， NAT 方法可以将该主机 IP 地址转换成全球 IP 地址。该协议能够有效解决 IP 地址不足的问题。
IPv6 ：作为接替 IPv4 的下一代互联网协议，其可以实现 2 的 128 次方个地址，而这个数量级，即使是给地球上每一颗沙子都分配一个IP地址，该协议能够从根本上解决 IPv4 地址不够用的问题。

### 路由器的分组转发流程

① 从 IP 数据包中提取出目的主机的 IP 地址，找到其所在的网络；

② 判断目的 IP 地址所在的网络是否与本路由器直接相连，如果是，则不需要经过其它路由器直接交付，否则执行 ③；

③ 检查路由表中是否有目的 IP 地址的特定主机路由。如果有，则按照路由表传送到下一跳路由器中，否则执行 ④；

④ 逐条检查路由表，若找到匹配路由，则按照路由表转发到下一跳路由器中，否则执行步骤 ⑤；

⑤ 若路由表中设置有默认路由，则按照默认路由转发到默认路由器中，否则执行步骤 ⑥；

⑥ 无法找到合适路由，向源主机报错。

### 路由器和交换机的区别

交换机：交换机用于局域网，利用主机的物理地址（MAC 地址）确定数据转发的目的地址，它工作与数据链路层。
路由器：路由器通过数据包中的目的 IP 地址识别不同的网络从而确定数据转发的目的地址，网络号是唯一的。路由器根据路由选择协议和路由表信息从而确定数据的转发路径，直到到达目的网络，它工作于网络层。

### ICMP 协议概念/作用

ICMP（Internet Control Message Protocol）是因特网控制报文协议，主要是实现 IP 协议中未实现的部分功能，是一种网络层协议。该协议并不传输数据，只传输控制信息来辅助网络层通信。其主要的功能是验证网络是否畅通（确认接收方是否成功接收到 IP 数据包）以及辅助 IP 协议实现可靠传输（若发生 IP 丢包，ICMP 会通知发送方 IP 数据包被丢弃的原因，之后发送方会进行相应的处理）。

### ICMP 的应用

Ping
Ping（Packet Internet Groper），即因特网包探测器，是一种工作在网络层的服务命令，主要用于测试网络连接量。本地主机通过向目的主机发送 ICMP Echo 请求报文，目的主机收到之后会发送 Echo 响应报文，Ping 会根据时间和成功响应的次数估算出数据包往返时间以及丢包率从而推断网络是否通常、运行是否正常等。

### 两台电脑连起来后 ping 不通，你觉得可能存在哪些问题？

首先看网络是否连接正常，检查网卡驱动是否正确安装。
局域网设置问题，检查 IP 地址是否设置正确。
看是否被防火墙阻拦（有些设置中防火墙会对 ICMP 报文进行过滤），如果是的话，尝试关闭防火墙 。
看是否被第三方软件拦截。
两台设备间的网络延迟是否过大（例如路由设置不合理），导致 ICMP 报文无法在规定的时间内收到。

### ARP 地址解析协议的原理和地址解析过程

ARP（Address Resolution Protocol）是地址解析协议的缩写，该协议提供根据 IP 地址获取物理地址的功能，它工作在第二层，是一个数据链路层协议，其在本层和物理层进行联系，同时向上层提供服务。当通过以太网发送 IP 数据包时，需要先封装 32 位的 IP 地址和 48位 MAC 地址。在局域网中两台主机进行通信时需要依靠各自的物理地址进行标识，但由于发送方只知道目标 IP 地址，不知道其 MAC 地址，因此需要使用地址解析协议。 ARP 协议的解析过程如下：

① 首先，每个主机都会在自己的 ARP 缓冲区中建立一个 ARP 列表，以表示 IP 地址和 MAC 地址之间的对应关系；

② 当源主机要发送数据时，首先检查 ARP 列表中是否有 IP 地址对应的目的主机 MAC 地址，如果存在，则可以直接发送数据，否则就向同一子网的所有主机发送 ARP 数据包。该数据包包括的内容有源主机的 IP 地址和 MAC 地址，以及目的主机的 IP 地址。

③ 当本网络中的所有主机收到该 ARP 数据包时，首先检查数据包中的 目的 主机IP 地址是否是自己的 IP 地址，如果不是，则忽略该数据包，如果是，则首先从数据包中取出源主机的 IP 和 MAC 地址写入到 ARP 列表中，如果已经存在，则覆盖，然后将自己的 MAC 地址写入 ARP 响应包中，告诉源主机自己是它想要找的 MAC 地址。

④ 源主机收到 ARP 响应包后。将目的主机的 IP 和 MAC 地址写入 ARP 列表，并利用此信息发送数据。如果源主机一直没有收到 ARP 响应数据包，表示 ARP 查询失败。

### TTL 是什么？有什么作用

TTL 是指生存时间，简单来说，它表示了数据包在网络中的时间。每经过一个路由器后 TTL 就减一，这样 TTL 最终会减为 0 ，当 TTL 为 0 时，则将数据包丢弃。通过设置 TTL 可以避免这两个路由器之间形成环导致数据包在环路上死转的情况，由于有了 TTL ，当 TTL 为 0 时，数据包就会被抛弃。

### 运输层协议和网络层协议的区别

网络层协议负责提供主机间的逻辑通信；运输层协议负责提供进程间的逻辑通信。

### 网络地址转换 NAT

NAT（Network Address Translation），即网络地址转换，它是一种把内部私有网络地址翻译成公有网络 IP 地址的技术。该技术不仅能解决 IP 地址不足的问题，而且还能隐藏和保护网络内部主机，从而避免来自外部网络的攻击。

NAT 的实现方式主要有三种：

静态转换：内部私有 IP 地址和公有 IP 地址是一对一的关系，并且不会发生改变。通过静态转换，可以实现外部网络对内部网络特定设备的访问，这种方式原理简单，但当某一共有 IP 地址被占用时，跟这个 IP 绑定的内部主机将无法访问 Internet。
动态转换：采用动态转换的方式时，私有 IP 地址每次转化成的公有 IP 地址是不唯一的。当私有 IP 地址被授权访问 Internet 时会被随机转换成一个合法的公有 IP 地址。当 ISP 通过的合法 IP 地址数量略少于网络内部计算机数量时，可以采用这种方式。
端口多路复用：该方式将外出数据包的源端口进行端口转换，通过端口多路复用的方式，实现内部网络所有主机共享一个合法的外部 IP 地址进行 Internet 访问，从而最大限度地节约 IP 地址资源。同时，该方案可以隐藏内部网络中的主机，从而有效避免来自 Internet 的攻击。

## 数据链路层

### MAC 地址和 IP 地址分别有什么作用

MAC 地址是数据链路层和物理层使用的地址，是写在网卡上的物理地址。MAC 地址用来定义网络设备的位置。
IP 地址是网络层和以上各层使用的地址，是一种逻辑地址。IP 地址用来区别网络上的计算机。

### 为什么有了 MAC 地址还需要 IP 地址

如果我们只使用 MAC 地址进行寻址的话，我们需要路由器记住每个 MAC 地址属于哪一个子网，不然每一次路由器收到数据包时都要满世界寻找目的 MAC 地址。而我们知道 MAC 地址的长度为 48 位，也就是说最多总共有 2 的 48 次方个 MAC 地址，这就意味着每个路由器需要 256 T 的内存，这显然是不现实的。

和 MAC 地址不同，**IP 地址是和地域相关的**，在一个子网中的设备，我们给其分配的 IP 地址前缀都是一样的，这样路由器就能根据 IP 地址的前缀知道这个设备属于哪个子网，剩下的寻址就交给子网内部实现，从而大大减少了路由器所需要的内存。

### 为什么有了 IP 地址还需要 MAC 地址

只有当设备连入网络时，才能根据他进入了哪个子网来为其分配 IP 地址，在设备还没有 IP 地址的时候或者在分配 IP 地址的过程中，我们需要 MAC 地址来区分不同的设备。

### 私网地址和公网地址之间进行转换：同一个局域网内的两个私网地址，经过转换之后外面看到的一样吗

当采用静态或者动态转换时，由于一个私网 IP 地址对应一个公网地址，因此经过转换之后的公网 IP 地址是不同的；而采用端口复用方式的话，在一个子网中的所有地址都采用一个公网地址，但是使用的端口是不同的。

### PPP 协议

互联网用户通常需要连接到某个 ISP 之后才能接入到互联网，PPP（点对点）协议是用户计算机和 ISP 进行通信时所使用的数据链路层协议。点对点协议为点对点连接上传输多协议数据包提供了一个标准方法。该协议设计的目的主要是用来通过拨号或专线方式建立点对点连接发送数据，使其成为各种主机、网桥和路由器之间简单连接的一种解决方案。

PPP 协议具有以下特点：

PPP 协议具有动态分配 IP 地址的能力，其允许在连接时刻协商 IP 地址。
PPP 支持多种网络协议，例如 TCP/IP、NETBEUI 等。
PPP 具有差错检测能力，但不具备纠错能力，所以 PPP 是不可靠传输协议。
无重传的机制，网络开销小，速度快。
PPP 具有身份验证的功能。

### 为什么 PPP 协议不使用序号和确认机制

IETF 在设计因特网体系结构时把其中最复杂的部分放在 TCP 协议中，而网际协议 IP 则相对比较简单，它提供的是不可靠的数据包服务，在这种情况下，数据链路层没有必要提供比 IP 协议更多的功能。若使用能够实现可靠传输的数据链路层协议，则开销就要增大，这在数据链路层出现差错概率不大时是得不偿失的。
即使数据链路层实现了可靠传输，但其也不能保证网络层的传输也是可靠的，当数据帧在路由器中从数据链路层上升到网络层后，仍有可能因为网络层拥塞而被丢弃。
PPP 协议在帧格式中有帧检验序列，对每一个收到的帧，PPP 都会进行差错检测，若发现差错，则丢弃该帧。

## 物理层

### 物理层主要做什么事情

作为 OSI 参考模型最低的一层，物理层是整个开放系统的基础，该层利用传输介质为通信的两端建立、管理和释放物理连接，实现比特流的透明传输。物理层考虑的是怎样才能在连接各种计算机的传输媒体上传输数据比特流，其尽可能地屏蔽掉不同种类传输媒体和通信手段的差异，使物理层上面的数据链路层感觉不到这些差异，这样就可以使数据链路层只考虑完成本层的协议和服务，而不必考虑网络的具体传输媒体和通信手段是什么。

### 主机之间的通信方式

单工通信：也叫单向通信，发送方和接收方是固定的，消息只能单向传输。例如采集气象数据、家庭电费，网费等数据收集系统，或者打印机等应用主要采用单工通信。
半双工通信：也叫双向交替通信，通信双方都可以发送消息，但同一时刻同一信道只允许单方向发送数据。例如传统的对讲机使用的就是半双工通信。
全双工通信：也叫双向同时通信，全双工通信允许通信双方同时在两个方向是传输，其要求通信双方都具有独立的发送和接收数据的能力。例如平时我们打电话，自己说话的同时也能听到对面的声音

## 计算机网络中的安全

### 安全攻击有哪些

网络安全攻击主要分为被动攻击和主动攻击两类：

被动攻击：攻击者窃听和监听数据传输，从而获取到传输的数据信息，被动攻击主要有两种形式：消息内容泄露攻击和流量分析攻击。由于攻击者并没有修改数据，使得这种攻击类型是很难被检测到的。
主动攻击：攻击者修改传输的数据流或者故意添加错误的数据流，例如假冒用户身份从而得到一些权限，进行权限攻击，除此之外，还有重放、改写和拒绝服务等主动攻击的方式。

### ARP 攻击

在 ARP 的解析过程中，局域网上的任何一台主机如果接收到一个 ARP 应答报文，并不会去检测这个报文的真实性，而是直接记入自己的 ARP 缓存表中。并且这个 ARP 表是可以被更改的，当表中的某一列长时间不适使用，就会被删除。ARP 攻击就是利用了这一点，攻击者疯狂发送 ARP 报文，其源 MAC 地址为攻击者的 MAC 地址，而源 IP 地址为被攻击者的 IP 地址。通过不断发送这些伪造的 ARP 报文，让网络内部的所有主机和网关的 ARP 表中被攻击者的 IP 地址所对应的 MAC 地址为攻击者的 MAC 地址。这样所有发送给被攻击者的信息都会发送到攻击者的主机上，从而产生 ARP 欺骗。通常可以把 ARP 欺骗分为以下几种：

洪泛攻击
攻击者恶意向局域网中的网关、路由器和交换机等发送大量 ARP 报文，设备的 CPU 忙于处理 ARP 协议，而导致难以响应正常的服务请求。其表现通常为：网络中断或者网速很慢。

欺骗主机
这种攻击方式也叫仿冒网关攻击。攻击者通过 ARP 欺骗使得网络内部被攻击主机发送给网关的信息实际上都发送给了攻击者，主机更新的 ARP 表中对应的 MAC 地址为攻击者的 MAC。当用户主机向网关发送重要信息使，该攻击方式使得用户的数据存在被窃取的风险。

欺骗网关
该攻击方式和欺骗主机的攻击方式类似，不过这种攻击的欺骗对象是局域网的网关，当局域网中的主机向网关发送数据时，网关会把数据发送给攻击者，这样攻击者就会源源不断地获得局域网中用户的信息。该攻击方式同样会造成用户数据外泄。

中间人攻击
攻击者同时欺骗网关和主机，局域网的网关和主机发送的数据最后都会到达攻击者这边。这样，网关和用户的数据就会泄露。

IP 地址冲突
攻击者对局域网中的主机进行扫描，然后根据物理主机的 MAC 地址进行攻击，导致局域网内的主机产生 IP 冲突，使得用户的网络无法正常使用。

### 对称加密和非对称的区别，非对称加密有哪些

加密和解密的过程不同：对称加密和解密过程使用同一个密钥；非对称加密中加密和解密采用公钥和私钥两个密钥，一般使用公钥进行加密，使用私钥进行解密。
加密和解密的速度不同：对称加密和解密速度较快，当数据量比较大时适合使用；非对称加密和解密时间较长，速度相对较慢，适合少量数据传输的场景。
传输的安全性不同：采用对称加密方式进行通信时，收发双方在数据传送前需要协定好密钥，而这个密钥还有可能被第三方窃听到的，一旦密钥泄漏，之后的通信就完全暴漏给攻击者了；非对称加密采用公钥加密和私钥解密的方式，其中私钥是基于不同的算法生成的随机数，公钥可以通过私钥通过一定的算法推导得出，并且私钥到公钥的推导过程是不可逆的，也就是说公钥无法反推导出私钥，即使攻击者窃听到传输的公钥，也无法正确解出数据，所以安全性较高。
常见的非对称加密算法主要有：RSA、Elgamal、背包算法、Rabin、D-H 算法等等

### DDoS 有哪些，如何防范

DDoS 为分布式拒绝服务攻击，是指处于不同位置的多个攻击者同时向一个或数个目标发动攻击，或者一个攻击者控制了不同位置上的多台机器并利用这些机器对受害者同时实施攻击。和单一的 DoS 攻击相比，DDoS 是借助数百台或者数千台已被入侵并添加了攻击进程的主机一起发起网络攻击。

DDoS 攻击主要有两种形式：流量攻击和资源耗尽攻击。前者主要针对网络带宽，攻击者和已受害主机同时发起大量攻击导致网络带宽被阻塞，从而淹没合法的网络数据包；后者主要针对服务器进行攻击，大量的攻击包会使得服务器资源耗尽或者 CPU 被内核应用程序占满从而无法提供网络服务。

常见的 DDos 攻击主要有：TCP 洪水攻击（SYN Flood）、放射性攻击（DrDos）、CC 攻击（HTTP Flood）等。

针对 DDoS 中的流量攻击，最直接的方法是增加带宽，理论上只要带宽大于攻击流量就可以了，但是这种方法成本非常高。在有充足网络带宽的前提下，我们应尽量提升路由器、网卡、交换机等硬件设施的配置。

针对资源耗尽攻击，我们可以升级主机服务器硬件，在网络带宽得到保证的前提下，使得服务器能有效对抗海量的 SYN 攻击包。我们也可以安装专业的抗 DDoS 防火墙，从而对抗 SYN Flood等流量型攻击。此外，负载均衡，CDN 等技术都能够有效对抗 DDoS 攻击
