# 概念介绍

shared_ptr 是一个**共享所有权的智能指针，允许对象之间进行复制或者赋值**，展示出来的就是值语义。

使用**引用计数**的观点。当对象之间进行复制或者赋值的时候，引用计数会加+1；当最后一个对象销毁的时候（例如局部的 shared_ptr 离开其作用域），引用计数减为 0 的时候，会负责回收托管的空间。

# 使用说明

主要理解这个智能指针的工作原理

```c++
void test()
{
    shared_ptr<int> sp(new int(10));
    cout << "*sp = " << *sp << endl;
    cout << "sp.get() = " << sp.get() << endl;
    cout << "sp.use_count() = " << sp.use_count() << endl; //引用计数

    cout << endl;
    shared_ptr<int> sp2 = sp;//拷贝复制ok
    cout << "*sp = " << *sp << endl;
    cout << "*sp2 = " << *sp2 << endl;
    cout << "sp.get() = " << sp.get() << endl;
    cout << "sp2.get() = " << sp2.get() << endl;
    cout << "sp.use_count() = " << sp.use_count() << endl;
    cout << "sp2.use_count() = " << sp2.use_count() << endl;

    cout << endl;
    shared_ptr<int> sp3(new int(20));
    sp3 = sp;//赋值ok
    cout << "*sp = " << *sp << endl;
    cout << "*sp2 = " << *sp2 << endl;
    cout << "*sp3 = " << *sp3 << endl;
    cout << "sp.get() = " << sp.get() << endl;
    cout << "sp2.get() = " << sp2.get() << endl;
    cout << "sp3.get() = " << sp3.get() << endl;
    cout << "sp.use_count() = " << sp.use_count() << endl;
    cout << "sp2.use_count() = " << sp2.use_count() << endl;
    cout << "sp3.use_count() = " << sp3.use_count() << endl;
}
```

运行结果：

```shell
*sp = 10
sp.get() = 0x56449b513eb0
sp.use_count() = 1

*sp = 10
*sp2 = 10
sp.get() = 0x56449b513eb0
sp2.get() = 0x56449b513eb0
sp.use_count() = 2
sp2.use_count() = 2

*sp = 10
*sp2 = 10
*sp3 = 10
sp.get() = 0x56449b513eb0
sp2.get() = 0x56449b513eb0
sp3.get() = 0x56449b513eb0
sp.use_count() = 3
sp2.use_count() = 3
sp3.use_count() = 3
```

**最安全的分配和使用动态内存的方法是调用一个名为 make_shared 的标准库函数**。此函数在动态内存中分配一个对象并初始化它，返回指向此对象的 shared_ptr。

使用 make_shared 时，必须指定想要创建的对象的类型。

```c++
// 指向一个值为42的int的shared_ptr
shared_ptr<int> p3 = make_shared<int>(42);

// p4指向一个值为"999999"的string
shared_ptr<string> p4 = make_shared<string>(6, '9')

// p5指向一个值初始化的int，即值为0
shared_ptr<int> p5 = make_shared<int>();
```

什么是值初始化？库会根据对象的元素类型创建一个默认的值并赋予该对象。

通常我们会用 auto 定义一个对象来保存 make_shared 的结果

```c++
auto p6 = make_shared<vector<string>>();
```

# 使用操作

![[20240714162836.png]]


![[20240714162850.png]]


# 问题缺陷

**问题：存在循环引用？** 该智能指针在使用的时候，会使得引用计数增加，从而会出现**循环引用**的问题？

两个 shared_ptr 智能指针互指，导致引用计数增加，不能靠对象的销毁使得引用计数变为 0，从而导致内存泄漏。。。

![[20240714215320.png|700]]


循环引用导致的原因是 shared_ptr 是强引用的智能指针，每一次指向都会使得引用计数进行累加。但是单独的靠智能指针这种栈对象的销毁又不能将引用计数减为 0，所以就会出现循环引用而不能完成空间回收的问题。

解决方式：需要有一种弱引用的智能指针，可以执行空间，但是不会将引用计数累加，这样的一种指针叫做 weak_ptr。互相引用的两个 shared_ptr 需要使得其中一个改为 weak_ptr，但不会增加引用计数，可以利用对象的销毁而打破引用计数减为 0 的问题。

---

# 补充说明

shared_ptr 的析构函数会递减它所指向的对象的引用计数。如果引用计数变为 0，shared_ptr 的析构函数就会销毁对象，并释放它占用的内存。

shared_ptr 的构造函数是 explicit 的。因此不能将一个内置指针隐式转换为一个智能指针，必须使用直接初始化形式。

```c++
shared_ptr<int> p1 = new int(1024); // 错误！

shared_ptr<int> p2(new int(1024));  // 正确！
```

一个返回 shared_ptr 的函数不能在其返回语句中隐式转换一个普通指针：

```c++
shared_ptr<int> clone(int p){
    return new int(p); // 错误：隐式转换为shared_ptr<int>
}

// 必须将shared_ptr显式绑定到返回的指针上
shared_ptr<int> clone(int p){
    return shared_ptr<int>(new int(p)); // 正确
}
```

![[20240714222810.png]]