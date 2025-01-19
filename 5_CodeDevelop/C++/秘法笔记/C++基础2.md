# 类与对象的概念

## C++中类的定义

C++中用类来描述对象，类是对现实世界中相似事物的抽象。类的定义分为两个部分：数据（相当于属性）和对数据的操作（相当于行为）。请看 [[访谈类与对象]]

从程序设计的观点来说，类就是数据类型，是用户定义的数据类型，对象可以看成某个类的实例（某个类的变量）。所以说 **类是对象的封装，对象是类的实例**。

C++中用关键字 class 来定义一个类，其基本形式如下：

```c++
class 类名 {
public:
//公有数据成员和成员函数
protected:
//保护数据成员和成员函数
private:
//私有数据成员和成员函数
}; // 千万不要忘了这个分号
```

class 内部可以拥有的是数据成员(属性)和成员函数(行为)，他们可以分别用三个不同的关键字进行修饰，public、protected、private。
- public 进行修饰的成员表示的是该类可以提供的接口、功能、或者服务；
- protected 进行修饰的成员，其访问权限是开放给其子类；
- private 进行修饰的成员是不可以在类之外进行访问的，只能在类内部访问，可以说封装性就是由该关键字来体现。

## class 与 struct 的区别

在 C++中，与 C 相比，struct 的功能已经进行了扩展。class 能做的事儿，struct 一样能做，他们之间唯一的区别，就是默认访问权限不同。class 的默认访问权限是 private，struct 的默认访问权限是 public



# 对象的创建与销毁

## 构造与析构

**构造函数**：主要作用就是 **用于创建对象时为对象的成员进行初始化**

**析构函数**：主要作用就是 **用于在对象销毁前，执行一些清理工作**

构造函数和解析函数：如果没有手动提供，编译器会提供一个默认的；默认的构造函数和析构函数中，什么都不做，编译器会强制执行。

**构造函数和析构函数要写到公共权限下**

构造函数和析构函数由编译器自动调用一次，无需手动调用。

如果没有提供 **构造函数**，编译器会自动提供一个无参的空实现的 **构造函数**。

如果没有提供 **析构函数**，编译器会自动提供一个无参的空实现的 **析构函数**



## 构造函数的使用

### 构造函数说明

主要作用：**在创建对象时为对象的成员进行初始化。**

有以下特点：
- 构造函数名与类名相同
- 没有返回值，也不用 void
- 允许多个参数，允许重载

构造函数：由编译器自动调用一次，无需手动调用；如果没有手动提供，编译器会提供一个默认的构造函数（**无参空实现的构造函数**）。
在默认的构造函数中，什么都不做，但编译器就是会强制执行该构造函数。

构造函数要写到公共权限（public）下。

```c++
#include<iostream>
#include<string>
  
class Person {
public:
	std::string _name;
	int _age;

	Person() {     //无参数的构造函数
		_name = "";
		_age = 0;
		std::cout << "Person() explicit execute" << std::endl;
	}
};

int main() {
	Person p1;
	p1._name = "Ellen";  //若注释掉，则会执行构造函数的初始值
	p1._age = 25;        //若注释掉，则会执行构造函数的初始值

	std::cout << p1._name << std::endl;
	std::cout << p1._age << std::endl;

	return 0;
}
```


### 构造函数的分类

以参数为分类目标，构造函数可分为：

- 无参构造(默认构造)    
- 有参构造

以类型为分类目标，构造函数可分为：

- 普通构造
- 拷贝构造

注意：**如果我们提供了构造函数，系统将不会提供无参的默认构造函数。并且有参构造函数支持重载。

```c++
Person(){          //无参构造
    _age=0;
    std::cout << "Person() explicit execute" << std::endl;
}

Person(int age){   //有参构造，即对无参构造的方法重载
    _age = age;
    std::cout << "Person(int) explicit execute" << std::endl;
}

// 有参构造的重载
Person(const Person& p){ //&p引用传来的对象，const限定无法通过引用更改底层对象，限定只读状态
    _age = p._age;
    std::cout << "Person(const Person&) explicit execute" << std::endl;
}

```



### 构造函数的显式调用

显示调用：即在创建对象的同时，初始化对象。

```c++
class Person {
public:
	std::string _name;
	int _age;

	Person() {     //无参构造函数
		_name = "";
		_age = 0;
		std::cout << "Person() explicit execute" << std::endl;
	}
    Person(string name, int age) {     //有参构造函数
		_name = name;
		_age = age;
		std::cout << "Person(string, int) explicit execute" << std::endl;
	}
};
-------------------------------------------------------------------------------------------------
//显示调用：即在创建对象的同时，初始化对象
Person p = Person();  //无参显式构造
Person p1 = Person("Ellen",20);  //有参显式构造
```



### 构造函数的隐式调用

在声明的时候便完成对象的初始化。

```c++
Person p2; //隐式调用无参构造   隐式调用无参构造函数不能加上括号，因为编译器会默认为是函数声明
Person p3("Bob",20); //显式调用有参构造

std::cout << p3._name << std::endl;
std::cout << p3._age << std::endl;
```


