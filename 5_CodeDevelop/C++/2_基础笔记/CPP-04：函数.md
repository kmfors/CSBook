# CPP-04 : 函数

## 1.1 函数重载

函数重载（overload）：在同一个作用域中，函数的名字相同，但是函数的参数列表不一样（参数的类型、参数的个数、参数顺序）。

```c++
int add(int a){
    return a;
}
int add(int a, int b){
    return a + b;
}
int add(int a, int b, int c){
    return a + b + c;
}
```

> [!NOTE]
>
> C 不支持函数重载，C++支持。C++支持的原因是因为支持对函数名与函数参数的改编（包括参数的顺序、参数的个数、参数的类型）。

linux 的 `nm` 命令：能够查看二进制文件的符号，下面为不同参数列表的 add 函数，因此 C++支持函数重载。

```c++
// c++编译
00000000000011c9 T _Z3addi
00000000000011d9 T _Z3addii
00000000000011f1 T _Z3addiii

// c编译
0000000000000000 T add
```


## 1.2 默认参数

该函数定义时，某形参默认赋某值，即默认参数。若在一个函数里有 n 个参数，实参传入 a 个参数，则 n-a 个未传参数将使用默认参数默认值。

```c++
int add(int x,int y = 10,int z = 10);
```

注意：默认参数必须要从右往左设置参数默认值（连续的赋值，中间不可空着跳过）。

因为对于 C/C++而言，函数的参数入栈顺序就是从右向左。默认参数的使用是为了减少代码的书写，等价于多个不同的函数重载。



## 1.3 内敛函数

函数的调用是有开销的：需要在入栈出栈时，利用栈进行中转。但它的优势在于如果有 bug 能够及时在编译阶段发现其 bug。

带参数的宏定义即宏函数：宏定义发生的时机在预处理的阶段，进行字符串的替换，如果有 bug 需要在运行的时候才可以发现，但宏函数比函数调用效率高。

内联函数吸收函数调用与宏函数的优质特性，以宏的形式嵌入函数体。

```c++
inline int add(int x,int y){
    return x+y;
}
-------------------------------------------------
inline
int add(int x,int y);
#include"add.cc"
```

当函数调用的时候，可以用函数体替代函数的调用。

注意：内联函数不能分成头文件与实现文件的形式（不能将声明与实现分开），否则就会报错，在链接的时候链接不到。

如果一定要将头文件与实现文件分开写，可以在头文件中包含实现文件（ `#include"add.cc"` )

在头文件中 `#include "xxx.cc"` 可以将实现文件的名字改为 `xxx.tcc` 

> [!NOTE]
>
> 现代的编译器可以自动识别一个函数是不是内联函数，已经弱化了 inline 的使用效果。
>
> 内联函数一般不建议使用 while for 等这种复杂语句，也不建议函数的嵌套使用。
>
> 内联函数是以代码复制为代价，仅仅省去了函数调用的开销，从而提高函数的执行效率。如果执行函数体内代码的时间相比于函数调用的开销较大，那么效率的收获会很小。另一方面，每一处内联函数的调用都要复制代码，将会使得程序的总代码量增大，消耗更多内存空间。

