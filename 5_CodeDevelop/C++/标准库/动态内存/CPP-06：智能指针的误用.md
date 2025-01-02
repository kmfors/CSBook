使用异常处理的程序能在异常发生后令程序流程继续，但这种程序需要确保在异常发生后资源能被正确地释放。一个简单的确保资源被释放的方法是使用智能指针。

函数的退出有两种可能，正常处理结束或者发生了异常，无论哪种情况，局部对象都会被销毁。但与之相对的，当发生异常时，我们直接管理的内存是不会自动释放的。如果使用内置指针管理内存，且在 new 之后在对应的 delete 之前发生了异常，则内存不会释放：

```c++
void func() {
    int *ip = new int(42);
    // 抛出一个异常，且在func中未被捕获
    delete ip;
}
```

如果在 new 和 delete 之间发生异常，且异常未在 func 中被捕获，则内存就永远不会被释放。而在函数 func 之外没有指针指向这块内存，因此就无法释放它了。

默认情况下，shared_ptr 被销毁时，它默认地对它管理的指针进行 delete 操作

---


## 智能指针的误用

1、使用不用的智能指针托管同一块堆空间（裸指针），导致**double free**

```c++
void test()
{
    //使用不用的智能指针托管同一块堆空间
    Point *pt = new Point(1,2);
    unique_ptr<Point> up(pt);
    unique_ptr<Point> up2(pt);
}

```

2、使用重置智能指针托管同一块堆空间，也会导致**double free**

```c++
void test2()
{
     unique_ptr<Point> up(new Point(1,2));
     unique_ptr<Point> up2(new Point(3,4));
     up.reset(up2.get());
}
void reset(T *data)
{
    if(_data)
    {
        delete _data;
        _data = nullptr;
    }
    _data = data;
}
```

3、share_ptr 也会出现跟第一种一样的情况，只是说当用裸指针给 share_ptr 初始化时，引用计数是不会加 1 的，因为它们两个指针没有关系。所以我们要给另一个 share_ptr 初始化时，要用 share_ptr 指针取初始化。

```c++
void test3()
{
    //使用不用的智能指针托管同一块堆空间
    Point *pt = new Point(1,2);
    share_ptr<Point> sp(pt);
    //share_ptr<Point> sp2(pt);
    share_ptr<Point> sp2(sp); //用智能指针初始化而不用裸指针，这样就不会报错
}

```

4、使用重置功能，也使用了裸指针初始化指针了。

```c++
void test4()
{	//报错，因为也使用了裸指针初始化指针了。
     shared_ptr<Point> sp(new Point(1,2));
     shared_ptr<Point> sp2(new Point(3,4));
     sp.reset(sp2.get());
} 
```

5、注意函数的返回类型式裸指针也不行，也一样会出错。然而当我们将返回类型改为了 share_ptr 类型后，还是报错，这是为什么？

因为在 return 的时候我们的 this 也就是 sp 所在的对象给匿名托管了，两个不同的智能指针托管同一块空间必然报错。

![[20240719154629.png]]


解决办法：使用 shared_from_this

```c++
shared_ptr<Point> addPoint(Point *pt)
{
    _ix += pt->_ix;
    _iy += pt->_iy;
    
    /* return this; */
    /* return shared_ptr<Point>(this); */
    return shared_from_this(); //返回的是智能指针给智能指针初始化，而不是用裸指针初始化
    //但在使用的时候，这个类为了要使用shared_from_this函数，所以可以继承std::enable_shared_from_this<Point>
}
class Point
: public std::enable_shared_from_this<Point>

```
