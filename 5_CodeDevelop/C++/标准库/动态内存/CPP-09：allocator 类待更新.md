![[20240719153030.png]]

标准库 allocator 类定义在头文件 memory 中，它帮助我们将内存分配和对象构造分离开来。它提供了一种类型感知的内存分配方法，它分配的内存是原始的、未知的。

类似 vector，allocator 是一个模板。为了定义一个 allocator 对象，我们必须指明这个 allocator 可以分配的对象类型。当一个 allocator 对象分配内存时，它会根据给定的对象类型来确定恰当的内存大小和对齐位置：

```c++
allocator<string> alloc;

// 分配n个未初始化的string
auto const p = alloc.allocate(n); 
```

![[20240719153824.png]]


![[20240719153908.png]]