## 概念介绍

`unique_ptr` 明确表明是 **独享所有权** 的智能指针，**无法进行拷贝，只能移动**。当它离开作用域时，会自动删除（删除器）它所管理的对象。

## 操作说明

### 基本用法

```c++
#include <memory>

// 创建一个 unique_ptr
std::unique_ptr<int> ptr1(new int(10));

// C++14 后更推荐使用 make_unique
auto ptr2 = std::make_unique<int>(20);

// 移动语义转移所有权
std::unique_ptr<int> ptr3 = std::move(ptr1); // ptr1 现在为 nullptr

// 访问指针内容
if (ptr3) {
    std::cout << *ptr3 << std::endl; // 输出 10
}
```

### 删除器

`unique_ptr` 默认使用 `delete` 释放资源，但允许自定义删除器。关键点在于：

- **删除器类型是 `unique_ptr` 类型的一部分**（编译期确定）
- 必须在模板参数中显式指定删除器类型
- 创建 `unique_ptr` 时需要传入具体的删除器对象

```c++
// 使用函数作为删除器
void FileDeleter(FILE* fp) {
    if (fp) fclose(fp);
}
std::unique_ptr<FILE, decltype(&FileDeleter)> filePtr(fopen("test.txt", "r"), FileDeleter);

// 使用 lambda 作为删除器
auto del = [](int* p) { delete p; };
std::unique_ptr<int, decltype(del)> ptr(new int, del);
```


### reset 使用

`reset` 接受一个可选的指针参数，令 `unique_ptr` 重新指向给定的指针。如果 `unique_ptr` 不为空，它原来指向的对象被释放。

```c++
unique_ptr<string> p2( new string("hello")); 
unique_ptr<string> p3( new string("Trex"));
// 释放 p2 指向的对象，并将 p3 对象的所有权转移给 p2
up2.reset(up3.release());
```

### release 使用

1. 切断 `unique_ptr` 与原对象的联系，放弃所有权
2. 返回原始指针，但 **不负责释放资源**
3. 必须由接收者管理该指针的生命周期：
   - 用另一个智能指针接管（推荐）
   - 手动 `delete`（不推荐，违背智能指针初衷）

```c++
unique_ptr<string> up( new string("hello")); 
up.release(); // 错误，内存泄漏

auto p = up.release();  // p 现在是原始指针，p2 变为空
delete p;               // 必须手动释放
```

当 `unique_ptr` 是一个右值（即将销毁的临时对象或显式转换为右值）时，可以通过移动构造或移动赋值来转移所有权，这看起来像是 "拷贝"，实际上是所有权的转移。

```c++
std::unique_ptr<int> create_int() {
    return std::make_unique<int>(42);  // 可以"拷贝"返回
}
```