# 可变参数的使用

## 可变参数的形式

需求：希望函数带有可变数量的参数，而不是预定义数量的参数

使用方式：

```c
int function(int arg1, ...);
```

其中，省略号 `...` 表示可变参数列表。需要注意的是：**如果想使用可变参数列表，则至少有一个固定参数。**

我们 C 语言常用的 printf 和 scanf 函数就是使用了可变参数列表的函数：

```c
int printf(const char *format, ...);

int scanf(const char *format, ...);
```


## 可变参数的提取

可变参数的提取主要依赖一个类型：va_list 和四个宏函数：va_start，va_arg，va_copy，va_end，而这些类型和宏函数都在 C 语言的头文件 <stdarg.h> 中。

**va_list 类型**：本质是一个 `char*` 类型，若使用可变参数列表，必须首先定义一个 va_list 类型的变量。

```c
va_list ap;

// ap：可变参数列表变量
```

**va_start 宏函数**：负责 va_list 类型的变量初始化。初始化后的 va_list 类型变量则指向第一个可变参数的首地址，该函数的函数原型如下：

```c
void va_start(va_list ap, last);

// last：最后一个传递给函数的已知的固定参数，即省略号之前的参数
```

**va_arg 宏函数**：提取可变参数列表中的参数。使用一次提取一个，每次提取的参数是直接返回的，并且该函数提取的同时会自动将 ap 指向下一个参数。

```c
type va_arg(va_list ap, type);

// type：要提取的参数的类型，如int , double，如果当前参数类型和type不统一，就会发生不可预知的错误
```

**va_copy 宏函数**：拷贝函数且不是必须使用的函数，初始化 dest 作为 src (当前状态) 的副本。

```c
void va_copy(va_list dest, va_list src);
```

**va_end 宏函数**：销毁 va_list 类型的变量，其本质就是将指针置为 NULL。

```c
void va_end(va_list ap);
```

## 例1：打印每一个参数

```c
#include <stdio.h>
#include <stdarg.h>

void PrintArgs(int num, ...) {
    va_list ap;
    va_start(ap, num);				// 1、初始化
    for (int i = 0; i < num; i++) {
        int a = va_arg(ap, int);	// 2、提取
        printf("%d ", a);
    }
    va_end(ap);						// 3、销毁
}

int main() {
    PrintArgs(4, 1, 3, 4, 5);
    return 0;
}

```


## 例2：包装一个格式化字符串

`vsnprintf`函数可以将**可变参数**，按照一定的格式，格式化为一个字符串，使用如下。

```c
int vsnprintf(char *str, size_t size, const char *format, va_list ap);

// str：缓冲区的起始地址
// size：缓冲区的大小
// 返回值：写入到缓冲区的字节数（不包括\0），如果返回值大于等于size意味着输出被截断了
```

代码实现：

```c
#include <stdarg.h>
#include <stdio.h>
#include <stdlib.h>

char *message(const char *fmt, ...) {
    va_list ap;

    va_start(ap, fmt);
    int byteLen = vsnprintf(NULL, 0, fmt, ap);
    va_end(ap);
    if (byteLen < 0)  return NULL;
  
    size_t size = (size_t)(byteLen + 1);
    char *p = malloc(size);
    if (p == NULL)  return NULL;

    va_start(ap, fmt);
    byteLen = vsnprintf(p, size, fmt, ap);
    va_end(ap);

    if (byteLen < 0) {
        free(p);
        return NULL;
    }
    return p;
}

int main(){
    const char* str = message("%d 年 hello, %s\n", 2024, "world");
    printf("测试message: %s\n", str);

    return 0;
}
```


# 可变参数的原理

以第一个例子分析：在调用函数 `PrintArgs` 时，其参数会先从右向左依次压栈。调用函数所使用的参数压入栈时数据是连续的。

那么也就是说我们只要拿到第一个参数的地址，后面所有的参数我们都可以拿到，只不过需要我们在读取数据时进行一下类型转换，改变一下指针的步长，保证我们拿到完整的数据。

1、为什么 va_list是 char* 类型？

    因为 char* 的指针读取数据时是按照1字节进行读取的，1字节读取方便我们读取数据。

