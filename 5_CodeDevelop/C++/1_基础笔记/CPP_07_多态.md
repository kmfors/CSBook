# 多态

## 多态概念
多态作为面向对象三大特性之一，实现了：
1. **接口与实现的解耦**：调用者只需关注接口语义（what），无需知晓具体实现细节（how）
2. **系统可扩展性**：允许新增功能时最小化修改现有代码（符合开闭原则）

C++支持两种多态机制：

**编译时多态（静态多态）**：

- 实现方式：函数重载、运算符重载、模板（泛型编程）
- 特征：编译器在编译阶段通过函数签名（函数名+参数列表）即可确定具体调用版本
- 绑定机制：早期绑定（early binding），又称静态联编

**运行时多态（动态多态）**：

- 实现方式：虚函数（virtual functions）配合继承体系
- 特征：当通过基类指针/引用调用虚函数时，具体调用的函数版本需在运行时根据对象实际类型决定
- 绑定机制：晚期绑定（late binding），通过虚函数表（vtable）机制实现动态联编

需要特别说明的是：

1. C++标准中 "多态" 特指通过虚函数实现的运行时多态
2. 动态多态是面向对象设计的关键特性，支持 "开闭原则"（对扩展开放，对修改关闭）
3. 静态多态中的模板元编程（TMP）可实现编译期多态，但属于更高级的模板应用

### 实现机制

- **静态绑定（编译时绑定）**：默认的函数调用方式（如非虚函数），调用目标在编译期确定。
- **动态绑定（运行时绑定）**：通过虚函数实现，调用目标在运行时根据对象实际类型决定。

| 类型         | 实现方式                   | 绑定时机 | 核心机制                  |
| ------------ | -------------------------- | -------- | ------------------------- |
| **静态多态** | 函数重载、运算符重载、模板 | 编译期   | 名称修饰（name mangling） |
| **动态多态** | 虚函数 + 继承体系          | 运行期   | 虚函数表（vtable）        |

### 绑定机制对比
**静态绑定（Early Binding）**：
- 编译期间通过函数签名（函数名+参数类型）确定调用实体
- 示例场景：
  ```cpp
  // 函数重载
  void print(int x) { cout << "Integer: " << x; }
  void print(double x) { cout << "Double: " << x; }
  
  print(5);    // 调用print(int)
  print(3.14); // 调用print(double)
  ```

**动态绑定（Late Binding）**：
- 通过虚函数表和对象类型信息在运行时确定调用实体
- 示例场景：
  ```cpp
  class Shape {
  public:
      virtual void draw() const { cout << "Drawing base shape\n"; }
  };
  
  class Circle : public Shape {
  public:
      void draw() const override { cout << "Drawing circle\n"; }
  };
  
  Shape* shape = new Circle();
  shape->draw(); // 实际调用Circle::draw()
  ```

### 关键差异可视化
```cpp
// 静态绑定示例（模板）
template <typename T>
T add(T a, T b) { return a + b; }

add(2, 3);    // 实例化为add<int>
add(2.5, 3.1); // 实例化为add<double>

// 动态绑定示例（工厂模式）
class Animal {
public:
    virtual void speak() = 0;
};

class Dog : public Animal {
    void speak() override { cout << "Woof!"; }
};

class Cat : public Animal {
    void speak() override { cout << "Meow!"; }
};

Animal* createAnimal(const string& type) {
    if(type == "dog") return new Dog();
    if(type == "cat") return new Cat();
    return nullptr;
}

// 运行时根据输入决定具体类型
Animal* pet = createAnimal(userInput); 
pet->speak(); // 动态绑定到具体实现
```

---

### 常见误区澄清
1. **函数隐藏 vs 覆盖**：
   ```cpp
   class Base {
   public:
       void func() {}        // 非虚函数
       virtual void vfunc() {}
   };
   
   class Derived : public Base {
   public:
       void func() {}        // 函数隐藏（静态绑定）
       void vfunc() override {} // 函数覆盖（动态绑定）
   };
   ```

2. **final 关键字影响**：
   ```cpp
   class Base final { /*...*/ };     // 禁止继承
   
   virtual void func() final;        // 禁止覆盖
   ```
   
3. **动态绑定开销**：
   - 虚函数调用比普通函数多一次指针解引用（访问 `vtable`）
   - 每个含虚函数的类增加一个 `vptr` 指针（通常 4-8 字节）

