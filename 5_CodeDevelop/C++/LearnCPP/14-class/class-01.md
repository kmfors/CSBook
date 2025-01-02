## 面向对象编程

在 **面向对象编程**（object-oriented programming，通常缩写为 OOP）中，重点是创建包含属性和一组定义良好的行为的程序定义的数据类型。 OOP 中的术语 **对象** 指的是类类型可以实例化的对象。

因为属性和行为不再分离，对象更容易模块化，这使得我们的程序更容易编写和理解，也提供了更高程度的代码可重用性。这些对象还允许我们定义如何与对象交互以及它们如何与其他对象交互，从而提供更直观的数据处理方式。

在传统编程中，**对象** 是一块存储值的内存，拥有名称的 **对象** 则称为 **变量**。然而在面向对象编程中，**对象** 意味着它既是传统编程意义上的对象，又结合了属性和行为。在这些教程中，我们将倾向于使用术语 **对象** 的传统含义，并且在具体指代 **OOP 对象** 时更倾向于使用术语 **类对象**。



## 类的介绍

### 类的不变式问题

也许结构体的最大困难是它们没有提供有效的方法来记录和强制执行 **类的不变式**（class invariant）。在此之前，我们将 **不变式** 定义为“在执行某个组件时必须为真的条件”。

在类类型（包括结构体、类和联合）的上下文中，**类的不变式** 是在对象的整个生命周期中必须为 `true` 的条件，以便使对象保持有效状态。违反 **类的不变式** 的对象被认为处于 **无效状态**，进一步使用该对象可能会导致意外或未定义的行为。

首先考虑以下结构：

```c++
struct Pair
{
    int first {};
    int second {};
};
```

`first` 和 `second` 成员可以独立设置为任何值，因此 `Pair` 结构体没有不变式。

现在考虑以下几乎相同的结构：

```c++
struct Fraction
{
    int numerator { 0 };
    int denominator { 1 };
};
```

我们从数学中知道，分母为 `0` 的分数在数学上是未定义的（因为分数的值是分子除以分母，而除以 `0` 在数学上是未定义的）。因此，我们要确保 `Fraction` 对象的 `denominator` 成员永远不会设置为 `0` 。如果是，则该 `Fraction` 对象处于无效状态，进一步使用该对象可能会导致未定义的行为。

鉴于 `Fraction` 示例的相对简单性，简单地避免创建无效的 `Fraction` 对象应该不会太困难。然而，在使用许多结构、具有许多成员的结构或成员具有复杂关系的结构的更复杂的代码库中，理解哪些值的组合可能违反某些 **类的不变式** 可能不是那么明显。



> [!NOTE]
>
> - 个人认为：猜测类的不变式的出现，本意是想使得类的定义更为具体、更符合实际的应用场景需求，而不是随意的设置。
> - ”式“ 或 “条件” 这个词更能体现出 `invariant` 是一种约束规则的含义。
> - 类的不变式是一个条件，在对象的整个生命周期内都必须满足这个条件，对象才能处于有效状态。

### 复杂类的不变量

`Fraction` 的类的不变式很简单即 `denominator` 成员不能为 `0` 。这在概念上很容易理解，而且也不太难避免。但当结构体的成员必须具有相关值时，类的不变式就变得更具挑战性。

```c++
#include <string>

struct Employee
{
    std::string name { };
    char firstInitial { }; // should always hold first character of `name` (or `0`)
};
```

在上面的结构体中，存储在成员 `firstInitial` 中的字符值应该始终与 `name` 的第一个字符匹配。

当 `Employee` 对象初始化时，用户负责确保维护类不变式。如果 `name` 被赋予了新值，用户还有责任确保 `firstInitial` 也被更新。对于使用 Employee 对象的开发人员来说，这种相关性可能并不明显，即使是这样，他们也可能会忘记这样做。

即使我们编写函数来帮助我们创建和更新 `Employee` 对象（确保 `firstInitial` 始终从 `name` 的第一个字符设置），但我们仍然依赖用户来了解和使用这些函数。

简而言之，依赖对象的用户来维护 **类的不变式** 可能会导致出现问题的代码。

理想情况下，我们希望对我们的类类型进行防御保护，以便对象不能被置于无效状态；或者如果处于无效状态，可以立即发出信号（而不是让未定义的行为在未来的某个随机点发生）。**结构体（作为聚合）只是不具备以优雅的方式解决此问题所需的机制。**

## 成员函数

### 属性和动作的分离

在编程中我们用变量来表示属性，用函数来表示动作。

类类型（包括结构体、类和联合）除了可以拥有成员变量之外，还可以拥有各自的函数！属于类类型的函数称为 **成员函数**。

