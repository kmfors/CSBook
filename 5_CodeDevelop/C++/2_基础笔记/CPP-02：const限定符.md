# CPP-02: const 与引用

## const

关键字 const 对变量的类型加以限定，保证变量的值不会被改变。无论初始化给一个 const 变量，还是将 const 变量拷贝给其他变量，都不会有影响的。

```c++
const int var1 = 10; 
int const var2 = 20;

const int var3;      // compile error: not initialized
const int var4{};    // ok: value initialized
const int var5{ 5 }; // ok: list initialized
```

当以编译时初始化的方式定义一个 const 对象时，编译器将在编译过程中把用到该变量的地方都替换成对应的值。也就是说，编译器会找到代码中所有用到该 const 对象的地方，用值去替换它。因此为了编译期间的上述替换，且避免对同一变量的重复定义，**默认情况状态下，const 对象仅在文件内有效**。

> [!IMPORTANT]
>
> 1. const 对象创建时必须初始化，且不能被改变。
> 2. 默认情况状态下，const 对象仅在本文件内有效，不能被其他文件访问；因此如果多个文件出现同名 const 变量，等同于在不同文件中分别定义了独立的变量。
> 3. 如果想要在多个文件之间共享 const 对象，那在该对象的声明或定义之前必须添加 extern 关键字。



## const 与指针

常量指针：保护其所指对象的值不发生改变，从左向右念修饰符。当 const 位于 `*` 左边时，称为常量指针：`const *`

```c++
double num = 3.14;
const double *ptr = &num; // const 保护 *ptr ,不能改变指针的指向内容，但可以改变指向方向
```


指针常量：保护其所指的对象不发生改变，从左向右念修饰符。当 const 位于 `*` 右边时，称为指针常量：`* const`

```c++
double num = 3.14;
double * const ptr = &num; // const 保护 pstr ,不能改变指针的指向方向，但可以改变指向内容
```


- `顶层const`：指针本身是个常量。（const 位于最右侧）
- `底层const`：指针指向的对象是个常量。拷贝时严格要求相同的底层 const 资格。

> [!NOTE]
>
> 1. **顶层 const** 表示 **指针本身** 是个常量，**底层 const** 表示 **指针所指的对象** 是一个常量。
> 2. 顶层 const 对任何数据类型通用，**底层 const** 只用于引用和指针。
> 3. 顶层 const 的指针表示该指针是 const 对象，因此必须初始化。底层 const 的指针则不用。
> 4. 实际上只有指针类型既可以是顶层 const 也可以是底层 const，因为引用实际上只能是底层 const，常量引用即为底层 const，不存在顶层 const 的引用。
> 5. 对于指针和引用而言，顶层 const 在右边，底层 const 在左边。对于其他类型，全都是顶层 const。
> 6. 执行对象的拷贝操作时，不能将底层 const 拷贝给非常量，反之可以，非常量将会转化为常量。
> 



## 引用（&）

引用就是变量的一个别名，对引用的修改操作就是对变量本身的修改，引用与变量指向的地址是相同的。

引用在定义时，必须要进行初始化，不能单独存在，必须要绑定上一个实体；并且引用一经绑定后，就不能改变绑定对象。

引用的底层实现就是一个指针（* const)，即* *指针常量**（不能改变自身的值，但可以改变指向的值）。

**C++中引入引用的目的，就是为了减少指针的使用。**

引用的形式如下：

```c++
int number = 10;
int &ref = number;
```

### 引用作为函数参数

一般的参数传递分为：值传递、地址传递、引用传递。

- 值传递：即产生一个副本数据、其本身具有拷贝开销。
- 地址传递：直观性不够友好。
- 引用传递：操作引用与操作变量本身的效果是一样的，直观性更好一些。

```c++
void swap(int &x,int &y);

//int &x = a; int &y = b;
```

### 引用作为函数的返回类型

函数的返回类型如果是引用的话，能够减少拷贝操作。因为函数的返回实则是将局部变量的值拷贝给使用者，也是存在拷贝开销的。

但注意的是正常函数内的局部变量是不能返回它的引用的，引用函数调用结束后，其声明周期也已经结束了。因此 **返回引用的前提：实体的生命周期一定要大于函数的生命周期**

```c++
int &getIndex(){
......
}
```


问题：如果在函数内申请堆空间，返回申请内存对象地址呢？

```c++
// 不建议这么做，因为会发生内存泄漏
int &getHeapData(){
	int *pInt = new int(10);
	return *pInt;
}

// 解决办法
int &ref = getHeapData();
delete &ref;//这样就可以解决堆空间返回引用时引起的内存泄露

```

所以正因为有这样的危险，在平时的代码编写中，不要写这样的代码，不然就需要别人对你写的代码进行内存回收的处理。

