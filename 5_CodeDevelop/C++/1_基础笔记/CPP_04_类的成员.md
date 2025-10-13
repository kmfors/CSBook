## 特殊数据成员

在 C++ 类中，有 4 种比较特殊的数据成员，他们分别是常量成员、引用成员、类对象成员和静态成员，他们的初始化与普通数据成员有所不同。

### 常量数据成员

在 C++ 中，类的 **常量数据成员（`const` 成员变量）** 必须在构造函数的 **初始化列表** 中进行初始化，而不能在类的定义中直接赋值或在构造函数体内赋值。这是因为：

1. **`const` 成员的不可修改性**：`const` 成员一旦初始化后就不能再修改，因此必须在对象创建时进行初始化。
2. **初始化列表的时机**：初始化列表在对象构造时执行，早于构造函数体的执行，因此适合用于初始化 `const` 成员。
3. **类内直接赋值的限制**：在 C++11 之前，类内非静态成员变量（包括 `const` 成员）不能在定义时直接初始化（C++11 引入了类内初始化，但 `const` 成员仍需在初始化列表中初始化）。

```c++
class Person{
public:
	Person(string name, int age)
	: _Name(name), _Age(age) {}
    // 必须显式在初始化列表中初始化（即使有类内初始化）
private:
	const string _Name;  // 常量数据成员
	const int _Age = 23; // C++11 允许类内初始化（但实际会被初始化列表覆盖）
};
```

> [!CAUTION]
>
> ### 关键点总结：
>
> 1. **必须通过初始化列表初始化 `const` 成员**。
> 2. **`static const` 整型/枚举成员可以在类内直接初始化**。
> 3. **C++11 允许 非静态 `const` 成员类内初始化，但初始化列表的优先级更高**（实际开发中仍建议显式初始化）。



### 引用数据成员

- **特点**：引用必须在初始化时绑定到一个变量，之后不能修改绑定目标。
- **初始化规则**：
  - **必须** 在构造函数的 **初始化列表** 中初始化（类似于 `const` 成员）。
  - 不能在构造函数体内赋值（因为引用必须在定义时初始化）。

```c++
class Person{
public:
	Person(int age)
	: _Age(age), _Ref(_Age) {}	
    // 引用数据成员必须显式在初始化列表中初始化
    
private:
	int &_Ref;// 引用数据成员
	int _Age;
};
```



### 类对象数据成员

- **特点**：如果数据成员是另一个类的对象（非内置类型），它的构造和析构受其类规则控制。
- **初始化规则**：
  - 如果该成员类 **没有默认构造函数**，则 **必须** 在初始化列表中显式调用其构造函数。
  - 如果成员类有默认构造函数，可以省略初始化列表（编译器会自动调用默认构造）。

```c++
class Line {
public:
    // 注意：如果Point类没有默认构造函数，则必须在初始化列表中显式调用其构造函数
    Line(int x1, int y1, int x2, int y2)
    : _pt1(x1, y1)    // 显式调用 Point(int, int) 构造函数
    , _pt2(x2, y2) {}

private:
    Point _pt1;  // 类对象成员：表示线段起点（Point类型）
    Point _pt2;  // 类对象成员：表示线段终点（Point类型）
};
```



### 静态数据成员

- **特点**：属于类而非对象，所有对象共享同一份静态成员。
- **初始化规则**：
  - **必须在类外单独初始化**（通常在 `.cpp` 文件中）。
  - C++17 允许用 `inline` 在类内直接初始化（非静态 `constexpr` 也可类内初始化）。
  - `static const` **整型或枚举** 成员可以在类内直接初始化（C++11）。

**静态数据成员的特点**：

- **全局唯一性**：
  - 静态数据成员属于 **类本身**，而非类的某个对象。
  - 无论创建多少个类对象，静态成员 **只有一份**，存储在 **全局/静态存储区**。
  - 不占用类对象的存储空间（`sizeof(class)` 不包含静态成员）。
- **生命周期**：
  - **在程序启动时初始化**（早于 `main()`），**在程序结束时销毁**（与全局变量相同）。
  - 即使没有创建任何类对象，静态成员仍然存在。
- **访问权限**：
  - 仍然受类访问控制（`public`/`private`/`protected`），但可以在类外定义时初始化（即使 `private`）。

```c++
class Database {
private:
    static int _connectionCount;   // 声明（类内）
    static Logger _errorLogger;    // 声明（类对象成员）
};

// 类外定义（全局作用域）
int Database::_connectionCount = 0;          // 基本类型初始化
Logger Database::_errorLogger("db_errors");  // 对象类型构造
```

静态数据成员具有类作用域，但它们的存储周期与程序生命周期相同。由于静态成员不属于任何类实例，因此 **声明与定义分离**