通过合理运用两种绑定机制，开发者可以在系统灵活性和运行效率之间取得最佳平衡。



## 虚函数

在基类中使用 `virtual` 关键字声明函数，被声明的函数成为虚函数。作用：允许派生类通过重写（override）函数，改变行为。

### 重定义（重写）要求

- 基类与派生类中 **函数名** 相同
- 函数的 **参数列表** 相同（包括参数的个数、参数的类型、参数的顺序）
- 函数的 **返回类型** 一致

**当通过基类指针（或引用）操作派生类对象时：**

- **若函数是普通成员函数（非虚函数）**：调用的是 **基类中定义的版本**，因为此时函数的调用由指针的类型（基类）在编译时决定。
- **若函数是虚函数**：调用的是 **派生类中重写的版本**，因为虚函数通过动态绑定机制，在运行时根据对象的实际类型（派生类）决定调用。



```c++
class Base {
public:
	void func() { cout << "Base::func" << endl; }  // 非虚函数
	virtual void vfunc() { cout << "Base::func" << endl; }  // 虚函数
};

class Derived : public Base {
public:
	void func() { cout << "Derived::func" << endl; }  // 隐藏基类同名函数
	void vfunc() override { cout << "Derived::func" << endl; }  // 重写基类虚函数
};

int main() {
	Derived der1;
	Base* p1 = &der1;
	p1->func();  // 输出 Base::func（静态绑定，只看指针类型）

	Derived der2;
	Base* p2 = &der2;
	p2->vfunc();  // 输出 Derived::vfunc（动态绑定，看对象实际类型）
	
	return 0;
}
```

> [!NOTE]
>
> - 基类与派生类中的同名虚函数，除了 **函数体** 可以不一样之外，其他的全部都要保持一致。
> - 多态的典型应用场景是通过 **基类指针或引用指向派生类对象** 来实现的
> - 基类中被声明为虚函数的成员函数，在派生类中重写时加不加 `virtual` 都是虚函数。

### 虚函数的原理

**虚函数表**（vtable）：每个包含虚函数的类（或继承自含虚函数的类）都会生成一个 **虚函数表**，它是一个函数指针数组，存储该类所有虚函数的实际地址。

```c++
class Animal {
public:
    virtual void speak() { ... }    // 虚函数
    virtual void eat() { ... }     // 虚函数
    virtual ~Animal() { ... }      // 虚析构函数
};

// Animal 的虚函数表（伪代码）：
vtable_Animal = [
    &Animal::speak,    // 第一个虚函数地址
    &Animal::eat,       // 第二个虚函数地址
    &Animal::~Animal    // 虚析构函数地址
];
```

**虚函数指针**（vptr）：每个对象实例的内存布局中会隐式添加一个 **指向其所属类的虚函数表** 的指针（`vptr`）。该指针在对象构造时由编译器自动初始化。

```c++
Animal cat;
// 对象实例 cat 的内存布局
+----------------+
| vptr  ---------|-----> 指向 Animal 的虚函数表
| 其他成员变量     |
| ......	     |
+----------------+
```

**派生类的虚函数表**：派生类继承基类的虚函数表，并 **覆盖** 重写里面的虚函数地址。未重写的虚函数仍指向基类实现。

**调用过程**：当通过基类指针或引用调用虚函数时，编译器会生成代码，通过对象的 `vptr` 找到虚函数表，再根据函数在表中的偏移量找到实际函数地址。

```c++
Animal* animal = new Dog();
animal->speak();  // 动态绑定过程：
                  // 1. 通过 animal 的 vptr 找到 Dog 的虚函数表
                  // 2. 从表中取 speak() 的地址（Dog::speak）
                  // 3. 调用该地址的函数
```

**虚析构函数** 的作用：

- **必要性**：如果基类的析构函数不是虚函数，通过基类指针删除派生类对象时，只会调用基类的析构函数，导致派生类资源泄漏。

- **原理**：虚析构函数会被加入虚函数表中，确保通过基类指针删除对象时，正确调用派生类的析构函数。

虚函数的本质是 **通过间接寻址实现多态**，是 C++ 运行时多态的核心设计。



### 动态多态激活条件

动态多态（运行时多态）的激活需要满足以下 **五个核心条件**，缺一不可：

#### 基类声明虚函数

- **必要条件**：基类中必须显式声明至少一个 **虚函数**（使用 `virtual` 关键字）。  
  
  ```cpp
  class Base {
  public:
      virtual void func();  // 声明为虚函数
  };
  ```