2、为什么 va_list 类型的变量需要进行初始化，可变参数列表必须至少要有一个固定参数，而且 va_start() 函数的第二个参数必须是最后一个固定参数？

    这是因为初始化的工作就是将 va_list 类型的变量根据第一个固定参数，让其指向第一个可变参数。如果没有一个固定参数，就会导致 va_list 类型的变量不能指向第一个可变参数的地址。

3、为什么我们使用 va_arg() 函数提取参数时，第二个参数需要一个类型？

    因为只有根据这个类型，va_list 类型的变量才知道接下来要提取的参数的大小是多少字节。


**原理总结：**

- 可变参数列表对应的函数，最终调用也是函数调用，也要形成栈帧。
- 栈帧形成前，临时变量是要先入栈的，入栈的参数之间位置关系是固定的。
- 通过上面的特例我们发现了短整型在可变参数部分，会默认进行整形提升，那么函数内部在提取该数据的时候，就要考虑提升之后的值，如果不加考虑，获取数据可能会报错或者结果不正确。

**注意事项：**

- 可变参数必须从头到尾逐个访问。如果你在访问了几个可变参数之后想半途终止，这是可以的，但是，如果你想一开始就访问参数列表中间的参数，那是不行的。
- 参数列表中至少有一个固定参数。如果连一个固定参数都没有，就无法使用 va_start 。
- 这些宏是无法直接判断实际存在参数的数量，提取时提取的个数由你控制，或者通过其他的方式让这些宏知道参数的个数，例如 printf() 的格式控制时，就是根据%来确定参数的个数的。
- 这些宏无法判断每个参数的是类型，提取时你必须显示指定类型，或者通过其他方式让这些宏知道参数的类型，例如 printf() 的格式控制中，就是根据 % 后面的d，s，c，lf来确定参数的类型的。
- 如果在 va_arg 中指定了错误的类型，那么其后果是不可预测的。
------------------------------------------------

原文链接：https://blog.csdn.net/qq_65207641/article/details/132909901


# 可变参数宏

C 语言在 GCC 编译器下为了表示宏定义中的可变参数，引入了两个宏 `##__VA_ARGS__` 和 `__VA_ARGS__` 。

宏替换的方式调用可变参数函数。

**情况一：一个固定参数和一个可变参数列表**

```c
#include <stdarg.h>
#include <stdio.h>

#define PRINT(fmt, ...)     printf(fmt, __VA_ARGS__)

#define PRINTF(fmt, ...)     printf(fmt, ##__VA_ARGS__)

 int main() {
     PRINT("hello\n");              // 编译失败
     PRINTF("hello, %s\n", "world");// 打印成功
     return 0;
}
```

注意当可变参数列表为空，即只有一个固定参数时，编译会报错。

![[20240623200812.png]]

如果参数列表为空的话，就需要在 `__VA_ARGS__` 前面加上 `##`，这样就会去掉前面的逗号，防止编译错误。

**情况二：一个可变参数列表**

```c
#include <stdarg.h>
#include <stdio.h>

// 可以
#define DISPLAY(...)        printf(__VA_ARGS__)

// 
#define DISPLAYF(...)       printf(##_VA_ARGS__)  // 不可以

int main() {
    DISPLAY("hello\n");
    DISPLAY("hello, %s\n", "world");
    // DISPLAYF("project start\n"); // 报错
    return 0;
}
```

由此可见，`##__VA_ARGS__` 必须搭配固定参数使用，`__VA_ARGS__` 可以单独使用。同时，在搭配固定参数使用时候，`__VA_ARGS__` 在使用时候，除固定参数外，还必须要有一个可变参数，`##__VA_ARGS__`使用时，除了固定参数外，可以不包含任何可变参数。

上述的变参宏定义不仅能自定义输出格式，而且配合 `#ifdef #else #endif` 在输出管理上也很方便，比如调试时输出调试信息，正式发布时则不输出，可以这样：

```c
// 在调试环境下，LOG宏是一个变参输出
#ifdef DEBUG

#define LOG(format, ...) fprintf(stdout, ">> "format"\n", ##__VA_ARGS__)

#else

#define LOG(format, ...)
#endif
```

