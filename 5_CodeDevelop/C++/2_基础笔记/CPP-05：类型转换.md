## 类型转换

C++语言中，如果两种类型有关联，那么当程序需要其中一种类型的运算对象时，可以用另一种关联类型的对象或值来替代。即这两种类型可以相互转换。

## 隐式类型转换

C++语言不会直接将两个不同的值进行操作，而是现根据类型转换规则设法将运算对象的类型统一后再求值，这个过程是自动执行的，被称为隐式转换。

隐式类型转换有以下几种：

- 算术转换
- 数组转换指针
- 指针转换
- 转换布尔类型
- 转换成常量
- 类类型定义的转换



## 显式类型转换

显式地将对象强制转换成另外一种类型，被称为显示类型转换。

### static_cast

`static_cast` 是 C++ 中的一种类型转换运算符，用于在兼容类型之间进行转换。它主要用于基本的、非多态的类型转换，比如内置类型之间的转换、类层次结构中向上转型（从派生类指针转换为基类指针）等。以下是 `static_cast` 的一些使用场景和特点：

1. **基本数据类型之间的转换**：比如从 `int` 转换为 `float`，或者从 `float` 转换为 `double`。
2. **类层次结构中的向上转型**：将派生类的指针或引用转换为基类的指针或引用。
3. **空指针与整数之间的转换**：将 `nullptr` 转换为整型，或者将整型转换为 `nullptr`。
4. **类类型之间的转换**：如果两个类之间有明确的继承关系，可以将一个类的指针或引用转换为另一个类的指针或引用。

`static_cast` 的基本语法如下：

```cpp
static_cast<目标类型>(表达式)
```

1. **基本数据类型转换**：

```cpp
int i = 10;
float f = static_cast<float>(i); // 将 int 转换为 float
```

2. **类层次结构中的向上转型**：

```cpp
class Base {};
class Derived : public Base {};

Derived d;
Base* b = static_cast<Base*>(&d); // 将 Derived 的地址转换为 Base 的指针
```

3. **空指针与整数之间的转换**：

```cpp
int* p = nullptr;
int i = static_cast<int>(p); // 将 nullptr 转换为 int
```

4. void* 到任意对象指针

```c++
void* p = &d;
double *dp = static_cast<double*>(p);
```

5. **类类型之间的转换**：由于没有动态类型检查，向下转换是不安全的

```cpp
class Base {};
class Derived : public Base {};

Base b;
Derived* d = static_cast<Derived*>(&b); // 将 Base 的地址转换为 Derived 的指针
```

> [!NOTE]
>
> - `static_cast` 不能用于多态向下转型（从基类指针转换为派生类指针），因为这需要运行时类型信息（RTTI），应该使用 `dynamic_cast`。
> - `static_cast` 不能用于去除对象的 const 属性，如果需要去除 const 属性，应该使用 `const_cast`。
> - `static_cast` 不能用于类层次结构中的交叉转换（比如从一个派生类转换到另一个非继承的派生类）。
> - `static_cast` 用于类层次结构中基类和派生类之间指针或引用的转换。进行向上转换（把派生类的指针或引用转换成基类指针或引用）是安全的；而进行向下转换（将基类指针或引用转换为派生类指针或引用）时，由于没有动态类型检查，所以是不安全的。
> - 使用 `static_cast` 时，编译器不会进行运行时检查，因此如果转换不安全，可能会导致未定义行为。
>
> 总的来说，`static_cast` 是一种安全的类型转换方式，适用于那些编译器可以保证安全的转换场景。在需要进行多态类型转换或者去除 const 属性时，应该使用 `dynamic_cast` 和 `const_cast`。



## const_cast（向非常量类型转换）

const_cast：去除常量属性。常量指针被转化成非常量指针，并且仍然指向原来的对象；

```c++
//强转去除const的保护，number可被其他指针指向
const int number = 10;
int *pInt = const_cast<int*>(&number);

// 不要这样，语法报错：未定义行为
*pInt = 20;
```

## dynamic_cast

该运算符主要用于基类和派生类的转换，尤其是向下转型的用法中（基类对象转换为其派生类对象）。

## reinterpret_cast 

该运算符可以处理无关类型之间的转换，即用在任意指针（或引用）类型之间的转换，以及指针与足够大的整数之间的转换。由此可以看出，reinterpret_cast 的效果很强大，但错误的使用 reinterpret_cast 很容易导致程序的不安全，只有将转换后的类型值转换回到其原始类型，这样才是正确使用 reinterpret_cast 方式。