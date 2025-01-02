# 概念介绍

weak_ptr 是一种不控制所指向对象生存期的智能指针，它指向由一个 shared_ptr 管理的对象。

将一个 weak_ptr 绑定到一个 shared_ptr 不会改变 shared_ptr 的引用计数。一旦最后一个指向对象的 shared_ptr 被销毁，对象就会被释放。即使有 weak_ptr 指向对象，对象也还是会被释放。


![[20240717134943.png]]

```c++
auto p = make_shared<int>(42);
// wp 弱共享 p；p的引用计数未改变
weak_ptr<int> wp(p);
```


# 使用说明

由于对象可能不存在，因此不能直接使用 weak_ptr 直接访问对象，而必须调用 lock。该函数检查 weak_ptr 指向的对象是否仍然存在。如果存在：lock 返回一个指向共享对象的 shared_ptr。

weak_ptr 它不能直接获取资源，必须通过 lock 函数从 wp 提升为 sp，从而判断共享的资源是否已经销毁。

```c++
void test() {
    // weak_ptr<int> wp(new int(20)); // 错误, 没有普通类型的构造函数
    shared_ptr<int> sp(new int(10));
    weak_ptr<int> wp;
    wp = sp;
    cout << "sp.use_count() = " << sp.use_count() << endl;
    cout << "wp.use_count() = " << wp.use_count() << endl;

    //如果引用计数为0，返回 true; 否则返回 false
    //若被管理对象已被删除则为 true；否则为 false
    bool flag = wp.expired(); 
    cout << "wp.expired() = " << flag << endl;
}
```

试验 expired 函数的作用，需要将sp给释放掉，但智能指针的释放不是delete，而是随着作用域的的生命周期结束而释放掉。

```c++
// 第一种方式
void test1() {
    bool flag;
    weak_ptr<int> wp;
    {
        shared_ptr<int> sp(new int(10));
        wp = sp;
        cout << "sp.use_count() = " << sp.use_count() << endl;
        cout << "wp.use_count() = " << wp.use_count() << endl;
        flag = wp.expired(); 
        cout << "wp.expired() = " << flag << endl;
    }

    cout << "--------离开作用域后，share_ptr释放----------" << endl;

    cout << "wp.use_count() = " << wp.use_count() << endl;
    flag = wp.expired(); 
    cout << "wp.expired() = " << flag << endl;
}
```

lock 将 weak_ptr 提升为一个 share_ptr，然后再来判断 shared_ptr，进而知道 weak_ptr 指向的空间还在还是不在。

```c++
//第2种方式
void test2() {
    weak_ptr<int> wp;
    {
        shared_ptr<int> sp(new int(10));
        wp = sp;
        cout << "sp.use_count() = " << sp.use_count() << endl;
        cout << "wp.use_count() = " << wp.use_count() << endl;
		// weak_ptr 调用 lock 函数，让自身提升为 shared_ptr
        shared_ptr<int> sp2 = wp.lock();
        if(sp2) {
            cout << "提升成功" << endl;
            cout << "*sp2 = " << *sp2 << endl;
            cout << "wp.use_count() = " << wp.use_count() << endl;
        } else{
            cout << "提升失败，wp指向的空间已经被回收了" << endl;
        }
    }
    cout << "--------离开作用域后，share_ptr释放----------" << endl;
    shared_ptr<int> sp2 = wp.lock();
    if(sp2) {
        cout << "提升成功" << endl;
        cout << "*sp2 = " << *sp2 << endl;
    } else {
        cout << "提升失败，wp指向的空间已经被回收了" << endl;
    }
}
```

