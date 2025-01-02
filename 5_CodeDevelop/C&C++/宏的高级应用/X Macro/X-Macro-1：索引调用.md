**X-Macro其实就是通过 `#define` 与 `#undef` 实现的一种宏定义的技巧。**

示例：`#undef` 可以取消定义宏，然后再通过 `#define` 重新定义宏，此时得到的 x 与 y 的值分别是 10 和 100。

```c++
#define X_MACRO(a, b)	a
int x = X_MACRO(10, 100)
#undef X_MACRO
    
#define X_MACRO(a, b)   b
int y = X_MACRO(10, 100)
#undef X_MACRO
```


规范一下 X Marcro 的使用：

# 宏-枚举

1、确定要使用的宏命令结构

```c++
// a, 意思：替换为普通文本a，后接一个逗号
#define X_MACRO(a, b)  a,
```

2、定义宏列表：将我们希望的宏命令，添加到宏列表中

```c++
#define MACROS_TABLE              \
    X_MACROS(TURN_ON,  turn_on)   \
    X_MACROS(TURN_OFF, turn_off)  
```

3、定义宏枚举：将宏列表转化为枚举形式

```c++
typedef enum  {
#define X_MACROS(a, b) a,
    MACROS_TABLE
#undef X_MACROS
    MACROS_MAX
} macros_enum;
```

宏枚举展开效果如下：宏命令直接转化为枚举变量对象。

```c++
typedef enum {
    TURN_ON,
    TURN_OFF,
    MACROS_MAX
} macros_enum;
```


# 宏-字符串数组

1、确定要使用的宏命令结构

```c++
// #a, 意思：替换为字符串形式的a，后接一个逗号
#define X_MACRO(a, b)  #a,
```

2、定义宏列表

```c++
#define MACROS_TABLE              \
    X_MACROS(TURN_ON,  turn_on)   \
    X_MACROS(TURN_OFF, turn_off)  
```

3、定义宏字符串数组：将宏命令转化为字符串数组形式

```c++
const char* get_str[] = 
{
    #define X_MACROS(a, b) #a,
    MACROS_TABLE
    #undef X_MACROS
};
```

宏宏字符串数组展开效果如下：宏命令直接转化为枚举变量对象。

```c++
const char* get_str[] = 
{
    "TURN_ON",
    "TURN_OFF"
};
```



# 宏-函数列表

1、确定要使用的宏命令结构

```c++
#define X_MACROS(a, b) b,
```


2、定义宏列表

```c++
#define MACROS_TABLE              \
    X_MACROS(TURN_ON,  turn_on)   \
    X_MACROS(TURN_OFF, turn_off)  
```

3、定义宏函数列表：将宏命令转化为函数列表形式

```c++
typedef void (*func)(void* p);

static void turn_on(void* p) {
    printf("%s \r\n", (char*)p);
}

static void turn_off(void* p) {
    printf("%s \r\n", (char*)p);
}

const func func_table[] = 
{
    #define X_MACROS(a, b) b,
    MACROS_TABLE
    #undef X_MACROS
};
```

宏宏字符串数组展开效果如下：宏命令直接转化为枚举变量对象。

```c++
const func func_table[] = 
{
    turn_on,
    turn_off
};
```


# 宏使用

由于宏-字符串数组和宏-函数列表，都是根据 MACROS_TABLE 这个宏列表拓展出来的，是一一对应的，所以我们可以搭配宏枚举，直接使用索引的方式来调用函数：

```c++
static void func_handle(macros_enum iter)
{
    if(iter < MACROS_MAX)
    {
        func_table[iter]((void*)get_str[iter]);
    }
}

int main(void) {
    func_handle(TURN_ON);
    func_handle(TURN_OFF);

    return 0;
}
```


# 整体代码

使用 X-MACRO 对于此类的命令消息处理十分高效简洁，非常实用，且拓展性非常强。

整体代码如下：

```c++
#include <stdio.h>

#define MACROS_TABLE              \
    X_MACROS(TURN_ON,  turn_on)   \
    X_MACROS(TURN_OFF, turn_off)  

// 宏-枚举
typedef enum {
#define X_MACROS(a, b) a,
    MACROS_TABLE
#undef X_MACROS
    MACROS_MAX
} macros_enum;

// 宏-字符串数组
const char* get_str[] =
{
    #define X_MACROS(a, b) #a,
    MACROS_TABLE
    #undef X_MACROS
    //"macros_str"
};


// 宏 - 函数列表
typedef void (*func)(void* p);

static void turn_on(void* p) {
    printf("调用打印：%s \r\n", (char*)p);
}

static void turn_off(void* p) {
    printf("调用打印：%s \r\n", (char*)p);
}

const func func_table[] =
{
    #define X_MACROS(a, b) b,
    MACROS_TABLE
    #undef X_MACROS
};


static void func_handle(macros_enum iter)
{
    if (iter < MACROS_MAX)
    {
        func_table[iter]((void*)get_str[iter]);
    }
}

int main(void) {
    func_handle(TURN_ON);
    func_handle(TURN_OFF);

    return 0;
}
```


