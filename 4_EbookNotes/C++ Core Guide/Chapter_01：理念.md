
# 1、在代码中直接表达思想

Reason：程序员应该直接用代码表达他们的思想，因为代码可以被编译器和工具检查。

Example1:

```c++
class Date {
public:
    Month month() const;  // do
    int month();          // don't
    // ...
};
```

第一个成员函数的声明更好的表达了它既不修改日期对象，也明确返回的是一个月份的意义。使得这个声明更贴切表达程序员的思想。


Example2：比如显式写一个范围内循环查询某个值，这样的操作实现通常需要理解且维护。然后使用 STL 的 std::find 函数，节省时间且不易出错且形象易懂。

一个精心设计的库所表达的意图（要做什么，而不仅仅是如何做某事）要比直接使用语言功能好得多。C++ 程序员应了解标准库的基本知识，并在适当的地方使用标准库。任何程序员都应了解所从事项目的基础库的基本知识，并合理使用它们。

Example3：语义的不明确

```c++
change_speed(double s);   // bad: what does s signify?
// ...
change_speed(2.3);
```

更好的办法是明确 double 的含义，因此可以采取更好的方式表达语义

```c++
change_speed(Speed s);    // better: the meaning of s is specified
// ...
change_speed(2.3);        // error: no unit
change_speed(23_m / 10s);  // meters per second
```


Enforcement：

- 持续使用 const（检查成员函数是否修改了其对象；检查函数是否修改了通过指针或引用传递的参数）
- 标记转换使用
- 检测模仿标准库的代码（很难）。

# 2、用 ISO 标准 C++ 写代码

使用当前的 C++ 标准，不要使用编译器扩展。注意未定义行为与实现定义行为。

使用有效的 ISO C++ 并不能保证可移植性（更不用说正确性）。请避免依赖未定义的行为（如未定义的求值顺序），并注意具有实现定义含义的构造（如 sizeof(int)）。

