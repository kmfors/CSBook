# 错误处理



## errno简介

`errno` 是一个预处理器宏，一些标准库函数通过写入正整数到 `errno` 指定错误。

通常，会将 `errno` 的值设置为错误码之一。错误码作为以字母 `E` 后随大写字母或数字开始的宏常量，**列于 `<errno.h>`（使用error需要引用头文件）** 。
`errno` 的值在程序启动时为 0 ，而且无论是否出现错误，都允许库函数将正整数写入 `errno` ，不过库函数决不会将 0 存储于 `errno` 。



总结：errno 是记录系统的最后一次的错误代码。代码是一个int型的值。当系统调用异常时,一般会将errno变量赋一个整数值,
不同的值表示不同的含义,可以通过查看该值推测出错的原因



> **注意：errno仅仅是一个数值，不具备任何错误描述，但不同的数值指定不同的错误描述。**



## stderr 标准错误输出流

当一个用户进程被创建的时候，系统会自动为该进程创建三个数据流：stdin、stdout、stderr

在默认情况下，stdout是行缓冲的，他的输出会放在一个 buffer 里面，只有到换行的时候，才会输出到屏幕。
而 stderr 是无缓冲的，会直接输出





## 获取errno错误描述

**库函数 [perror](https://zh.cppreference.com/w/c/io/perror) 和 [strerror](https://zh.cppreference.com/w/c/string/byte/strerror) 能用于获得对应当前 `errno` 值的错误条件的文本描述。**

都需要引入头文件 <string.h>



### strerror

返回指向系统错误码 `errnum` 的文本表示的指针，它等同于 perror() 会打印的描述。

```c
char* strerror( int errnum );
```

**`errnum` 通常获得自 `errno` 对象，不过函数接受任何 int 类型值。字符串的内容是本地环境限定指定的**

返回值：指向**静态只读字符串字面量**的不同指针，程序是无法修改这一段字符串，视为错误码 errno 的错误描述



### perror

两个功能：

1.  perror参数可以**自定义**字符串，来描述并打印自定义的错误信息
2. perror 参数可以接收来自strerror函数返回的错误码描述信息的指针，打印系统限定的错误码信息

```c
void perror( const char *s );
```

- **s** 指向带解释消息的空终止字符串的指针
- 实现定义的，描述存储于 `errno` 的错误码的错误消息字符串后随 '**\n**' 。错误消息字符串等同于 strerror(errno) 的结果

```c
#include <stdio.h>

int main(void){
    FILE *f = fopen("non_existent", "r");
    
    if (f == NULL) {
        perror("fopen() failed");	//实现自定义的错误描述输出
    } else {
        fclose(f);
    }
}
```