#### 派生类重写虚函数

- **必要条件**：派生类必须对基类的虚函数进行 **重写（override）**，且满足以下条件：  
  - **函数签名一致**：函数名、参数列表、返回类型（协变返回类型允许例外）必须完全相同。  
  - **使用 `override` 关键字（C++11+）**：显式标记重写，避免隐藏（hiding）基类函数。  
  ```cpp
  class Derived : public Base {
  public:
      void func() override;  // 正确重写基类虚函数
  };
  ```

#### 通过基类指针或引用调用
- **必要条件**：必须通过 **基类指针** 或 **基类引用** 操作派生类对象。  
  - 直接通过对象调用虚函数时，多态失效（静态绑定）。  
  ```cpp
  Derived derived;
  Base* ptr = &derived;   // 基类指针指向派生类对象
  Base& ref = derived;    // 基类引用绑定派生类对象
  ptr->func();            // 动态绑定（多态生效）
  ref.func();             // 动态绑定（多态生效）
  ```

#### 对象实际类型为派生类
- **必要条件**：基类指针或引用实际指向的是 **派生类对象**。  
  - 若指向的是基类对象，调用基类函数。  
  ```cpp
  Base base;
  Derived derived;
  Base* ptr1 = &base;     // 指向基类对象
  Base* ptr2 = &derived;  // 指向派生类对象
  ptr1->func();           // 调用 Base::func()
  ptr2->func();           // 调用 Derived::func()
  ```

#### 不在构造函数或析构函数中调用
- **必要条件**：虚函数的调用不能发生在 **构造函数** 或 **析构函数** 中。  
  - 在构造函数中，对象尚未完成派生类部分的构造，此时 `vptr` 指向基类的虚表。  
  - 在析构函数中，派生类部分已被销毁，此时 `vptr` 指向基类的虚表。  
  ```cpp
  class Base {
  public:
      Base() {
          func();  // 调用 Base::func()，多态失效
      }
      virtual void func() { ... }
  };
  
  class Derived : public Base {
  public:
      Derived() {
          func();  // 调用 Derived::func()
      }
      void func() override { ... }
  };
  ```





### 无法设置为虚函数的函数

在 C++中，虚函数是实现运行时多态的核心机制，但并非所有函数都可以声明为虚函数。以下是 **不能被设置为虚函数** 的函数类型及其原因：

#### 非成员函数（包括友元函数）

- **原因**：虚函数必须是类的成员函数，而非成员函数（如全局函数或友元函数）没有所属的类，无法通过对象实例调用，因此不能声明为虚函数。

  ```cpp
  class MyClass {
  public:
      friend void friendFunc(); // 友元函数，不能是虚函数
  };
  
  void globalFunc() {}          // 全局函数，不能是虚函数
  ```



#### 静态成员函数（`static` 成员函数）
- **原因**：静态成员函数属于类本身，而非对象实例。它没有隐式的 `this` 指针，无法访问对象的虚函数表（vtable），因此无法实现动态绑定。

  ```cpp
  class MyClass {
  public:
      static void staticFunc(); // 静态函数，不能是虚函数
  };
  ```



#### 构造函数
- **原因**：构造函数用于初始化对象，此时对象的虚表指针（`vptr`）尚未正确指向派生类的虚表（在构造函数执行期间，`vptr` 会逐步调整）。如果构造函数是虚函数，会导致未定义行为，因此 C++ 直接禁止虚构造函数。

  ```cpp
  class Base {
  public:
      virtual Base() {} // 错误：构造函数不能是虚函数
  };
  ```

#### 模板成员函数

- **原因**：模板函数的实例化会生成多个具体版本的函数，而虚函数需要在类的虚表中占据固定位置。模板成员函数的动态性和虚函数表的静态布局冲突，因此不能声明为虚函数。

  ```cpp
  class MyClass {
  public:
      template<typename T>
      virtual void templateFunc(T arg) {} // 错误：模板成员函数不能是虚函数
  };
  ```

#### 内联函数（`inline` 函数）
- **矛盾点**：  
  - **语法允许**：C++允许将内联函数声明为虚函数（例如 `virtual inline void func() {}`）。  
  - **实际行为**：当通过基类指针/引用调用虚函数时，由于需要动态绑定，编译器会忽略内联请求，直接通过虚函数表寻址。此时内联无效，但仍可正常使用多态。