- 类内仅作声明（使用 `static` 关键字）
- 类外必须单独定义（不含 `static` 关键字）

因为静态数据成员不属于类的任何一个对象，所以它们并不是在创建类对象时被定义的。这意味着它们不是由类的的构造函数初始化的。一般来说，我们不能在类的内部初始化静态数据成员，**必须在类的外部定义和初始化静态数据成员，且不再包含 static 关键字，用类作用域声明**，格式如下:

```c++
// 基本类型静态成员
类型 类名::静态成员名 = 初始化值;  

// 对象类型静态成员
类型 类名::静态成员名(构造参数); 
```

> [!NOTE]
>
> - **C++17 起**：可用 `inline static` 在类内直接定义
> - 静态数据成员必须在类外定义（通常在实现文件 `.cpp` 中）



## 成员函数分类

在 C++ 中，类的普通成员函数可以进一步细分为 **静态成员函数** 和 **const 成员函数**，它们具有不同的特性和用途。

### 静态成员函数

成员函数可以定义成静态的，静态成员函数的特点:

- **属于类本身，而非对象**：
  - 不依赖于任何类实例（没有 `this` 指针）。
  - 可以直接通过类名调用（`ClassName::func()`），也可以通过对象调用（但不推荐）。
- **不能访问非静态成员**：
  - 只能访问 **静态数据成员** 或 **其他静态成员函数**。
  - 不能使用 `this` 指针（因为没有绑定到对象）。
- **没有 `const` 限定**：
  - 静态成员函数不能声明为 `const`（因为它不操作对象状态）。

> [!NOTE]
>
> 非静态成员函数可以访问静态的数据成员与静态的成员函数

1、对于静态成员函数而言，其第一个参数的位置没有隐含的 this 

2、静态的成员函数内部不能访问非静态的数据成员与非静态的成员函数，可以访问和调用静态数据成员和静态的成员函数

3、普通的成员函数可以访问静态的数据成员与静态的成员函数

4、**如果静态的成员函数想访问非静态的数据成员或者成员函数，可以使用函数传参的形式**，或者在静态成员函数体中创建对象

5、静态成员函数可以使用类名与作用域限定符的形式进行调用（独特之处，其他的非静态成员函数不能这么调用）





### const 成员函数

const 成员函数特点：

- **不修改对象状态**：
  - 承诺不会修改类的任何非静态数据成员（除非是 `mutable` 成员）。
  - 可以被 `const` 对象调用（如 `const MyClass obj; obj.func();`）。
- **可以访问所有成员（静态 + 非静态）**：
  - 但只能调用其他 `const` 成员函数（避免间接修改对象）。
- **可以重载非 `const` 版本**：
  - 编译器根据对象的 `const` 属性选择调用哪个版本。

> [!NOTE]
>
> 1. 非 const 的对象默认情况调用的是非 const 版本的成员函数；而 const 对象调用是 const 版本的成员函数
> 2. 非 const 对象也是可以调用 const 版本的成员函数的
> 3. const 对象是不能调用非 const 版本的成员函数



**第一条结论结果**：非 const 的对象默认情况调用的是非 const 版本的成员函数；而 const 对象调用是 const 版本的成员函数

```c++
#include <iostream>

using std::cout;
using std::endl;

class Person {
public:
	// 两个 print 函数可以重载，原因是隐含的 this 不一样

    // Person * const this
	void print() {
        cout << "非 const 函数 print 调用" << endl;
    }

    // const Person * const this
	void print() const {
        cout << "const 函数 print调用" << endl;
    }
};

int main() {
	Person p1;
	const Person p2;
	p1.print();	// 调用 非const 函数
	p2.print();	// 调用 const 函数
}
```





**第二条结论结果**：非 const 对象也是可以调用 const 版本的成员函数的

```c++
#include <iostream>

using std::cout;
using std::endl;

class Person {
public:
	// 两个 print 函数可以重载，原因是隐含的 this 不一样

    // Person * const this
	// void print() {
    //     cout << "非 const 函数 print 调用" << endl;
    // }

    // const Person * const this
	void print() const {
        cout << "const 函数 print 调用" << endl;
    }
};

int main() {
	Person p3;
	p3.print();	// 普通对象调用 const 函数
}
```





**第三条结论结果**：const 对象是不能调用非 const 版本的成员函数

```c++
#include <iostream>

using std::cout;
using std::endl;

class Person {
public:
	// 两个 print 函数可以重载，原因是隐含的 this 不一样

    // Person * const this
	void print() {
        cout << "非 const 函数 print 调用" << endl;
    }

    // const Person * const this
	// void print() const {
    //     cout << "const 函数 print 调用" << endl;
    // }
};

int main() {
	const Person p4;
	p4.print();	// 调用非 const 函数, 报错
}
```
