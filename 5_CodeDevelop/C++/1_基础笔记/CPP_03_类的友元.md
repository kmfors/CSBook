## 友元概念

面向对象四大基本特征：抽象、封装、继承、多态。友元具有可以打破封装性的特点。

友元的缘由：类中的 **私有成员** 被声明为 `private` 类型，只能被类内的成员函数访问，类外是不能访问的。友元的出现打破类外无法访问私有成员的问题。

友元的使用：当一个外部函数或外部类，需要访问某个类的私有或保护成员时，我们可以在该类的内部将外部函数或者外部类声明为友元。 被声明为友元的外部函数或者外部类，将获得直接访问该类所有私有和保护成员的特权。

友元的意义：单独开辟一条绿色通道，供其访问私有成员。它提供了一种 **突破封装** 的机制，允许特定的外部函数或外部类访问当前类的私有和保护成员

```C++
class ClassA 
{
    friend retType funcName(params); // 可以是成员函数也可以是非成员函数
    friend class ClassB;
};
```

1. **友元(全局)函数** ：你是我的好朋友，我只允许你知道我的秘密
2. **友元类**：你们家都是我的好朋友，我只允许你们家人知道我的秘密
3. **友元成员函数**：你们家的 **某一个特定成员** 是我的好朋友，我只允许他知道我的秘密

> [!NOTE]
>
> 1. 友元关系不能被继承
> 2. 友元关系是单向的，类 A 是类 B 的朋友，但类 B 不一定就是类 A 的朋友
> 3. 友元关系不具有传递性。类 B 是类 A 的朋友 ，类 C 是类 B 的朋友，但类 C 不一定是类 A 的朋友
> 4. 友元的声明是不受 `public/protected/private` 关键字限制的，任意位置声明都可以。
>



## 友元函数

1. **声明方式**：
   - 若要使普通函数能够访问类的私有成员，必须在该类的定义内部显式声明该函数为友元，使用 `friend` 关键字修饰函数原型。
   - 此声明不受类访问权限（`public/private/protected`）区域的影响。
2. **访问权限**：
   - 被声明的友元函数在类外定义时，可通过以下方式访问类的私有成员：
     - **通过对象实例**（包括局部对象、全局对象）
     - **通过指针或引用形参**（传递对象的地址或别名）
     - *注意：友元函数本身不属于类，因此无 `this` 指针*

```c++
#include <iostream>
#include <cmath>

using std::cout;
using std::endl;

class Point{
public:
    // 友元全局函数声明
    friend double distance(const Point &lhs, const Point &rhs);

    ~Point() = default;
    Point(int ix = 0, int iy = 0) : _ix(ix), _iy(iy) {}

    static void print(const Point &lhs, const Point &rhs) {
    cout << "(" << lhs._ix << ", " << lhs._iy << ")"
        << " ----> "
        << "(" << rhs._ix << ", " << rhs._iy << ")" ;
    }
private:
    int _ix;
    int _iy;
};

// 1、友元全局函数（自由函数）
double distance(const Point &lhs, const Point &rhs) {
    int dx = lhs._ix - rhs._ix;
    int dy = lhs._iy - rhs._iy;
    return hypot(dx, dy);
}

int main() {
    Point pt1(1, 2);
    Point pt2(4, 6);

    // 调用友元函数实现对私有成员的间接访问
    Point::print(pt1, pt2);
    cout << " 之间的距离: " << distance(pt1, pt2) << endl;
}
```




## 友元类

将一个类声明为友元后，该友元类的所有成员函数都可以访问声明友元关系的类的私有成员。

```c++
#include <iostream>
#include <cmath>

using std::cout;
using std::endl;

class Point{
public:
    friend class Line; // 友元类声明

    ~Point() = default;
    Point(int ix = 0, int iy = 0) : _ix(ix), _iy(iy) {}

private:
    int _ix;
    int _iy;
};

// 2、友元类
class Line{
public:
    void print(const Point &lhs, const Point &rhs) {
    cout << "(" << lhs._ix << ", " << lhs._iy << ")"
        << " ----> "
        << "(" << rhs._ix << ", " << rhs._iy << ")" ;
    }

    double distance(const Point &lhs, const Point &rhs) {
        int dx = lhs._ix - rhs._ix;
        int dy = lhs._iy - rhs._iy;
        return hypot(dx, dy);
    }
};


int main() {
    Point pt1(1, 2);
    Point pt2(4, 6);

    Line line;

    // 友元类创建对象，对象调用函数间接访问私有数据
    line.print(pt1, pt2);
    cout << " 之间的距离: " << line.distance(pt1, pt2) << endl;
}
```



## 友元成员函数

当需要仅允许其他类的 **特定成员函数**（而非整个类）访问当前类的私有成员时，可使用 **友元成员函数** 机制。

注意，尤其在友元成员函数中，一定要注意 **循环依赖** 和 **声明顺序** 的问题。

```c++
#include <iostream>
#include <cmath>

using std::cout;
using std::endl;

// 前向声明Point类
class Point;

// 先声明Line类
class Line{
public:
    void print(const Point &lhs, const Point &rhs);
    double distance(const Point &lhs, const Point &rhs);
};

class Point{
public:
    // 友元成员函数
    friend void Line::print(const Point &lhs, const Point &rhs);
    friend double Line::distance(const Point &lhs, const Point &rhs);

    ~Point() = default;
    Point(int ix = 0, int iy = 0) : _ix(ix), _iy(iy) {}

private:
    int _ix;
    int _iy;
};

// 最后实现Line类的成员函数
void Line::print(const Point &lhs, const Point &rhs) {
    cout << "(" << lhs._ix << ", " << lhs._iy << ")"
         << " ----> "
         << "(" << rhs._ix << ", " << rhs._iy << ")" ;
}

double Line::distance(const Point &lhs, const Point &rhs) {
    int dx = lhs._ix - rhs._ix;
    int dy = lhs._iy - rhs._iy;
    return hypot(dx, dy);
}

int main() {
    Point pt1(1, 2);
    Point pt2(4, 6);

    Line line;

    // 友元类创建对象，对象调用函数间接访问私有数据
    line.print(pt1, pt2);
    cout << " 之间的距离: " << line.distance(pt1, pt2) << endl;
}
```