### 拷贝构造的显式与隐式调用

拷贝函数：使用已经创建初始化好的对象去初始化一个新的对象。

```C++
class Person {
public:
	std::string _name;
	int _age;

	Person() {     //无参构造函数
		_name = "";
		_age = 0;
		std::cout << "Person() explicit execute" << std::endl;
	}
    Person(string name, int age) {     //有参构造函数
		_name = name;
		_age = age;
		std::cout << "Person(string, int) explicit execute" << std::endl;
	}
    // 拷贝构造函数
    Person(const Person &p) {
		_name = p._name;
		_age = p._age;
		std::cout << "Person(const Person &) explicit execute" << std::endl;
	}
};
-----------------------------------------------
Person p;
Person p4(p);  //拷贝构造的隐式调用，用p里面的数据去构造p4
Person p5 = Person(p);//拷贝构造函数的显式调用
```



### 匿名对象

```c++
class Person {
public:
	string _name;
	int _age;
	
    Person();//本行执行完了之后立即释放，可以使用析构函数配合检查
    
};
--------------------------------
Person p = Person();	//第一种定义，编译通过，初始化一个匿名对象赋值给p
Person(p);				//第二种定义，编译不通过，编译器认为Person p; 不能利用拷贝构造函数去初始化一个匿名对象
```



### 隐式转换法

```c++
Person p6 = 10;		//编译器会将10(int)数据转换成 Person类型，不推荐这样用！

//利用隐式转换法调用拷贝构造函数
Person p7 = p1;		//转换Person p7 = Person(p1);

//初始化列表，很少用
Person p8 = {};		//调用无参的构造函数

Person p9 {"p9",20};		//调用有参的构造函数
Person p10 =  {"p10",20};	//调用有参的构造函数
```



### 构造函数的调用规则

构造函数的调用规则

默认情况下，C++编译器至少会为我们提供类的 3 个函数

1. 默认构造函数（无参，函数体为空）
2. 默认析构函数（无参，函数体为空）
3. 默认拷贝构造函数，对类中非静态成员属性进行简单的值拷贝

**如果用户定义了普通的构造（非拷贝构造），C++不会提供默认无参构造；**

**但如果用户定义了拷贝构造函数，C++不会提供任何默认构造函数。** 一般我们会手动定义构造函数和拷贝构造函数



## 初始化列表

初始化列表(一种语法，用来对成员数据进行初始化)，**一般是用来对 const 修饰的成员进行初始化。不然用常规的构造函数 const 修饰的成员变量报错。**

对于引用类型的成员，也必须使用初始化列表的语法进行初始化。

**初始化列表是在定义构造函数时初始化。**

固定参数写法:

```c++
class Person{
	Person() :_a(10), _b(20), _c(30) {} // 固定参数
	
};
```

有参灵活写法：

```c++
class Person{
	Person(int a, int b, int c) :_(a), _b(b), _c(c) {}
	
};

class Person {
    // 默认参数用法
    Person(int a = 0, int b = 0, int c = 0)
    :_(a), _b(b), _c(c){}
};
```

注意：数据成员的初始化的顺序与其在初始化列表中的顺序没有关系，只与数据成员被声明的顺序有关。

```c++
Test(int value)//输入10
: _B(value), _A(m_B){}
-------------------------------
//正常来说，B初始化后，A被B初始化，A与B的值应该是一样的，但实际打印如下
m_A:32766	m_B:10
m_A:32765	m_B:10
//由于A先声明，先初始化A，但A没有初始化是随机值，B随后初始化，所以A与B不一样
```



## 析构函数

### 析构函数说明

构造函数在创建对象时被系统自动调用，而 **析构函数(Destructor)在对象被销毁时被自动调用**。

析构函数在对象销毁时自动调用，用以执行一些清理任务，如释放成员函数中动态申请的内存等。如果程序员没有显式的定义它，系统也会提供一个默认的析构函数。

析构函数的作用：完成数据成员的清理操作。
- 析构函数名和类名相同 前面加上 `~`
- 没有返回值，不用写 void
- 不允许有多个参数，是无参的，不支持重载，具有唯一性。
- 当对象在离开作用域被销毁的时候，析构函数会被自动调用。

```c++
class Person(){
    //....
    ~Person() {
		std::cout << "~Person() execute" << std::endl;
	}
}
-------------------------------------------------------------------------
int main() {
	Person p1;
	//p1._name = "hello";
	//p1._age = 25;
	std::cout << p1._name << std::endl;
	std::cout << p1._age << std::endl;

    //显式调用析构函数，但不建议显式调用
    Person p2;		
    p2.~Person();		//显式调用析构函数
    Person().~Person()	//匿名函数显式调用析构函数

    system("pause"); //让我的程序先运行到这里，暂时不销毁对象；按任意键，程序继续运行
    
	return 0;
}
```



### 析构函数的调用时机