如果使用堆空间返回引用也不是不可以，如果能实现自动内存回收的话，这样的写法也是可以的。**所以在函数里申请堆空间时，推荐使用智能指针，完美解决内存泄漏！**

## const 引用

1、非常量引用无法绑定到指向常量引用（初始化时）。同时引用的类型必须与其所引用对象的类型一致。

```c++
const int var1 = 2023;

const int &var2 = var1;	// 正确
int &var3 = var1;		// 报错：var3是非常量引用
```


2、在初始化常量引用时允许用任意表达式作为初始值，只要该表达式的结果能转换成引用的类型即可。与上面相反的是，常量引用可以绑定到非常量或非常量引用（初始化时）。

```c++
int var = 42;
const int &var1 = var;	// 绑定指向非常量类型
const int &var2 = var1;	// 绑定常量引用类型
const int &var3 = 2023;	// 正确：表达式结果可以转换成对应类型，可以绑定

int &var4 = var;
const int &var5 = var4;	// 常量引用可以绑定到非常量引用，但不允许修改
```

> [!NOTE]
>
> - 临时量对象：当编译器需要一个空间来暂存表达式的求值结果时，临时创建的一个未命名的对象。
>
> - 对临时量的引用是非法行为！



## 常量表达式

常量表达式：是指值不会改变，且在编译过程中就能得到计算结果的表达式。注意：字面值是常量表达式，并且用常量表达式初始化的 const 对象也是常量表达式。

`C++11` 标准规定，允许将变量声明为 `constexpr` 类型，由编译器来验证变量的值是否是一个常量的表达式。

- 由 constexpr 声明的变量一定是一个常量，且 **必须用常量表达式初始化**。
- 如果认定一个变量是常量表达式，就把它声明为 constexpr 类型
- 不能用普通函数初始化 constexpr 变量，但可以用 constexpr 函数初始化 constexpr 变量。

```c++
constexpr int num1 = 20;		// 20是常量表达式
constexpr int num2 = num + 1;	// 右侧是常量表达式
constexpr int num3 = size();	// 只有当 size 是一个 constexpr 函数，才算正确
```



## 编译时常量

编译时常量是指在编译时必须知道其值的常量。这包括：

- 字面值：例如，5，2023
- constexpr 变量，也称常量表达式
- 常量整型变量：例如，`const int x {5};` 
- 非类型模板参数
- 枚举值

### 字面值类型

常量表达式的值需要在编译时就得到计算，因此对声明 constexpr 是用到的类型必须有所限制。对于值显而易见、容易得到的就称之为 **字面值类型** 。

- 算术类型、引用和指针都属于字面值类型。（待补充）
- 一个 constexpr 指针的初始值必须是 nullptr 或 0 或存储于某个固定地址中的对象。
- 函数体之外的对象和静态变量的地址都是固定不变的。

> [!CAUTION]
>
> 注意区分 constexpr 和 const ，constexpr 都是顶层 const，仅对指针本身有效。
>
> ```c++
> const int *p = nullptr;     // p 是一个指向整型常量的指针
> constexpr int *q = nullptr; // q 是一个指向整数的常量指针 
> ```
>
> constexpr 限定了变量是编译器常量，即变量的值在编译器就可以得到。
>
> const 则并未区分是编译器常量还是运行期常量。即 const 变量可以在运行期间初始化，只是初始化后就不能再改变了。constexpr 变量是真正的常量，而 const 现在一般只用来表示只读。





### constexpr 函数

定义一个 constexpr 函数需要遵循几项约定：

- 函数的返回值、所有形参的类型都是字面值类型。
- 函数体中必须有且只有一条 return 语句。

初始化执行时，编译器把对 constexpr 函数的调用替换成其结果值。为了能在编译过程中随时展开，constexpr 函数被隐式地指定为内联函数。

> [!WARNING]
>
> 在 C++11 中，`constexpr` 函数的要求确实比 C++14 及以后的版本更为严格。对于 C++11 中的 `constexpr` 函数，有以下几点要求：
>
> 1. **函数体中只能有一条语句**：这条语句必须是返回语句，因为 `constexpr` 函数的目的是在编译时就能计算出结果，因此函数体中不允许有其他类型的语句（如赋值、递增、递减等）。
>
> 2. **返回类型**：返回类型必须是一个基本的标量类型或枚举类型。基本的标量类型包括整数类型、浮点类型等。
>
> 3. **参数类型**：尽管形参不需要是 `constexpr`，但传递给 `constexpr` 函数的参数类型必须属于字面量类型，这意味着它们在编译时就能确定其值。
>
> 4. **顶层 constexpr**：如果一个对象被声明为 `constexpr`，那么它在编译时就必须被初始化，且其初始值必须在编译时就能计算出来。
>
> 从 C++14 开始，对 `constexpr` 函数的限制有所放宽，允许函数体中有多个语句，只要这些语句都能够在编译时执行。例如，可以包含局部变量的声明和初始化、if 语句、lambda 表达式等，只要这些操作不违反编译时常量性的要求。
>