- **结论**：内联虚函数在语法上是合法的，但“内联”特性在动态调用时会失效。严格来说，内联函数可以被声明为虚函数，但无实际优化意义。

#### 某些运算符重载
- **允许的运算符**：大多数运算符重载可以声明为虚函数（如 `operator+`、`operator==` 等）。  

- **例外**：  
  
  - **`operator=`（赋值运算符）**：可以声明为虚函数，但需谨慎处理对象切片问题。  
  - **`new`/`delete`（内存管理运算符）**：不能是虚函数，因为它们本质上是静态的，与对象实例无关。
  
  ```cpp
  class MyClass {
  public:
      virtual MyClass& operator=(const MyClass& other) { // 合法，但需注意多态赋值问题
          return *this;
      }
      
      void* operator new(size_t size) { // 不能是虚函数
          return malloc(size);
      }
  };
  ```

#### 总结
| **函数类型**         | **能否为虚函数**               | **原因**                                 |
| -------------------- | ------------------------------ | ---------------------------------------- |
| 非成员函数（含友元） | 否                             | 必须为成员函数                           |
| 静态成员函数         | 否                             | 无 `this` 指针，无法动态绑定             |
| 构造函数             | 否                             | 对象构造期间 `vptr` 未稳定               |
| 模板成员函数         | 否                             | 实例化后生成多个函数，与虚表固定布局冲突 |
| 内联函数             | 语法允许，但动态调用时内联失效 | 动态绑定需通过虚表寻址                   |
| `new`/`delete`       | 否                             | 静态成员函数特性                         |

## 抽象类

在 C++ 中，**抽象类（Abstract Class）** 是一种特殊的类，用于定义 **接口规范** 和 **共享实现逻辑**，但它本身 **不能被实例化**，只能作为其他类的基类。抽象类的核心目的是强制子类遵循特定的设计规则，同时提供可复用的代码。

核心特性：

1. 通过纯虚函数定义：抽象类必须包含至少一个 **纯虚函数**，声明方式为在虚函数后添加 `= 0`。
2. 不可实例化：任何尝试创建抽象类对象的行为都会导致编译错误
3. 子类必须重写基类中的所有纯虚函数，否则子类也会成为抽象类
4. 抽象类可以拥有：**非虚函数**（子类直接继承）、**普通成员变量**、**已实现的虚函数**（非纯虚函数）。
5. 为了确保正确释放子类资源，**抽象类的析构函数** 必须声明为 `virtual`



> [!NOTE]
>
> 在 C++ 中，**纯虚函数** 是一种特殊的虚函数，它的核心作用是定义 **接口规范**，强制子类必须实现该函数。
>
> - **语法**：在虚函数声明末尾添加 `= 0`。
> - **特点**：
>   - 基类中 **不提供实现**（但允许在基类外单独定义默认实现）。
>   - 使基类成为 **抽象类（Abstract Class）**，无法直接实例化。
>   - 子类必须重写（Override）所有纯虚函数，否则子类也会成为抽象类。



> [!CAUTION]
>
> 在 C++ 中，虽然纯虚函数要求子类必须重写，但基类仍然可以为纯虚函数提供默认实现，子类需要通过基类作用域显式调用这个默认实现。
>
> ```c++
> // 基类声明纯虚函数，但提供默认实现
> class Animal {
> public:
>     virtual void makeSound() = 0; // 纯虚函数声明
> };
> 
> // 纯虚函数的默认实现（独立于类定义）
> void Animal::makeSound() {
>     std::cout << "Unknown sound" << std::endl;
> }
> 
> // 子类 1：显式调用基类的默认实现
> class Cat : public Animal {
> public:
>     void makeSound() override {
>         Animal::makeSound(); // 显式调用基类的默认实现
>     }
> };
> ```





---

## 自己总结



```c++
#include <iostream>
using namespace std;

class Base {
public:
	void Func1() {
		cout << "Base::Func1()" << endl;
	}
private:
	int m_num_b = 1;
};

int main() {
	Base base;
	cout << "sizeof(Base): " << sizeof(Base) << endl;
	return 0;
}
```

在调试结果中我们发现，`sizeof(Base)` 的大小为 4 字节，对应着其 Base 类的数据成员的大小。而当我们为 Func1 函数设置为虚函数时：

```c++
class Base {
    //...
    // 添加virtual关键字，设置虚函数
    virtual void Func1() { /*...*/ }
    // ...
};
```

