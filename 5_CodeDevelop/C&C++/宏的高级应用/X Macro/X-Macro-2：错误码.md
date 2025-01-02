X-Macro 放在项目里用做错误码也是可以的。也还是沿用我们经典的老套路，去如何设计高效的宏使用。

1、确定要使用的宏命令结构

```c++
// name = code, 作用：定义枚举值，逗号进行隔开
#define XX(name, code, _) name = code,
```


2、定义宏列表：将我们希望的宏命令，添加到宏列表中

```c++
#define PROJECT_CODE_MAP(XX)           \
        XX(SUC_CODE, 0, "成功")         \
        XX(ERR_INVALID_PB,  "失败")
```

3、定义宏枚举：将宏列表转化为枚举形式

```c++
typedef enum {
#define XX(name, code, _) name = code,
    PROJECT_CODE_MAP(XX)
#undef XX
} PROJECT_CODE_ENUM;
```

4、定义宏 case，switch 展开宏列表

```c++
#define PROJECT_CODE_MSG(name, code, msg)           \
    case name:                                      \
        return msg;

inline const char* getMsgByCode(int err) {
    switch (err) {
        PROJECT_CODE_MAP(PROJECT_CODE_MSG)
    default:
        return ERR_UNKNOW_MSG;
    }
}
```

# 整体代码 

```c++
#include <iostream>
#include <string>

#define ERR_CODE_BASE 1000

const char* ERR_UNKNOW_MSG = "未知错误";


#define PROJECT_CODE_MAP(XX)                             \
        XX(SUC_CODE, 0, "成功")                           \
        XX(ERR_INVALID_PB, (ERR_CODE_BASE + 1), "失败")
  

typedef enum {
#define XX(name, code, msg) name = code,
    PROJECT_CODE_MAP(XX)
#undef XX
} PROJECT_CODE_ENUM;


#define PROJECT_CODE_MSG(name, code, msg)           \
    case name:                                      \
        return msg;

inline const char* getMsgByCode(int err) {
    switch (err) {
        PROJECT_CODE_MAP(PROJECT_CODE_MSG)
    default:
        return ERR_UNKNOW_MSG;
    }
}


int main() {

    std::cout << std::string(getMsgByCode(1001)) << std::endl;

    return 0;
}
```