成员函数必须在类类型定义内部声明，并且可以在类类型定义内部或外部定义。提醒一下，定义也是声明，因此如果我们在类中定义成员函数，那么它就被视为声明。

**成员函数在类类型定义中声明**

非成员函数是在类结构之外的全局命名空间中定义的。默认情况下，它具有外部链接，因此可以从其他源文件调用它。

成员函数是在类结构定义内声明的。因此成员函数被声明为类的一部分，而告知编译器这是一个成员函数。

类类型内部定义的成员函数是隐式内联的，因此如果类类型定义包含在多个代码文件中，它们不会导致违反单一定义规则。成员函数也可以在类定义内部（前向）声明，并在类定义之后定义。

所有（非静态）成员函数都必须使用该类类型的对象来调用。

调用成员函数的对象被 *隐式* 传递给成员函数。因此，调用成员函数的对象通常称为 **隐式对象**。

- 对于非成员函数，我们必须显式地将对象传递给要使用的函数，并且通过该对象显式访问成员。
- 对于成员函数，我们隐式地将对象传递给要使用的函数，并且通过该对象隐式访问成员。

**成员变量和函数可以按任意顺序定义**

> [!WARNING]
>
> 数据成员按照声明的顺序进行初始化。如果数据成员的初始化访问了稍后才声明的另一个数据成员（因此尚未初始化），则初始化将导致未定义的行为。

在 C 语言中，结构体只有数据成员，没有成员函数。

如果您的类类型没有数据成员，最好使用命名空间。

编译器不允许我们在 const 对象上调用非 const 成员函数。

 **const 成员函数** 是保证不会修改对象或调用任何非 const 成员函数（因为它们可能会修改对象）的成员函数。

在 C++ 中，以“m_”前缀开头命名私有数据成员是一种常见约定。这样做有几个重要原因。

这与我们建议对局部静态变量使用“s_”前缀以及对全局变量使用“g_”前缀的原因相同。

没有命名访问函数的通用约定。但是，有一些命名约定比其他命名约定更流行：以“get”和“set”为前缀

```c++
int getDay() const { return m_day; }  // getter
void setDay(int day) { m_day = day; } // setter
```

“set” prefix only: 仅“set”前缀, 强烈建议 setter 使用“set”前缀。 Getter 可以使用“get”前缀或不使用前缀。

Getters 应提供对数据的“只读”访问。因此，最佳实践是它们应该按值（如果制作成员的副本成本低廉）或 const 左值引用（如果制作成员的副本成本昂贵）返回。

返回引用的成员函数应返回与返回的数据成员类型相同的引用，以避免不必要的转换。

右值对象在创建它们的完整表达式结束时被销毁。当右值对象被销毁时，对该右值成员的任何引用都将失效并悬空，并且使用此类引用将产生未定义的行为。因此，对右值对象成员的引用只能在创建右值对象的完整表达式中安全地使用。

```c++
#include <iostream>
#include <string>
#include <string_view>

class Employee
{
	std::string m_name{};

public:
	void setName(std::string_view name) { m_name = name; }
	const std::string& getName() const { return m_name; } //  getter returns by const reference
};

// createEmployee() returns an Employee by value (which means the returned value is an rvalue)
Employee createEmployee(std::string_view name)
{
	Employee e;
	e.setName(name);
	return e;
}

int main()
{
	// Case 1: okay: use returned reference to member of rvalue class object in same expression
	std::cout << createEmployee("Frank").getName();

	// Case 2: bad: save returned reference to member of rvalue class object for use later
	const std::string& ref { createEmployee("Garbo").getName() }; // reference becomes dangling when return value of createEmployee() is destroyed
	std::cout << ref; // undefined behavior

	// Case 3: okay: copy referenced value to local variable for use later
	std::string val { createEmployee("Hans").getName() }; // makes copy of referenced member
	std::cout << val; // okay: val is independent of referenced member

	return 0;
}
```

当调用 `createEmployee()` 时，它将按值返回一个 `Employee` 对象。返回的 `Employee` 对象是一个右值，它将一直存在，直到包含对 `createEmployee()` 调用的完整表达式结束。当该右值对象被销毁时，对该对象成员的任何引用都将变为悬空。

*在使用完整表达式作为初始值设定项后*，完整表达式的求值就会结束。这允许使用相同类型的右值来初始化对象（因为在初始化发生之前右值不会被销毁）。

但是，如果我们想保存一个通过引用返回成员的函数的值以供以后使用，该怎么办？我们可以使用返回的引用来初始化非引用局部变量，而不是使用返回的引用来初始化局部引用变量。

最好使用立即通过引用返回的成员函数的返回值，以避免当隐式对象是右值时出现悬空引用问题。
