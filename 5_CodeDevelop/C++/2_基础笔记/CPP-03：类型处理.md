# CPP-03 : 类型处理

## 一、知识讲解

### 1.1 类型别名

- 传统别名：使用 **typedef** 来定义类型的同义词。 `typedef double wages;`
- 新标准别名：别名声明（alias declaration）： `using SI = Sales_item;`（C++11）

### 1.2 auto 类型说明符 

- **auto**类型说明符：让编译器通过初始值**自动推断类型**。
- 一条声明语句只能有一个数据类型，所以一个 auto 声明多个变量时只能相同的变量类型。
- 会忽略 `顶层const`。

### 1.3 decltype 类型指示符

**decltype**：选择并返回操作数的**数据类型**。从表达式的类型推断出要定义的变量的类型，并用该类型初始化变量，并**不会忽略顶层 const** 。

```c++
// 推断 sum 的类型是函数 f() 的返回类型。
decltype(f()) sum = x;
```

- 如果对变量加括号，编译器会将其认为是一个表达式，得到的结果为这种类型的引用。

```c++
  int i = 42;
  decltype((i)) d; // 错误，d 是 int&，必须初始化
  decltype(i) e;   // 正确，e 是一个（未初始化的）int
  ```

赋值是会产生引用的一类典型表达式，引用的类型就是左值的类型。也就是说，如果 i 是 int，则表达式 i=x 的类型是 int&。

```c++
intd func(int, int);

decltype(f)* pfunc; // 函数指针类型
```


decltype 指明函数指针类型，由于 decltype(end_connection) 返回一个函数类型，所以我们必须添加一个 * 来指出我们正在使用该类型的一个指针。