1. 栈对象（局部），程序流程到达该对象的定义处就调用构造函数，在程序离开栈对象的作用域时自动调用析构函数。
2. 全局/静态对象，当程序流程第一次到达该对象定义处调用构造函数，在整个程序结束时调用析构函数。
3. **堆对象**，每当创建该对象时调用构造函数，必须要 **手动的执行 delete 操作**，才能调用析构函数。



### 栈对象销毁

现举一个例子：

```c++
class Person {
public:
    std::string _Name;
    int _Age;
    //有参构造函数
    Person(std::string name, int age):_Name(name),_Age(age) {}
    //析构函数
    ~Person() {
        std::cout << "~Person() execute" << std::endl;
    }
    //打印函数
    void print(){
        std::cout << _Name << " : " << _Age << std::endl;
    }

};
void test() {
    Person p1("Lisa", 21);
    p1.print();
}


int main(void) {
    test();	// 栈对象在离开test作用域时销毁调用析构函数
    
    std::cout << "--------- end ----------" << std::endl;
    return 0;
}
```

运行结果如下：

```shell
Lisa : 21
~Person() execute
--------- end ----------
```



### 全局静态对象销毁

全局静态对象的生存周期是程序运行周期，程序一旦运行结束，全局静态对象便会销毁。

```c++
void test() {
    static Person p2("Smith", 22);	//添加static，成为全局静态对象
    p2.print();
}

int main(void) {
    test();
    std::cout << "--------- end ----------" << std::endl;
    return 0;
}
```

运行结果如下：p2 是在程序执行结束后，自动调用析构函数，所以是在 end 后打印的。

```shell
Smith : 22
--------- end ----------
~Person() execute
```


### 堆对象销毁

**堆对象**，每当创建该对象时调用构造函数，必须要 **手动的执行 delete 操作**，才能调用析构函数。

```c++
void test() {
    //堆对象
    Person *p3 = new Person("Peter",23);
    p3->print();

    delete p3;	//释放堆空间
    p3 = nullptr;	//指针指向空
}


int main(void) {
    test();
    std::cout << "--------- end ----------" << std::endl;
    return 0;
}
```

运行结果如下：

```shell
Peter : 23
~Person() execute
--------- end ----------
```



## 拷贝构造函数

### 拷贝构造的使用

拷贝构造函数是构造函数的一种特殊方式，对象的创建，就必然需要调用构造函数，而这里会调用的就是复制构造函数，又称为拷贝构造函数。

如果类中没有显式定义拷贝构造函数时，编译器会自动提供一个缺省的拷贝构造函数。其原型是：`类名::类名(const 类名 &);`

```c++
class Person {
public:
    //有参构造函数
    Person(std::string name, int age):_Name(name),_Age(age) {}

    //拷贝构造函数
    Person(const Person &rhs) :_Name(rhs._Name) ,_Age(rhs._Age) {
        std::cout << "Execute Copy_Call_Function " << std::endl;
    }
    
    void print(){
        std::cout << _Name << " : " << _Age << std::endl;
    }
private:
    std::string _Name;
    int _Age;
};
void test() {
    Person p1 = Person("Ellen",21);
    Person p2 = Person(p1);	//执行拷贝调用函数，将p1的数据拷贝到p2
    //Person p2 = p1; 也适用
    p2.print();
    
}

int main(void) {
    test();
    std::cout << "--------- end ----------" << std::endl;
    return 0;
}
```

运行结果如下：

```shell
Execute Copy_Call_Function
Ellen : 21
--------- end ----------
```

### 浅拷贝与深拷贝

上例子只是在栈区内进行对象的数据拷贝，对象一旦脱离作用域就会被销毁。所以当在必要时我们要对对象数据进行保存或需要申请堆内存空间保存。

但是在堆空间进行数据拷贝时分为两种：浅拷贝与深拷贝。

