我们在用智能指针来管理空间的时候，对于这块空间的开辟方式有很多，比如：malloc()函数，fopen 文件对象，new 堆上开辟，还有 new 开辟一个数组；这些不同方式开辟的空间对于这些空间的释放方式是不一样的。

所以，我们在删除这些动态开辟的空间的时，就不能直接一种方式来实现，要实现多种方式。使用者自己控制释放空间的方式的话，这种实现其实在之前已经说过很多了，就是使用仿函数，lambda表达式，函数指针的方式来实现。但是，**函数指针是不推荐使用**的。

函数指针不推荐使用的原因有以下几点：

1. 函数指针不够灵活。**在使用函数指针时，需要提前定义好函数的原型**，这限制了使用者在实现自定义释放方式时的灵活性。而使用仿函数、lambda 表达式等方式可以更灵活地实现自定义释放方式。

2. 函数指针可能导致内存泄漏。如果在使用函数指针时没有正确地释放动态开辟的空间，就可能导致内存泄漏。而使用智能指针时，可以通过自定义删除器来确保空间的正确释放，从而避免内存泄漏的问题。

3. 函数指针不利于代码的可读性和可维护性。使用函数指针时，需要在代码中查找对应的函数实现，这降低了代码的可读性。而使用仿函数、lambda 表达式等方式可以将自定义释放方式与智能指针的使用放在一起，提高了代码的可读性和可维护性。

综上所述，虽然函数指针可以实现自定义释放方式，但由于其不够灵活、可能导致内存泄漏以及不利于代码的可读性和可维护性等原因，不推荐使用函数指针来实现自定义释放方式。相反，推荐使用仿函数、lambda 表达式等更灵活、安全的方式来实现自定义释放方式。

具体例子引入：打开文件，写数据，关闭文件

```c++
void test(){
    string msg = "hello,world\n";
    FILE *fp = fopen("wd.txt", "a+");
    fwrite(msg.c_str(), 1, msg.size(), fp);
    fclose(fp);
}
```

通过fopen得到的fp看似像是一个指针，实则是一个文件描述符，内部并没有做new操作。如果使用unique_ptr：

```c++
void test(){
    unique_ptr<FILE> up(fopen("wd.txt", "a+"));
    fwrite(msg.c_str(), 1, msg.size(), up.get());
    flose(up.get());
}
```

运行后显然会报错的，原因在于 unique_ptr 是一个智能指针，fp是伪指针。虽然程序结束了，但并没有去做关闭文件的操作，导致报错。

看一下 unique_ptr 的定义

![[20240718093903.png]]

原来 unique_ptr 在使用的时候默认用的是 `std::default_delete<T>` 删除器，内部默认设置的 delete 函数。所以在程序结束时，我们是无法对一个未 new 的文件进行 delete 操作的。

那该怎么解决这个问题呢？ --- **重写一个删除器**

```c++
struct FILECloser{
    void operator()(FILE *fp){
        if(fp){
            fclose(fp);
            cout << "fclose(fp)" << endl;
        }
    }
}

void test(){
    string msg = "hello,world\n";
   	//传入一个删除器 FILECloser, 是仿照默认删除器写的
    unique_ptr<FILE, FILECloser> up(fopen("wd.txt", "a+"));
    fwrite(msg.c_str(), 1, msg.size(), up.get());
}
```

默认的删除器执行的是 delete 或者 `delete []` 的形式，所以针对 FILE 或者 socket 这种创建的资源，需要重新改写删除器，使用 fclose 或 close。

其他智能指针除了 share_ptr 具有删除器的操作，其他都没有。但 share_ptr 的删除器与 unique_ptr 的删除器不太一样。

<img src="https://pnglist.obs.cn-east-3.myhuaweicloud.com/typora/202312052128555.png" alt="image-20230515210458901" style="zoom:80%;" />

**share_ptr 的删除器是在构造函数里传入的。那写一下关于 share_ptr 删除器的使用**

```c++
void test2(){
    //要想从类型调用到对象，可以直接显式使用构造函数，比如：使用无参构造函数
    string msg = "hello,world\n";
    //注意 share_ptr 构造函数传入的删除器是一个对象，类名使用括号为匿名对象,调用的是无参构造函数
   	share_ptr<FILE> sp(fopen("wh.txt", "a+"), FILECloser());
    fwrite(msg.c_str(), 1, msg.size(), sp.get());
}
```