继续调试运行发现，`sizeof(Base)` 的大小为 16 字节，除去数据成员的大小，还剩余的 12 字节从哪来呢？

- 64 位系统通常以 **8 字节（64 位）** 为单位读取内存数据的
- 为了保证 `Base` 对象的起始地址始终是 **8 字节对齐**，编译器会在 `m_num_b` 后插入 **4 字节的填充**
- `vptr`（8 字节） + `m_num_b`（4 字节） + 对齐填充（4 字节）= 16 字节

为什么需要填充到 16 字节？

- 如果对象大小为 12 字节
  - 当创建 `Base` 数组时，第二个对象的 `vptr` 会从 `12` 字节开始，而 CPU 读取未对齐的 `vptr` 可能需要额外周期，降低性能
- 填充到 16 字节后
  - 每个对象的 `vptr` 始终位于 `0, 16, 32,...` 字节处，CPU 可以高效地一次性读取 `vptr`

请注意 `vptr` 是虚函数表指针（二级指针），所以当类类型内存在有虚函数的定义时，类内会隐式创建一个指向虚函数表的指针成员，用于支持动态多态。

### 重新优化



```c++
#include <iostream>
using namespace std;

class Base {
public:
	virtual void Func1() {
		cout << "Base::Func1()" << endl;
	}
	virtual void Func2() {
		cout << "Base::Func2()" << endl;
	}
private:
	int m_num_b = 1;
};

class Derive : public Base {
public:
	void Func1() override {
		cout << "Derive::Func1()" << endl;
	}
	int m_num_d = 2;
};

int main() {
	Base base;
	Derive deri;
	cout << "sizeof(Base): " << sizeof(Base) << endl;
	cout << "sizeof(Derive): " << sizeof(Derive) << endl;

	return 0;
}
```

运行结果如下：

```
sizeof(Base): 16
sizeof(Derive): 24
```

调试运行发现以下地址

```c++
Base::vptr -> 0x1614ACB0 // 不一样
Base::vptr[0](Func1函数) -> 0x161410C3 // 不一样
Base::vptr[1](Func2函数) -> 0x1614148D // 一样

Derive::vptr -> 0x1614AC18 // 不一样
Derive::vptr[0](Func1函数) -> 0x16141488 // 不一样
Derive::vptr[1](Func2函数) -> 0x1614148D // 一样
```

### 归纳

**虚函数表指针（`vptr`）的创建**：

- 只要一个类定义了至少一个虚函数（或继承了含虚函数的基类），编译器会隐式在该类中插入一个 `vptr`（虚函数表指针）
- 每个类（基类/派生类）有自己独立的 `vtable`（虚函数表），但派生类的 `vtable` 是基于基类 `vtable` 扩展或修改的。
- 无论基类（或派生类）创建多少个实例对象，其 `vptr` 的值都是相同，都指向同一个虚函数表。

**基类与派生类的 `vptr` 指向**：

- 派生类的 `vptr` 不会继承基类的 `vptr`，而是派生类会生成自己的 `vptr`，指向自己的 `vtable`
- 派生类的 `vtable` 是基类 `vtable` 的扩展或修改版本，而非直接共享同一张表

**虚函数表（`vtable`）的内容**：

- 派生类的 `vtable` 不会完全继承基类 `vtable` 的内容，而是会：
  1. 拷贝基类 `vtable` 的初始条目（未被重写的虚函数地址保持不变）
  2. 替换重写的虚函数地址为派生类的实现
  3. 追加派生类新增的虚函数地址到末尾

**虚函数表的初始化时机**：

- 基类的 `vtable` 在编译时直接生成，存放在只读数据段中。
- 派生类的 `vtable` 在编译时基于基类 `vtable` 独立生成的，存放在只读数据段中。
  - 复制基类的虚函数地址（未重写时）
  - 替换为重写后的派生类函数地址（重写时）。
  - 追加新增的虚函数地址。
- 虚函数表中的虚函数是正常存放在代码段中的。


**对象如何关联正确的 `vtable`？**

- 当创建对象时，对象的 `vptr` 在构造函数中被初始化为 **指向当前类** 的 `vtable`。

在继承体系中，派生类对象的内存布局通常如下（以单继承为例）：

```
| 派生类对象的起始地址   |
├─────────────────────┤
│ 派生类的 vptr        │ -→ 指向派生类的 vtable
├─────────────────────┤
│ 基类成员变量          │
├─────────────────────┤
│ 派生类新增成员变量     │
└─────────────────────┘
```



