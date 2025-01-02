# 概念介绍

unique_ptr 明确表明是**独享所有权**的智能指针，**无法进行复制与赋值**。它保存指向某个对象的指针，当它本身被删除释放的时候，会使用给定的删除器释放它指向的对象。

**具有移动语义，可以作为容器的元素（容器中可以存 unique_ptr 的类型）**


# 使用说明

```c++
void test()
{
    unique_ptr<int> up(new int(10));
    cout << "*up = " << *up << endl;
    cout << "up.get() = " << up.get() << endl;//获取托管的指针的值
	
    // error,独享资源的所有权,不能进行拷贝复制
    // unique_ptr<int> up2(up);
    
    unique_ptr<int> up3(new int(10));
    // error,不能进行赋值操作
    // up3 = up;
    //独享所有权的智能指针，对托管的空间独立拥有。在语法层面就将auto_ptr的缺陷摒弃掉了，具有对象语义
    
    /*--------------------------------------------------------------------*/
    
    //通过移动语义move转移up的所有权，初始化智能指针up4
    unique_ptr<int> up4(std::move(up));
    cout << "*up4 = " << *up4 << endl;
    cout << "up4.get() = " << up4.get() << endl;
    
    unique_ptr<string> up5(new string("hello"));
    vector<unique_ptr<string>> vect;
    //不能传左值，但是可以传右值
    vect.push_back(std::move(up5));
    
    //或者直接构建右值，显式调用unique_ptr的构造函数
    vect.push_back(unique_ptr<string>(new string("world")));
}
```


# 使用操作

![[20240716233927.png]]


我们虽然不能拷贝或赋值 unique_ptr，但可以通过调用 release 或 reset 将指针的所有权从一个（非 const）unique_ptr 转移给另一个 unique

```c++
unique_ptr<string> up2(up1.release()); // release将p1置为空
unique_ptr<string> up3(new string("Trex"));
// 将所有权从p3转移给p2
up2.reset(up3.release());
```


## reset 使用

reset 接受一个可选的指针参数，令 unique_ptr 重新指向给定的指针。如果 unique_ptr 不为空，它原来指向的对象被释放。

```c++
unique_ptr<string> p2( new string("hello")); // release将p1置为空
unique_ptr<string> p3(new string("Trex"));
// 释放p2指向的对象，并将p3对象的所有权从p3转移给p2
up2.reset(up3.release());
```

## release 使用

调用 release 会切断 unique_ptr 和它原来管理的对象间的联系。release 返回的指针通常被用来初始化另一个智能指针或给另一个智能指针赋值。但注意的是它仅仅是放弃对对象的所有权，并没有负责对资源的释放。因此如果我们不用另一个智能指针来保存 release 返回的指针，我们的程序就要负责资源的释放：

```c++
// error, p2不会释放内存，而是丢失了指针
p2.release();

// 正确，但必须记得delete(p)
auto p = p2.release()
```



# 传递与返回

不能拷贝 unique_ptr 的规则有一个例外：我们可以拷贝或赋值一个将要销毁的 unique_ptr。

例如：函数返回 unique_ptr（返回匿名对象）。

```c++
unique_ptr<int> clone(int p) {
	// 正确
    return unique_ptr<int>(new int(p));
}
```

例如：返回一个局部对象的拷贝（返回局部对象）。

```c++
unique_ptr<int> clone(int p) {
    unique_ptr<int> ret(new int (p));
    // ...
    return ret;
}
```

# 传递删除器

unique_ptr 默认情况下用 delete 释放它所指向的对象。

可以重载一个 unique_ptr 中默认的删除器，但 unique_ptr 管理删除器的方式与 shared_ptr 不同。重载 unique_ptr 中的删除器时必须在尖括号中 unique_ptr 指向类型之后提供删除器类型。在创建或 reset 一个这种 unique_ptr 类型的对象时，必须提供一个指定类型的可调用对象（删除器）。

```c++
// p 指向一个类型为 objT 的对象，并使用一个类型为 delT 的对象释放 objT 对象
// 它会调用一个名为 fcn 的 delT 类型对象
unique_ptr<objT, delT> p (new objT, fcn);
```

具体例子：

```c++
void func(dest &d){
    connection c = connect(&d);
    // 当 p 被销毁时，连接将会关闭
    unique_ptr<connection, decltype(end_connection)*> p (&c, end_connection);
    // 当 func 退出时（即使是由于异常而退出），connection会被正确关闭
}
```

decltype 指明函数指针类型，由于 decltype(end_connection) 返回一个函数类型，所以我们必须添加一个 * 来指出我们正在使用该类型的一个指针。

**函数类型和函数指针类型的区别**：

函数类型和函数指针类型是两种不同的类型。函数类型是指一个函数的特征，包括返回类型和参数类型。而函数指针类型是指指向函数的指针的类型，它由函数类型派生而来。例如，对于函数类型 `int (int, int)`，对应的函数指针类型为 `int (*)(int, int)`。这意味着它是一个指针，指向一个函数，该函数接受两个整数参数并返回一个整数。