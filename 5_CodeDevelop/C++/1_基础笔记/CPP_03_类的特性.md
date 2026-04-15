## 拷贝控制

C++ 的拷贝控制（Copy Control）是管理类对象生命周期和资源的核心机制。

### 核心成员函数
C++ 通过 6 个特殊成员函数控制对象的创建、拷贝、移动和销毁：
1. **默认构造函数**（`ClassName()`）
   无参或参数有默认值的构造函数，用于默认初始化对象。
2. **拷贝构造函数**（`ClassName(const ClassName&)`）
   用同类型对象初始化新对象（深拷贝或浅拷贝）。
3. **拷贝赋值运算符**（`ClassName& operator=(const ClassName&)`）
   将同类型对象的值赋给已存在的对象。
4. **移动构造函数**（`ClassName(ClassName&&)`，C++11）
   从右值（临时对象）转移资源初始化新对象。
5. **移动赋值运算符**（`ClassName& operator=(ClassName&&)`，C++11）
   将右值的资源转移给已存在的对象。
6. **析构函数**（`~ClassName()`）
   释放对象持有的资源（如内存、文件句柄）。

- 拷贝和移动构造函数定义了当用同类型的另一个对象初始化本对象时做什么。
- 拷贝和移动赋值运算符定义了将一个对象赋予同类型的另一个对象时做什么。
- 析构函数定义了当此类型对象销毁时做什么。

---

### 设计原则

三法则：

- **适用场景**：类管理动态资源（如堆内存）。
- **规则**：若自定义了 **析构函数**、**拷贝构造函数** 或 **拷贝赋值运算符** 中的任意一个，通常需要同时定义另外两个。
- **原因**：避免浅拷贝导致的重复释放或内存泄漏。

五法则（C++11）：

- **扩展**：在 Rule of Three 基础上，增加 **移动构造函数** 和 **移动赋值运算符**。
- **目的**：支持高效的资源转移，优化性能。

零法则：

- **理想目标**：通过 RAII（如 `std::unique_ptr`、`std::vector`）管理资源，依赖编译器生成的默认拷贝控制函数，避免手动实现。

---

### 深拷贝与浅拷贝
| **类型**   | **行为**                                 | **风险**                           | **适用场景**                 |
| ---------- | ---------------------------------------- | ---------------------------------- | ---------------------------- |
| **浅拷贝** | 仅复制指针值，不复制指向的资源           | 多个对象共享资源，可能导致重复释放 | 无资源所有权的简单对象       |
| **深拷贝** | 复制指针指向的资源，每个对象独立拥有资源 | 安全但可能性能开销大               | 管理动态资源的类（如字符串） |

```cpp
// 浅拷贝示例（默认行为）
class Shallow {
    int* data;
public:
    Shallow(const Shallow& other) : data(other.data) {} // 危险！
};

// 深拷贝示例
class Deep {
    int* data;
public:
    Deep(const Deep& other) : data(new int(*other.data)) {} // 安全
};
```

---

### 移动语义
- **核心目的**：避免不必要的深拷贝，提升性能。
- **右值引用**（`ClassName&&`）：绑定到临时对象（右值），允许转移资源的所有权。
- **移动操作**：
  - 移动构造函数/赋值运算符将资源所有权从右值转移到当前对象。
  - 转移后，源对象应处于有效但可析构的状态（如指针置 `nullptr`）。

```cpp
class MyString {
    char* data;
public:
    // 移动构造函数
    MyString(MyString&& other) noexcept : data(other.data) {
        other.data = nullptr; // 转移后置空源对象
    }
};
```

---

### 实现细节与陷阱
#### 拷贝赋值运算符
- **处理自赋值**：必须检查 `this != &other`。
- **异常安全**：先分配新资源，再释放旧资源。
```cpp
MyClass& operator=(const MyClass& rhs) {
    if (this != &rhs) {
        int* new_data = new int(*rhs.data); // 先分配
        delete data;                        // 后释放
        data = new_data;					// 重新分配
    }
    return *this;
}
```

#### **移动操作**
- **标记为 `noexcept`**：确保标准库容器（如 `std::vector`）在调整大小时优先使用移动而非拷贝。
- **置空源对象**：避免资源被重复释放。

---

### 现代 C++ 工具
#### 智能指针
- `std::unique_ptr`：独占所有权，自动管理资源，禁止拷贝，允许移动。
- `std::shared_ptr`：共享所有权，引用计数管理资源，支持拷贝和移动。

#### 默认与删除函数
- `= default`：显式要求编译器生成默认实现。
- `= delete`：禁用特定操作（如禁止拷贝）。
```cpp
class NonCopyable {
public:
    NonCopyable(const NonCopyable&) = delete;
    NonCopyable& operator=(const NonCopyable&) = delete;
};
```





## 移动语义

### 概念理解

**左值 (lvalue)**

- 是可以取地址的表达式（有持久的内存地址）
- 代表一个持久存在的对象或变量
- 示例：变量名、返回左值引用的函数调用、前置自增/减表达式等

**右值 (rvalue)**

- 是不能取地址的临时表达式
- 代表临时对象或即将销毁的值
- 包括：
  - 纯右值：字面量（如 42、true）、算术表达式结果
  - 将亡值：即将被移动的临时对象
- 生命周期短暂，通常存在于寄存器或栈上

### 关键点说明

1. **const 左值引用** 的特殊性：

```c++
const int& a = 10;  // 合法：绑定到右值
int x = 5;
const int& b = x;   // 合法：绑定到左值
```

- 这种灵活性使其无法区分左右值
- 但保证了不会修改被引用的对象