```c++
#include <iostream>

constexpr double compute(double radius) { // now a constexpr function

    constexpr double pi{ 3.14159265359 };
    return 2.0 * pi * radius;
}

int main() {
    constexpr double cLength{ compute(3.0) }; // now compiles
    // 编译时求值等价于
    // constexpr double circumference { 18.8496 };

    std::cout << "Our circle has cLength " << cLength << std::endl;

    return 0;
}
```

常量表达式必须在编译时求值。如果所需的常量表达式包含一个 constexpr 函数调用，则该 constexpr 函数调用必须在编译时求值。在本例中，compute 函数中的实参是常量表达式，但形参类型却不是

在编译时进行求值，还必须满足以下两个条件：

1. 对 constexpr 函数的调用必须有编译时已知的参数（例如常量表达式）。constexpr 函数中的所有语句和表达式都必须可在编译时求值。
2. 当 constexpr（或 consteval）函数在编译时求值时，它调用的任何其他函数也必须在编译时求值（否则初始函数将无法在编译时返回结果）。

### constexpr 函数也可以在运行时求值?

constexpr 函数也可以在运行时求值，在这种情况下，它们将返回一个非 constexpr 结果。例如:

```c++
#include <iostream>

constexpr int greater(int x, int y) {
    return (x > y ? x : y);
}

int main() {
    int x{ 5 }; // not constexpr
    int y{ 6 }; // not constexpr

    std::cout << greater(x, y) << " is greater!\n"; // will be evaluated at runtime

    return 0;
}
```

在本例中，由于参数 x 和 y 不是常量表达式，函数无法在编译时解析。不过，该函数在运行时仍能解析，返回的预期值是 非常量表达式的 int。

> [!NOTE]
>
> 1. 当 constexpr 函数在运行时进行求值时，其求值方式与普通（非 constexpr）函数无异。即 constexpr 在这种情况下没有任何作用。
> 2. 总结第 1 点就是，constexpr 函数不一定返回常量表达式。
> 3. 在编译时或运行时对带有 constexpr 返回类型的函数进行评估，这种行为是被允许的，因为要让单个函数同时满足这两种情况的需要。否则不仅重复代码，还需要为两个函数使用不同的名称。

### 等待更新中

对于常量表达式中调用 constexpr 函数调用不是必要的？

你可能以为 Constexpr 函数会尽可能在编译时进行求值，但不幸的是，情况并非如此。

在第 5.4 课 -- 常量表达式和编译时优化中，我们注意到在不需要常量表达式的上下文中，编译器可以选择在编译时或运行时评估常量表达式。因此，作为非必要的常量表达式一部分的任何 constexpr 函数调用都可以在编译时或运行时进行评估。

```c++
#include <iostream>

constexpr int getValue(int x)
{
    return x;
}

int main()
{
    int x { getValue(5) }; // may evaluate at runtime or compile-time

    return 0;
}
```

只有在需要常量表达式时，才保证对 constexpr 函数进行编译时评估。


诊断 constexpr 函数？

在编译时实际求值之前，编译器无需确定 constexpr 函数在编译时是否可求值。要编写一个在运行时编译成功，但在编译时评估时却无法编译的 constexpr 函数是相当容易的。

```c++
#include <iostream>

int getValue(int x)
{
    return x;
}

// This function can be evaluated at runtime
// When evaluated at compile-time, the function will produce a compilation error
// because the call to getValue(x) cannot be resolved at compile-time
constexpr int foo(int x)
{
    return getValue(x); // call to non-constexpr function here
}

int main()
{
    int x { foo(5) };           // okay: will evaluate at runtime
    constexpr int y { foo(5) }; // compile error: foo(5) can't evaluate at compile-time

    return 0;
}
```

在上例中，当 foo(5) 用作非 constexpr 变量 x 的初始化器时，它将在运行时求值。然而，当 foo(5) 用作 constexpr 变量 y 的初始化器时，它必须在编译时进行求值。此时，编译器会判断对 foo(5) 的调用不能在编译时求值，因为 getValue() 不是 constexpr 函数。

因此，在编写 constexpr 函数时，一定要明确测试该函数在编译时的求值是否可编译（在需要常量表达式的上下文中调用该函数，例如在 constexpr 变量的初始化中）。

所有 constexpr 函数都应可在编译时求值，因为在需要常量表达式的上下文中需要这样做。一定要在需要常量表达式的上下文中测试您的 constexpr 函数，因为 constexpr 函数在运行时求值可能有效，但在编译时求值可能失败。