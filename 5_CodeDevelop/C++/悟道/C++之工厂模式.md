在 C++中，工厂模式是一种创建型设计模式，它提供了一种在不指定具体类的情况下创建对象的方法。工厂模式主要有三种类型：**简单工厂模式**、**工厂方法模式**和**抽象工厂模式**。每种模式都有其特点和适用场景。

对象与实例的概念区别：

总体来讲，对象就是类的实例，所有的对象都是实例，但并不是所有的实例都是对象。例如，一个关联（UML 关系中的一种）的实例不是一个对象，它只是一个实例、一个连接。我们常见的实例都是类的实例，此时二者没有区别。除了类的实例外的实例都不是对象。

main 函数的代码区是 client 的使用区，在这个区下一定得是站在 client 的角度下去理解思想。
# 简单工厂

特点：通过一个工厂类来封装产品对象的创建过程。客户端向工厂类传递不同的参数，由此创建不同的具体产品对象返回给客户端。而创建不同具体产品的对象则基于类的多态，所有的**具体产品类**都继承**抽象产品类**从而调用共性的行为。

优点：将对象的创建逻辑集中到一个工厂类中，客户端代码只需要与工厂类进行交互，而不需要直接与具体对象进行交互。（客户端只知道传入工厂类的参数，对于如何创建的产品并不关心）

缺点：1、违反了单一职责原则；2、违反了开闭原则；3、违反了依赖倒置原则；

- 若工厂类 createProduct 方法出现问题，将影响到所有具体类的实例化；同时工厂类承载与太多其他类的交互，使得相互之间的代码耦合度较高。
- 对于每个新增的具体产品，都需要修改工厂类的创建逻辑，可能导致工厂类的代码过于臃肿。

![[20240601221647.png]]

简单工厂又称之为静态工厂，之所以有这个静态的说法是因为，创建产品时都通过工厂类调用静态函数来实现。


# 工厂方法

在不指定对象具体类型的情况下创建对象：定义一个用于创建对象的接口，让实现接口的意义来决定实例化哪个类型的对象。FactoryMethod 使一个类的实例化延迟到其子类。

创建一个对象常常需要复杂的过程，所以不适合包含在一个复合对象中。创建对象可能会导致大量的重复代码，可能会需要复合对象访问不到的信息，也可能提供不了足够级别的抽象，还可能并不是复合对象概念的一部分。工厂方法模式通过定义一个单独的创建对象的方法来解决这些问题，由子类实现这个方法来创建具体类别的对象。对象创建中的有些过程包括决定创建哪个对象、管理对象的生命周期，以及管理特定对象的建立和销毁的概念。

**意图**：定义一个用于创建对象的接口，让子类决定实例化哪一个类。FactoryMethod 使一个类的实例化延迟到其子类。

目的：让客户指定

产品类与工厂类都是抽象类。利用多态的特性：生产什么类型的产品前，先创建该类型的工厂。
![[20240604221150.png]]

![[20240602210512.png]]



# 抽象工厂

**意图**：提供一个接口用于创建一系列相关或相互依赖的对象，而无须指定它们具体的类。

**适用性**：在以下情况下使用 AbstractFactory 模式：

- 一个系统要独立于它的产品的创建、组合和表示。
- 一个系统要由多个产品系列中的一个来配置。
- 要强调一系列相关的产品对象的设计以便进行联合使用。
- 提供一个产品类库，但只想显示它们的接口而不是实现。

**模式结构图**：

![[20240603221733.png]]

![[20240603192021.png]]


**参与者**：

- AbstractFactory：声明一个创建抽象产品对象的操作接口
- ConcreteFactory：实现创建具体产品对象的操作。
- AbstractProduct：为一类产品对象声明一个接口。
- ConcreteProduct：定义一个将被相应的具体工厂创建的产品对象，实现 AbstractProduct 接口。
- Client：仅使用由 AbstractFactory 和 AbstractProduct 类声明的接口。

**协作**：

通常在运行时创建一个 ConcreteFactroy 类的实例。这一具体的工厂创建具有特定实现的产品对象。为创建不同的产品对象，客户应使用不同的具体工厂。AbstractFactory 将产品对象的创建延迟到它的ConcreteFactory 子类。

**效果**：

AbstractFactory 模式有以下优点和缺点：

1. 它分离了具体的类： AbstractFactory 模式帮助你控制一个应用创建的对象的类。因为一个工厂封装创建产品对象的责任和过程，它将客户与类的实现分离。客户通过它们的抽象接口操纵实例。产品的类名也在具体工厂的实现中被隔离，即它们不出现在客户代码中。
2. 它使得易于交换产品系列：一个具体工厂类在一个应用中仅出现一次（在它初始化的时候）。这使得改变一个应用的具体工厂变得很容易。只需要改变具体的工厂即可使用不同的产品配置，这是因为一个抽象工厂创建了一个完整的产品系列，所以整个产品系列会立刻改变。比如我们可以将抽象工厂类的指针指向不同的具体工厂，从而实现不同具体工厂的操作改变。
3. 它有利于产品的一致性：**当一个系列中的产品对象被设计成一起工作时，一个应用一次只能使用同一个系列中的对象，这一点很重要。而 AbstractFactory 很容易实现这一点**。
4. 难以支持新种类的产品：难以扩展抽象工厂以生产新种类的产品。这是因为 AbstractFactory 接口确定了可以被创建的产品集合。支持新种类的产品就需要扩展该工厂接口，这将涉及 AbstractFactory 类及其
所有子类的改变（拓展的新接口意味着子类必须重定义新接口而使自身发生改变）。我们会在实现一节讨论这个问题的一个解决办法。

抽象工厂接口中就确定了抽象产品的集合，如果要支持新种类的产品，就必须拓展抽象工厂的产品集合。那就涉及到抽象工厂类的及其子类的改变。

**实现**：

下面是实现 AbstractFactor 模式的一些有用技术：

1. 将工厂作为单例：一个应用中一般每个产品系列只需要一个 ConcreteFactory 的实例。因此工厂通常最好实现为一个单例
2. 创建产品：AbstractFactory 仅声明一个创建产品的接口，真正创建产品是由 ConcreteProduct 子类实现的。最通常的办法是为每一个产品定义一个工厂方法（参见 FactoryMethod）。一个具体的工厂将为每个产品重定义该工厂方法以指定产品。虽然这样的实现很简单，但它却要求每个产品系列都要有一个新的具体工厂子类，即使这些产品系列的差别很小。


工厂模式中有: 工厂方法 (Factory Method) 抽象工厂 (Abstract Factory).
这两个模式区别在于需要创建对象的复杂程度上。