2. **右值引用** 的区分能力：

```c++
int x = 5;
int&& r1 = 10;     // 合法：绑定右值
int&& r2 = x;      // 错误：不能绑定左值
```

- 这种排他性使得函数重载时可以专门处理右值情况
- 是移动语义和完美转发的基础

**生命周期说明**：

- 右值确实通常短暂存在于栈上，但现代编译器会进行优化（如 RVO）
- 通过右值引用可以延长临时对象的生命周期（直到引用离开作用域）





### 移动构造函数

将拷贝构造函数以及赋值运算符函数称为具有复制控制语义的函数。
将移动构造函数以及移动赋值运算符函数称为具有移动语义的函数。

1、移动语义的函数优先于具有拷贝语义的函数的执行
2、具有移动语义的函数如果不写的话，编译器是不会自动生成，必须手写

```c++
String(String &&rhs)
: _pstr(rhs._pstr)//将左值指向右值的空间
{
	cout << "String(String &&)" << endl;
	rhs._pstr = nullptr; //右值空间被转移后，使命完成，为避免共同指向销毁，右值指向为空
}
```



### 移动赋值运算符函数

```c++
String &operator=(String &&rhs)
{
	cout << "String &operator=(String &&)" << endl;
	if(this != &rhs)//1、自移动
	{
		delete [] _pstr; //2、释放左操作数
		_pstr = nullptr;
		_pstr = rhs._pstr; //3、浅拷贝，指向临时开辟的空间
		rhs._pstr = nullptr; //改变临时的指向，防止临时销毁后空间也跟着销毁
	}
	return *this;//4、返回*this
}
```



### std::move 函数

将左值转换为右值，在内部其实上是做了一个强制转换，`static_cast<T &&>(lvaule)`

```c++
//s1是左值，现在将s1转换成右值，右值s1 = s1
s1 = std::move(s1);
cout << "s1 = " << s1 << endl;// 执行为空
//在移动赋值运算函数中，s1指向nullptr，那右边的s1也就为空了，再去给s1赋值，那其实大家都是空指针了嘛，所以打印为空
```

几个问题注意一下：

1、`String("world") = String("world")` 这里不存在自复制，因为这里虽然字符串一样，但是确实两个不一样的临时对象，且这样做无意义

2、自移动或者自复制这一步操作还是要有的，就是为了避免出现 `s1 = std::move(s1) ` 这样的操作。



## RVO 技术

编译器优化技术 `RVO`(Return Value Optimization)

当函数需要返回一个对象的时候，如果自己创建一个临时对象用户返回，那么这个临时对象会消耗一个构造函数(Constructor)的调用、一个复制构造函数的调用(Copy Constructor)以及一个析构函数(Destructor)的调用的代价。

而如果稍微做一点优化，就可以将成本降低到一个构造函数的代价

http://blog.csdn.net/zwvista/article/details/6845020

http://www.cnblogs.com/xkfz007/articles/2506022.html



## RAII 思想

RAII：利用栈对象的生命周期管理资源

RAII 的核心思想是 **将资源（内存、文件、锁等）的生存期绑定到对象的生存期**，确保资源在对象构造时获取，在对象析构时释放，从而避免资源泄漏。

### 四个基本特征

1. **构造函数中获取资源**（分配内存、打开文件、加锁等）
2. **析构函数中释放资源**（释放内存、关闭文件、解锁等）
3. **提供访问资源的方法**（如文件读写、指针解引用等）
4. **通常禁止拷贝和赋值**（避免资源重复释放或所有权混乱）



### 关键原则

1. **资源获取即初始化**：

   - 在构造函数中获取资源，确保资源立即可用。
   - 若构造失败（如文件打开失败），应抛出异常，避免无效对象。

2. **资源释放顺序**：**必须保证析构顺序与构造顺序严格相反**（栈式后进先出，LIFO）。

   ```c++
   {
       File file("a.txt");  // 构造顺序：file → lock
       Lock lock(mutex);
   } // 析构顺序：lock → file（自动反向释放）
   ```

3. **所有权明确**：若需转移资源所有权，应使用移动语义（`std::move`）或智能指针（`std::unique_ptr`）。



### RAII 的代码实现

```c++
class Point {
public:
    Point(int ix = 0, int iy = 0)
    : _ix(ix), _iy(iy) {
        cout << "Point(int = 0, int = 0)"  << endl;
    }

    void print() const {
        cout << "(" << _ix << ", " << _iy
             << ")" << endl;
    }

    ~Point() { cout << "~Point()" << endl; }
private:
    int _ix;
    int _iy;
};

template <typename  T> 
class RAII {
public:
    //1、在构造函数中初始化资源
    RAII(T *data) : _data(data) {
        cout <<"RAII(T *)" << endl;
    }

    //2、在析构函数中释放资源
    ~RAII() {
        cout << "~RAII()" << endl;
        if(_data) {
            delete _data;
            _data = nullptr;
        }
    }

    //3、提供若干访问资源的方法
    T *operator->() { return _data; }

    T &operator*() { return *_data; }

    T *get() const { return _data; }

    // 4、重置
    void reset(T *data) {
        if(_data) {
            delete _data;
            _data = nullptr;
        }
        _data = data;
    }

    // 4、不允许复制或者赋值
    RAII(const RAII &rhs) = delete;
    RAII &operator=(const RAII &rhs) = delete;
private:
    T *_data;
};

void test()
{
    RAII<int> raii(new int(10));
    
    RAII<Point> pt(new Point(1, 2));
    pt->print();
    (*pt).print();
}
```

