关于 C 语言可变参数的讲解已经放在这里了。 [[C可变参数|C语言可变参数]]

C 语言变参函数缺点：

1. 缺乏类型检查，容易出现不合理的强制类型转换。在获取实参时，是通过给定的类型进行获取，如果给定的类型与实际参数类型不符，则会出现类型安全性问题，容易导致获取实参失败。
2. 不支持自定义类型。自定义类型在程序中经常用到，比如我们要使用 printf () 来打印一个 Student 类型的对象的内容，该用什么格式字符串去指定实参类型，通过 C 提供的 va_list，我们无法提取实参内容。

鉴于以上两点，李健老师在其著作《编写高质量代码改善 C++程序的 150 个建议》建议尽量不要使用 C 风格的变参函数。

原文链接：https://blog.csdn.net/K346K346/article/details/53241115

---


# initializer_list 类型

要求所有实参的类型相同，从而可以传递一个名为 initializer_list 的标准库类型的变量作为可变参数。

initializer_list 提供的操作有：

```c++
// 默认初始化，空列表
initializer_list<T> lst;
// 直接初始化
initializer_list<T> lst{a, b, c...};

lst2(lst)
lst2 = lst
lst.size()
lst.begin()
lst.end()
```

注意：initializer_list 对象中的元素永远都是常量值，只作为参数传递是无法改变的。


# 可变参数模板

一个可变参数模板就是一个接受可变参数目参数的模板函数或模板类。可变数目的参数被称为参数包。

存在两种参数包：

- 模板参数包：表示零个或多个模板参数。
- 函数参数包：表示零个或多个函数参数。

我们用一个省略号（...）来指出一个模板参数或函数参数，来表示一个包。

## 模板参数与函数参数列表

在一个模板参数列表中，class... 或 typename... 指出接下来的参数表示零个或多个类型的列表。

```c++
// 模板参数列表，Args为模板参数包。这个过程称为打包
template <typename T, typename ...Args>
```

在一个函数参数列表中，使用模板参数包（例如 Args），用来指出接下来的参数表示零个或多个类型的列表。

```c++
// 函数参数列表，args为函数参数包。这个过程称为打包
void func(const T& t, const Args& ...args);
```

    注意区分打包与解包：
    1、...args 可以看作，无数个参数打入了args里，称为打包
    2、args... 可以看作，从args里解出来无数个参数，称为解包

## sizeof... 运算符

**返回参数包中的数目**。返回的是一个常量表达式，而且不会对其实参求值。

```c++
template<typename ...Args> 
void NumsList(Args ...args) {
    cout << sizeof...(Args) << endl;  // 类型参数的数目
    cout << sizeof...(args) << endl;  // 函数参数的数目
}

// NumsList("hello", "world"); // 数目为2
```

## 可变参数函数

可变参数函数的定义通常是**递归**的，第一步调用处理包中的第一个实参，然后用剩余实参调用自身。同时还需要定义一个非可变参数的函数来终止递归。代码如下：

```c++
// 递归出口（无参）
void display(){
    cout << endl;
}

// 模板参数列表至少有一个参数T
template<typename T, typename ...Args>
void display(const T& t, const Args& ...args){
// 函数参数列表至少有一个参数t
    cout << t << " ";
    display(args...);
}
// display("My", "year", "old", "is", 20);
```

注意递归的出口函数可以是无参也可以是有参，且有参优先于无参：

```c++
// 递归出口（无参）
void display() {
    cout << " display()" << endl;
}

// 递归出口（有参），优先于无参（是互斥关系）
template<typename T>
void display(const T& t) {
    cout << t << endl;
    cout << "display(const T&)" << endl;
}
```