**1、浅拷贝（假拷贝）**：[[秘法笔记#浅拷贝|浅拷贝-示例代码]]

运行结果如下：

```shell
Jordon : 10
p3 addr: 000000C0B55EF778, name addr: 0000021FE7FD0680
Execute Copy_Call_Function
Jordon : 10
p4 addr: 000000C0B55EF7A8, name addr: 0000021FE7FD0680
# Q：p3与p4的name地址为什么会相同？
```

A：p3 与 p4 的 name 都指向了堆内存空间的同一块地址，浅拷贝并没有真正的拷贝，只是将对象的初始化数据 **指向** 已经初始化对象的数据区域。

Tips：若对 p3 与 p4 进行销毁时，各自会调用析构函数，那对于指向相同地址的堆空间数据会进行两次的释放操作，引起野指针报错。

```c++
//补充析构函数
class Person(){
	~Person(){
		delete _Name;
		_Name = nullptr;
	}
}
```

运行结果如下：

```shell
Jordon : 10
p3 addr: 0x7ffcf379a490, name addr: 0x56140a1c8eb0
Execute Copy_Call_Function
Jordon : 10
p4 addr: 0x7ffcf379a480, name addr: 0x56140a1c8eb0
free(): double free detected in tcache 2
Aborted

```



**2、深拷贝（真拷贝）**：[[秘法笔记#深拷贝|深拷贝-示例代码]] 

运行结果如下：

```shell
Jordon : 10
p3 addr: 0000004E68AFF768, name addr: 0000015C564E2210
Execute Copy_Call_Function
Jordon : 10
p4 addr: 0000004E68AFF798, name addr: 0000015C564E2170
--------- end ----------
# 实例对象各自申请了堆空间的一块内存，互不干扰，是真正的数据拷贝
```



### 拷贝构造的调用时机

1. 当用一个已存在的对象初始化另一个新对象时，会调用拷贝构造。

   ```c++
   //p1已经初始化
   Person p2 = p1;
   Person p2 = Person(p1)
   ```
   
2. 当实参和形参都是对象，进行 **实参与形参的结合时，会调用拷贝构造**。

   ```c++
   //p1已经初始化
   void func(Person p){	// Person p = p1
       p.print();
   }
   func(p1);
   ```
   
3. 当函数的返回值是对象，函数调用完成返回时，会调用拷贝构造函数。(**优化选项-fno-elide-constructors**)

   ```c++
   Person func(){
       Person p1("Ellen"，21)；
       return p1;
   }
   Person p2 = func();
   ```

   注意：返回值是先拷贝给临时对象（匿名对象）再拷贝到目标对象。临时或匿名对象生命周期较短且创建与销毁都在本行，只是起到了桥梁作用。



### 拷贝构造的参数问题

左值：可以进行取地址的称为左值，左值是实体。

右值：不能进行取地址的称为右值，右值不是实体。右值包括：临时对象、匿名对象、临时变量、匿名变量、字面值常量（10）

**Q1：拷贝构造参数中的引用符号能不能去掉？**

A1：引用符号不能去掉。如果去掉引用符号，那在进行形参与实参结合时，会调用拷贝构造。而拷贝构造现在没有引用符号，那么就是用已存在的对象去初始化一个新创建的对象，也就是满足拷贝构造的调用时机 1。拷贝构造里调用拷贝构造，是会继续无限的调用拷贝构造。而函数的参数是会入栈的，且栈是有大小的，这样会将栈压垮，导致栈溢出。

**Q2：拷贝构造函数参数中的 const 可以去掉吗？**

A2：不能去掉，当执行拷贝构造的时候，如果传递进来的参数是右值的时候，会出现非 const 左值引用不能绑定到右值（注意：没有 const 的时候，传递的参数是左值，是没有问题）。编译器会自动生成拷贝构造函数。



# this 指针

在 C++ 中，每一个对象都能通过 **this** 指针来访问自己的地址。**this** 指针是所有成员函数的隐含参数。因此，在成员函数内部，它可以用来指向调用对象。

this 指针指向用来调用 **成员函数的对象**（指向类实例化对象 p1，成员变量属性）, 谁调用方法，this 指针就指向谁。

this 被作为隐式参数传递给方法，每一个 **非静态** 的成员方法中都有 this 指针。this 存在于每一个非静态的成员函数的第一个参数的位置。

特点：this 是一个指针常量 `*const`，用来记录对象本身的地址。

this 指针的作用：

1. 当形参和成员变量同名时，可以用 this 指针来区分。
2. 在类的 **非静态的成员方法** 中返回对象本身，可以使用 `return *this`

```c++
class Person {
public:
	Person() {} //无参构造函数
	Person(string name, int age) { //有参构造函数
		name = name;
		age = age;
	}
public:
    string name;
	int age;
};

int main() {
	Person p1("张三",20);
	cout << "姓名 " << p1._name << endl;
	cout << "年龄 " << p1._age << endl;

	return 0;
}
```

运行结果如下：
```shell
姓名
年龄 -858993460
# 问题原因：有参构造中的局部变量中的name, age 与成员变量重名了
```


两种解决办法：

1. 成员变量与局部变量命名不同的名字
2. 利用 this 指向成员变量

```c++
# 第二种解决方法
class Person {
public:
	Person() {}
	Person(string name, int age) {
		this->name = name;  //添加this指针，指向对象的name属性
		this->age = age;	//添加this指针，指向对象的age属性
	}
public:
	string name;
	int age;
};

int main() {
	Person p1("张三",20);
	cout << "姓名 " << p1.name << endl;
	cout << "年龄 " << p1.age << endl;

	return 0;
}
```

Tips：**谁调用方法，this 指针就指向谁**！

```c++
// 返回值是引用
Person &getOlder(Person &p) {
	if (age > p.age) {
		cout << p.age << "要比" << age << "小" << endl;
		return *this;
	} else {
    	return p;
	}	
}
-------------------------------------------------------------------
p2.getOlder(p1);
```



# 赋值运算符函数

当 `p2 = p1` 时，默认情况下，编译器会自动生成赋值运算符函数

## 函数形态

```c++
Person p2 = p1;
//赋值运算符函数形态
-------------------------------------------
Person &operator=(const Person &rhs){
	this->_Name = rhs.name;
	_Age = rhs.name;
    return *this;
}
```

## 编写赋值运算符函数

对象给对象赋值的时候，注意是深拷贝；还要考虑自己给自己赋值的情况。

```c++
class Person{
    char *_Name;
    int _Age;
}
//Person p2 = p1; // 初始化
p2 = p1;	// 赋值
---------------------------------------------------------
Person &Person::operator=(const Person &rhs){
    //if判断的是否是自己赋值给自己，this是个指针，利用指针判断
    //1、自复制，防止自己赋值给自己
	if(this != &rhs){	//等价于 *this != rhs
        delete [] _Name;	//2、释放左操作数，解决内存泄漏
        _Name = nullptr;
        //3、深拷贝，防止内存越界
        _Name = new char[strlen(rhs._Name)+1]();
        strcpy(_Name,rhs._Name);
        _Age = rhs.age;
    }
    return *this	//4、返回 *this
}
```

## 参数问题

**Q1：赋值运算符函数参数中的引用符号能去掉吗？**

A1：不能去掉，否则在形参与实参结合的时候会调用拷贝构造函数，多执行一次函数调用，就损失效率（注意：没有栈溢出）。至于为什么没有栈溢出，那是因为拷贝构造里的参数可带着引用符号呢。

**Q2：赋值运算符函数参数中的 const 能去掉吗？**

A2：如果传递的是右值，那么 **非 const 左值引用** 不能绑定到右值，与之前研究的拷贝构造函数参数中的 const 不能去掉的原因一致，对于右值传递会有问题。

**Q3：赋值运算符函数的返回类型中的引用可以去掉吗？**

A3：不能去掉，如果去掉，那么赋值运算符函数的返回类型是一个类类型，那么在执行 return 语句的时候，会调用拷贝构造函数，所以会损失效率

**Q4：赋值运算符函数的返回类型可以是空吗**？

A4：不可以，不然无法实现执行 `pt3 = pt2 = pt1`  连等操作。



# 特殊数据成员的初始化

在 C++的类中，有 4 种比较特殊的数据成员，他们分别是常量成员、引用成员、类对象成员和静态成员，他们的初始化与普通数据成员有所不同。

## 常量数据成员

必须要在初始化表达式中进行初始化，并且不能进行赋值。

```c++
class Person{
public:
	Person(string name,int age)
	: _Name(name)	// 常量数据成员的初始化必须要在初始化列表中
	, _Age(age) {}
private:
	const string _Name;// 常量数据成员
	const int _Age;
};
```



## 引用数据成员

引用数据成员必须在初始化列表中进行初始化。

```c++
class Person{
public:
	Person(int age)
	: _Age(age)
	, _Ref(_Age) {}	// 引用数据成员的初始化必须要在初始化列表中
	
private:
	int &_Ref;//引用数据成员
	int _Age;
};
```

注意：**空类也是占用大小的**
! [[C++基础 2-8.png]]


## 类对象数据成员

类对象数据成员的初始化也是在初始化列表中。

```c++
class Line{
public:
	Line(int x1,int y1,int x2,int y2)
	: _pt1(x1,y1)	//类对象数据成员的初始化必须要放在初始化列表中
	, _pt2(x2,y2) {}
private:
	Point _pt1;	//一个point类
	Point _pt2;
};
```



## 静态数据成员

C++允许使用 static（静态存储）修饰数据成员，这样的成员在编译时就被创建并初始化的（与之相比，对象是在运行时被创建的），且其实例只有一个，被所有该类的对象共享。静态数据成员与的静态变量一样，当程序执行时，该成员已经存在，一直到程序结束。任何该类对象都可对其进行访问，静态数据成员存储在全局/静态区，并不占据对象的存储空间。

特点：不管这个类创建了多少个对象，静态成员只有一个拷贝。这个拷贝被所有属于这个类的对象所共享。

静态成员变量:

​	1.必须在类中声明，在类外定义

​	2.静态成员变量不属于某个对象，而是属于类，所有对象共享

​	3.静态成员变量可以通过类名或者对象名来获取。

**静态数据成员不能在初始化列表中进行初始化，否则会报错**。静态数据成员可以不受访问权限的控制。

```c++
class Book{
private:	
	char *_brand;	//8个字节
	float _price;	//4个字节
	static float _totalPrice;	//总价，静态数据成员不占用类的大小
	//静态变量被类的所有对象共享的
};
float Book::_totalPrice = 0;//类外初始化
```

因为静态数据成员不属于类的任何一个对象，所以它们并不是在创建类对象时被定义的。这意味着它们不是由类的的构造函数初始化的。
一般来说，我们不能在类的内部初始化静态数据成员，**必须在类的外部定义和初始化静态数据成员，且不再包含 static 关键字，用类作用域声明**，格式如下:

不然只会默认为全局变量跟类没有关系。

```c++
类型 类名::变量名 = 初始化表达式; //普通变量
类型 类名::对象名(构造参数); //对象变量
```

Q：为什么要放在实现文件里进行初始化呢？

A：如果放在头文件里，静态数据成员会被定义，首先是 testComputer.cc 包含了头文件，其次是 Computer.cc 也会再次定义，这样静态数据成员就出现了重定义问题。
! [[C++基础 2-9.png]]



# 特殊的成员函数

## 静态成员函数

成员函数也可以定义成静态的，静态成员函数的特点:

1、对于静态成员函数而言，其第一个参数的位置没有隐含的 this 

2、静态的成员函数内部不能访问非静态的数据成员与非静态的成员函数，可以访问和调用静态数据成员和静态的成员函数

3、普通的成员函数可以访问静态的数据成员与静态的成员函数

4、**如果静态的成员函数想访问非静态的数据成员或者成员函数，可以使用函数传参的形式**，或者在静态成员函数体中创建对象

5、静态成员函数可以使用类名与作用域限定符的形式进行调用（独特之处，其他的非静态成员函数不能这么调用）

```c++
#include <iostream>
#include <string>

using namespace std;

class Person {
public:

	static void Func() { // 静态的成员函数
		_C = 201;
		cout << "static Func(): _C = " << _C << endl;
	}

	void func() {      //普通成员函数
		_A = 30;
		_C = 300;
		cout << "func(): _A = " << _A << ", _C =  " << _C << endl;
	}
private:
	int _A;
	static int _C; // 静态成员

};

int Person::_C = 200;	// 静态成员变量在类外定义

int main() {
	Person p1;
	p1.Func(); //对象访问静态成员函数
	Person::Func();//类访问静态成员函数

	p1.func();
	// Person::func(); // 不允许，非静态成员引用必须与特定对象相对

	return 0;
}
```



## const 成员函数

1. 非 const 的对象默认情况调用的是非 const 版本的成员函数；而 const 对象调用是 const 版本的成员函数
2. 非 const 对象也是可以调用 const 版本的成员函数的
3. const 对象是不能调用非 const 版本的成员函数

**第一条结论结果：**

```c++
#include <iostream>

using namespace std;

class Person {
public:
	//两个print函数可以重载，原因是隐含的this不一样
	void print();
	void print() const;
};
void Person::print(/* Person * const this */) {
	cout << "非const函数print调用" << endl;
}
//具有只读特性（对数据成员只能读不能改）
void Person::print(/* const Person * const this */) const {
	cout << "const函数print调用" << endl;
}


int main() {
	Person p1;
	const Person p2;
	p1.print();	//调用非const函数
	p2.print();	//调用const函数

	return 0;
}
```

运行结果如下：
```shell
非const函数print调用
const函数print调用
```



**第二条结论结果：**

```c++
// 只有一个print函数
class Person{
	//两个print函数可以重载，原因是隐含的this不一样
	//void print();
	void print() const;
};

//具有只读特性（对数据成员只能读不能改）
void Person::print(/* const Person * const this */) const{
    cout << "const函数print调用" << endl;
}
-------------------------------------------------------------
Person p3;
p3.print();	//普通对象调用const函数

// 运行结果：const函数print调用
```



**第三条结论结果：报错**

```c++
class Person{
	//两个print函数可以重载，原因是隐含的this不一样
	void print();
	//void print() const;
};
void Person::print(/* Person * const this */){
    cout << "非const函数print调用" << endl;
}

-------------------------------------------------------------
const Person p4;
p4.print();	//调用非const函数
```


! [[C++基础 2-12.png]]



# 对象的组织

使用类创建对象，其机制与基本类型等创建普通变量几乎完全一致，同样可以创建 const 对象、创建指向对象的指针、创建对象数组，还可使用 new 和 delete 等创建动态对象。

## const 对象

类对象也可以声明为 const 对象，一般来说，能作用于 const 对象的成员函数除了它的构造函数和析构函数，便只有它的 const 成员函数了，因为 const 对象只能被创建、撤销以及只读访问，改写是不允许的。

```c++
const Point pt(1, 2); 
pt.print(); //const对象只能调用const成员函数
```



## 指向对象的指针

对象占据一定的内存空间，和普通变量一致， C++ 程序中采用如下形式声明指向对象的指针：

```c++
类名 *指针名 [=初始化表达式];
```

初始化表达式是可选的，既可以通过取地址（&对象名）给指针初始化，也可以通过申请动态内存给指针初始化，或者干脆不初始化（比如置为 nullptr），在程序中再对该指针赋值。指针中存储的是对象所占内存空间的首地址。针对上述定义，则下列形式都是合法的：

```c++
Point pt(1, 2);
Point *p1 = nullptr;
Point *p2 = &pt;
Point *p3 = new Point(3, 4);
Point *p4 = new Point[5];
p3->print();//合法
(*p3).print();//合法
```



## 对象数组

对象数组和标准类型数组的使用方法并没有什么不同，也有声明、初始化和使用 3 个步骤。

```
类名 数组名[对象个数];
```

这种格式会自动调用默认构造函数或所有参数都是缺省值的构造函数。

```c++
Point pt1(3,4);
Point pts[2] = {Point(1, 2), pt1};
Point pts[2] = {Point(1, 2), Point(3, 4)};
Point pts[] = {Point(1, 2), Point(3, 4)};
Point pts[5] = {Point(1, 2), Point(3, 4)};
//或者
Point pts[2] = {{1, 2}, {3, 4}};//这里需要注意，除了去掉Point，还换成了大括号
Point pts[] = {{1, 2}, {3, 4}};
Point pts[5] = {{1, 2}, {3, 4}};
```

Tips：逗号表达式： `a = b, c, d, e`  结果是 a = e，所以不应使用 小括号 `Point pts[2] = {(1, 2), (3, 4)};` 会导致错误结果。小括号的时候注意逗号表达式。



## 堆对象

可以用 new 和 delete 表达式为对象分配动态存储区，在介绍拷贝构造中已经介绍了为类内的指针成员分配动态内存的 [[秘法笔记#深拷贝|深拷贝相关范例]]，这里主要讨论如何为对象和对象数组动态分配内存。如：

```c++
void test()
{
	Point *pt1 = new Point(11, 12);
	delete pt1;
	pt1 = nullptr;
    
	Point * pt2 = new Point[5]();//注意
	delete [] pt2;//可以试试使用free去进行释放会发生什么？为什么会发生这个问题？
}
```

**注意**: 使用 new 为对象数组分配动态空间时，不能显式调用对象的构造函数。因此，对象要么没有定义任何形式的构造函数（由编译器缺省提供），要么显式定义了一个（且只能有一个）所有参数都有缺省值的构造函数。



# 单例模式

单例模式是 23 种 GOF 模式中最简单的设计模式之一，属于创建型模式，它提供了一种创建对象的方式，确保只有单个对象被创建。这个设计模式主要目的是想在整个系统中只能出现类的一个实例，即一个类只有一个对象。

单例的好处：防止创建过多的对象，节省内存。单例：单个实例（对象），创建一个对象，实例化一个对象，创建一个类的实例。

思路：
1. 创建的对象肯定是要入内存的，也就是用户态的那块空间
2. 不能在类外面创建对象，所以就在类中进行创建
3. 为了让类外的对象创建报错，将构造函数设置为私有的

设计步骤：
1. 将构造函数私有化
2. 在类中定义一个静态的指向本类型的指针变量
3. 定义一个返回值为类指针的静态成员函数

使用场景：全局唯一的对象，网页库、日志记录器

示例代码请看：[[秘法笔记#单例模式|单例模式]]


# 内存对齐

## 内存对齐解释

对齐规则是按照成员的声明顺序，依次安排内存，其偏移量为成员大小的整数倍，0 看做任何成员的整数倍，最后 **结构体的大小为最大成员的整数倍**


## 为什么要内存对齐？

1.平台原因(移植原因)：不是所有的硬件平台都能访问任意地址上的任意数据的；某些硬件平台只能在某些地址处取某些特定类型的数据，否则抛出硬件异常。

2.性能原因：数据结构(尤其是栈)应该尽可能地在自然边界上对齐。原因在于，为了访问未对齐的内存，处理器需要作两次内存访问；而对齐的内存访问仅需要一次访问。

解释二

原因有这么几点：

1、CPU 对内存的读取不是连续的，而是分成块读取的，块的大小只能是 1、2、4、8、16 ... 字节；

2、当读取操作的数据未对齐，则需要两次总线周期来访问内存，因此性能会大打折扣；

3、某些硬件平台只能从规定的相对地址处读取特定类型的数据，否则会产生异常。

## 内存对齐的三大规则

### 数据成员对齐规则

结构(struct) (或联合(union)) 的数据成员，第一个数据成员放在 offset(偏移)为 0 的地方，以后每个数据成员的对齐按照 `#pragma pack` 指定的数值和这个数据成员自身长度中，比较小的那个进行。

### 结构(或联合)的整体对齐规则

在数据成员完成各自对齐之后，结构(或联合)本身也要进行对齐，对齐将按照 `#pragma pack` 指定的数值和结构(或联合)最大数据成员长度中，比较小的那个进行。

### 结构体作为成员

如果一个结构里有某些结构体成员，则 **内部结构体成员** 要从成员最大元素大小的整数倍和 `#pragma pack` 指定的数值中最小的一个的整数倍的地址开始存储。

`#pragma pack(n) 对齐系数`



# new 和 delete 工作机制

## new 表达式的三个工作步骤

1. 执行 **operator new 库函数**，申请原始的未初始化的堆空间
2. 在该空间上构建对象
3. 返回执行新空间的指针

## delete 表达式的两个工作步骤

1. 调用析构函数，将对象中的 **数据成员所申请的资源进行回收**
2. 调用 **operator delete 库函数**，回收对象本身所占用的空间。


## operator new 与 delete 的重载

```c++
//operator new库函数
void *operator new(size_t);
void *operator new[](size_t);

//operator delete库函数
void operator delete(void *);
void operator delete[](void *);
```



## 问题 1：对象的销毁与析构函数的执行是等价的吗?

A：对于堆对象而言, 对象的销毁会执行析构函数, 但是析构函数的执行只是对象销毁中的一个步骤，如果是栈对象, 那么对象的销毁与析构函数的执行就是等价的。请看 [[秘法笔记#问题 1|示例代码]] 

运行结构如下：

```c++
void *operator new(size_t )
Person(const char* name,int age)
name = Ellen
age = 24
~Person()
void operator delete(void *)
```

可见在调用 delete 之前，先调用析构函数，而调用析构函数则是将对象中的 **数据成员所申请的资源进行回收**，随后 delete 销毁对象。所以对于堆对象而言，析构函数的执行与 delete 是不等价的，析构函数的执行是 delete 的一个步骤过程。

注意：不能将 operator new 与 operator delete 在类外定义，否则修改的是全局的 new 与 delete 的表达式，那在给数据成员初始化与回收数据成员时的表达式会发生变化，导致打印会各自多显示一次。

```c++
void *operator new(size_t )
void *operator new(size_t )
Person(const char* name,int age)
name = Ellen
age = 24
~Person()
void operator delete(void *)
void operator delete(void *)
```



## 要求一个类只能创建栈对象

如果要求一个类只能创建栈对象，其意思是 **在创建出栈对象的同时，不能创建堆对象**。以 Person 类型 而言，需求：只能创建栈对象, 不能创建堆对象

解决方案：**将 operator new/delete 库函数设置为私有的** : 这样在进行 new 操作的时候便无法调用 new 函数 去创建堆对象。

Q: 创建栈对象的条件有哪些？	A: 构造函数与析构函数都需要设置为 public

```c++
#include <string.h>
#include <iostream>
using std::cout;
using std::endl;


class Person{
public:
    Person(const char* name,int age):_Name(new char[strlen(name)+1]()),_Age(age){
        cout << "Person(const char* name,int age)" << endl;
        strcpy(_Name, name);
    }
    void print() const{
        if(_Name){
            cout << "name = " << _Name << endl
                 << "age = " << _Age << endl;
        }
    }
    ~Person(){
        cout << "~Person()" <<endl;
        if(_Name){
            delete [] _Name;
            _Name = nullptr;
        }
    }
private:
    static void *operator new(size_t sz){	//应该被所以对象共享使用加static
        cout << "void *operator new(size_t )" << endl;
        //申请的原始的未初始化的空间
        void *pret = malloc(sz);

        return pret;
    }

    static void operator delete(void *ptr){	//应该被所以对象共享使用加static
        cout << "void operator delete(void *)" <<endl;
        free(ptr);
    }
private:
    char *_Name;
    int _Age;
};

int main(void){
    Person p1("Ellen",24);
    Person *p2 = new Person("埃尔文",24);

    return 0;
}
```



## 要求一个类只能创建堆对象

如果要求一个类只能创建堆对象，其意思是在创建出堆对象的同时，不能创建栈对象。需求：只能创建堆对象, 不能创建栈对象

解决方案：**将析构函数设置为私有的**，由于栈对象的创建与堆对象的创建都需要用到构造函数，直接将构造函数置为 private，会导致两个都不可以创建。

但是，堆对象的销毁是会提前调用析构函数，栈对象的销毁与析构函数调用等价，所以将析构函数置为私有后，栈对象就无法创建成功。同时堆对象在进行 delete 操作时会自动调用析构函数（无论公有私有），但由于析构函数私有化了，无法调用，所以需要在类里专门写一个函数调用 delete，在类内便可访问析构函数。

```c++
#include <string.h>
#include <iostream>

using std::cout;
using std::endl;

//需求：只能创建堆对象,不能创建栈对象
//解决方案：将析构函数设置为私有的

class Student {
public:
    Student(int id, const char *name)
    : _id(id)
    , _name(new char[strlen(name) + 1]()) {
        cout << "Student(int, const char *)" << endl;
        strcpy(_name, name);
    }

    static void *operator new(size_t sz) {
        cout << "void *operator new(size_t )" << endl;
        //申请的原始的未初始化的空间
        void *pret = malloc(sz);
        return pret;
    }

    static void operator delete(void *ptr){
        cout << "void operator delete(void *)" <<endl;
        free(ptr);
    }

    void print() const {
        if(_name) {
            cout << "id = " << _id << endl
                 << "name = " << _name << endl;
        }
    }

    void destroy() {
        /* this->~Student();//显示调用析构函数 */
        delete this;
    }
private:
    ~Student() {
        cout << "~Student()" <<endl;
        /* delete this;//error */
        if(_name) {
            delete [] _name;
            _name = nullptr;
        }
    }
private:
    int _id;
    char *_name;
};

int main(int argc, char **argv) {
    //Q:创建栈对象的条件有哪些？
    //A:构造函数与析构函数都需要设置为public
    /* Student stu(1234, "xiaogang");//error,栈对象 */
    /* stu.print(); */

    Student *pstu = new Student(4201, "xiaohong");//堆对象,ok
    pstu->print();
    pstu->destroy();

    return 0;
}


```



















