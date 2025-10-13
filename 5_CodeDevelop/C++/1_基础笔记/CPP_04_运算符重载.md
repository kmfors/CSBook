# 运算符重载

C++ 预定义中的运算符的操作对象只局限于基本的内置数据类型，但是对于我们自定义的类型是没有办法操作的。但是大多时候我们需要对我们定义的类型进行类似的运算，这个时候就需要我们对这么运算符进行重新定义，赋予其新的功能，以满足自身的需求。

为了使对用户自定义数据类型的数据的操作与内置数据类型的数据的操作形式一致，C++ 提供了运算符的重载。通过把 C++ 中预定义的运算符重载为类的成员函数或者友元函数，使得对用户的自定义数据类型的数据（对象）的操作形式与 C++ 内部定义的类型的数据一致。

运算符重载的实质就是 **函数重载** 或 **函数多态**。运算符重载是一种形式的 C++ 多态。目的在于让人能够用同名的函数来完成不同的基本操作。要重载运算符，需要使用被称为运算符函数的特殊函数形式，运算符函数形式：

```c++
ReturnType operator op(ArgumentType arg);
```

## 运算符重载的规则

运算符重载具有以下规则：

1. 为了防止用户对标准类型进行运算符重载，C++规定重载的运算符的操作对象必须至少有一个是自定义类型或枚举类型
2. 运算符重载的时候，不能改变优先级与结合性
3. 运算符重载的时候，不能改变操作数的个数、不能改变操作数顺序，当然肯定不能设置默认值。
4. 不能臆造不存在的运算符，例如：$、@
5. 重载逻辑运算符（&&,||）后，不再具备短路求值特性  



### 不能重载的运算符

- 成员访问运算符 `.`
- 成员指针运算符 `*`
- 三目运算符 `?:`
- 作用域限定符 `::`
- `sizeof` 运算符：如果 `sizeof()` 是一个函数的话，那么肯定需要使用参数列表。

### 可以重载的运算符

![image-20250120214259447](/1_store/1_asset/image-20250120214259447.png)



## 运算符重载的形式

运算符重载的形式有三种：

- 采用普通函数的重载形式
- 采用成员函数的重载形式
- 采用友元函数的重载形式

### 以普通函数形式重载

运算符重载的第一种形式：运算符重载之普通函数的形式。特点是：

- **不是类的成员**，而是独立的全局函数或命名空间内的函数
- **无法直接访问类的私有成员**，必须通过公有接口（如 **get** 方法）或友元声明。
- **适用于对称运算符**（如 `+`、`==`），尤其是当左操作数不是当前类对象时（如 `3 + obj`）

```c++
class Complex {
private:
    double real, imag;
public:
    Complex(double r, double i) : real(r), imag(i) {}
    double getReal() const { return real; } // 提供 get 方法
    double getImag() const { return imag; }
};

// 普通函数形式的运算符重载(通过公有接口访问)
Complex operator+(const Complex& a, const Complex& b) {
    return Complex(a.getReal() + b.getReal(), a.getImag() + b.getImag());
}

int main() {
    Complex c1(1, 2);
    Complex c2(3, 4);
    Complex c3 = c1 + c2;
}
```

---

### 以成员函数形式重载

运算符重载的第二种形式：成员函数的形式。特点：

- **是类的成员函数**，默认以 `*this` 作为左操作数。
- **可以直接访问类的私有成员**，无需友元或 `get` 方法
- **适用于修改当前对象状态的运算符**（如 `+=`、`=`、`++`）

```c++
class Complex {
private:
    double real, imag;
public:
    Complex(double r, double i) : real(r), imag(i) {}

    // 成员函数形式的运算符重载
    Complex operator+(const Complex& other) const {
        return Complex(real + other.real, imag + other.imag);
    }
};
int main() {
    Complex c1(1, 2);
    Complex c2(3, 4);
    Complex c3 = c1 + c2;
}
```

---

### 以友元函数形式重载（推荐）

运算符重载的第三种形式：友元函数的形式。特点：

- **不是成员函数**，但通过 `friend` 声明获得访问私有成员的权限。
- **结合了普通函数和成员函数的优点**：支持对称操作 + 直接访问私有成员。
- **需在类内显式声明 `friend`**。

```c++
class Complex {
private:
    double real, imag;
public:
    Complex(double r, double i) : real(r), imag(i) {}

    // 声明友元函数
    friend Complex operator+(const Complex& a, const Complex& b);
};

// 友元函数可直接访问私有成员
Complex operator+(const Complex& a, const Complex& b) {
    return Complex(a.real + b.real, a.imag + b.imag);
}
int main() {
    Complex c1(1, 2);
    Complex c2(3, 4);
    Complex c3 = c1 + c2;
}
```



## 特殊运算符的重载

### 复合赋值运算符

以成员函数的形式进行重载，对于 **对象本身发生变化**的运算符，一般建议使用**成员函数**的形式进行重载。

复合赋值运算符： `+=   -=    *=    /=    %=   <<=	>>=`

```c++
// c2 += c1; //等价于c2.operator+=(c1); 
Complex operator+=(const Complex &rhs) {
    _dreal += rhs._dreal;
    _dimag += rhs._dimag;
    return *this;
}
```

注意：返回的是对象，所以会创建一个匿名对象，数据拷贝结束后，匿名对象调用析构销毁。赋值运算符函数重载要考虑深拷贝的问题。

---

### 自增自减运算符

自增运算符`++`和自减运算符`--`，**推荐以成员函数形式重载**，分别包含两个版本，即运算符前置形式(如 `++x`)和运算符后置形式(如 `x++`)。

二者操作不一样，需要在对这两个运算符进行重载区分前置和后置形式。

**C++根据参数的个数来区分前置和后置形式**。如果按照通常的方法（成员函数形式）来重载`++`/`--`运算符，那么重载的就是前置版本。

要对后置形式进行重载，就必须为重载函数再增加一个 int 类型的参数，该参数仅仅用来告诉编译器这是一个运算符后置形式，在实际调用时不需要传递实参。

#### 前置运算符重载：

```c++
// ++c2; //等价于c2.operator++();
// 自增自减运算符重载推荐：以成员函数形式重载
Complex operator++() {
    ++_dreal;
    ++_dimag;
    return *this;
}
```

#### 后置运算符重载：

```c++
// c2++ ; //等价于c2.operator++(10)，括号里随便写一个整数
Complex operator++(int) { // int知识一个标识，并不代表传参
    Complex tmp(*this);
    _dreal++;
    _dimag++;
    return tmp;
}
```



### 输入输出流运算符

注意：不能将输入输出流运算符以成员函数的形式进行重载，要以**友元函数**进行重载

如果将输出流运算符函数以成员函数的形式进行重载，那么就必须满足两个参数的要求，所以需要将 Complex 类型的对象用 this 指针进行替代。
但是这样就会改变流对象 os 与 Complex 对象的位置，这样就违背了运算符重载的规则，不能改变操作的位置，所以就不能将输出流运算符以成员函数的形式进行重载。

对于运算符重载而言，要确定函数的名字（这个比较简单），然后需要确定函数的参数列表，最后确定函数的返回类型。[[秘法笔记#输入输出流运算符重载|示例代码]]

```c++
#include <iostream>

using namespace std;

class Complex {
    //以友元函数进行输出运算符重载
    friend std::ostream& operator << (std::ostream& os, const Complex& rhs);
    //以友元函数进行输入运算符重载
    friend std::istream& operator >> (std::istream& is, Complex& rhs);
public:
    //std::ostream &operator << (std::ostream &os);不可以，这会导致成员函数的第一个参数是this，导致后面
    //需要用类名或对象调用，不美观也与初衷相违背，也无法实现连操作
    Complex(int dreal = 0, int dimag = 0) :_dreal(dreal), _dimag(dimag) {}
private:
    int _dreal;
    int _dimag;
};

std::ostream& operator << (std::ostream& os, const Complex& rhs) {
    os << "operatpr<<()" << endl;
    os << "(" << rhs._dreal << "," << rhs._dimag << ")";
    return os;
}
std::istream& operator >> (std::istream& is, Complex& rhs) {
    cout << "operator>>()" << endl;
    is >> rhs._dreal >> rhs._dimag;
    return is;
}
```




### 重载小括号（函数调用运算符）

了解什么是函数对象

注意：将重载了函数调用运算符的类创建的对象称为函数对象。

```c++
//已将函数调用运算符重载的类，创建的对象称为函数对象
class FunctionObject {
public:
    FunctionObject() :_cnt(0) {}
    int operator()(int& x, int& y) {
        ++_cnt;
        return x + y;
    }
    int operator()(int& x, int& y, int& z) {
        ++_cnt;
        return x * y * z;
    }
private:
    int _cnt;//函数对象的状态
};

int func(int& x, int& y, int& z) {
    static int cnt = 0;
    ++cnt;
    return x + y + z;
}

void test() {
    int a = 3, b = 4, c = 5;
    FunctionObject obj1; //函数对象
    cout << obj1(a, b) << endl; // 等价于obj1.operator()(a,b)
    cout << obj1(a, b, c) << endl;
    //调用普通函数
    cout << func(a, b, c) << endl;
}
```






### 重载中括号(下标访问运算符）

特征：下标访问运算符的重载可以让程序更加安全点。

**引用符号什么时候需要加上？**

1、如果函数的返回值的生命周期比函数的生命周期大的时候，为了避免在执行 return 语句的时候多执行一次拷贝操作，所以尽量使用引用

2、在流中，是可以无限传递参数执行下去，这个时候用引用可以减少多次拷贝，提高程序的执行效率

3、在赋值运算符函数的返回类型的时候，也是返回的引用，因为连等的时候，可以少执行拷贝操作，也可以提高程序的执行效率

```c++
#include <string.h>
#include <iostream>

using std::cout;
using std::endl;

class CharArray {
public:
    CharArray(size_t size) :_size(size), _data(new char[_size]) {}

    char& operator[](size_t idx) {
        //2、如果索引值不超过数组大小，就返回_data[idx]
        if (idx < _size) {
            return _data[idx];
        } else {
            //只要传来的数大于容量，就做这一步操作，防止数组越界，保证自身数组安全
            static char nullchar = '\0';
            return nullchar;
        }
    }
    size_t size()const { return _size; }
    ~CharArray() { delete[] _data; }

private:
    size_t _size;
    char* _data;
};

void test() {
    const char* ptr = "hello,world";
    CharArray arr1(strlen(ptr) + 1); // 1、记录空间大小，申请空间

    for (size_t idx = 0; idx != 13; ++idx) {
        arr1[idx] = ptr[idx]; //  还原：arr1.operator[](idx) = ptr[idx];
        //  3、如果索引值正确，则等价于_data[idx] = ptr[idx];
        //  索引值不正确，返回 nullchar = ptr[idx];
    }
    for (size_t idx = 0; idx != arr1.size(); ++idx) {
        //  还原：cout << arr1.operator[](idx) <<endl;
        cout << arr1[idx] << endl;  // 多读出来的是空字符
    }
    cout << arr1.size() << endl;    // 打印12

}
int main(void) {
    test();
    return 0;
}
```



### 成员访问运算符

箭头访问运算符 `->` 重载、解引用运算符 `*` 重载。

```c++
#include <iostream>

using std::endl;
using std::cout;

class Data { // 数据类
public:
    Data(int data = 0) :_data(data) {
        cout << "Data(int data = 0)" << endl;
    }
    ~Data() { cout << "~Data()" << endl; }

    int getData() const { return _data; }

private:
    int _data;
};

// 智能指针类：不考虑内存回收，本身具有内存回收的功能
// 直接管理对象，管理的是 Data 对象指针
class SecondLayer { 
public:
    // 重载箭头访问运算符
    Data* operator->() {
        return _data; // 箭头访问 数据类对象指针，访问数据类内部用->
    }
    // 重载解引用运算符
    Data operator*() {
        return *_data;//解引用访问 数据类对象，访问数据类内部使用.
    }

    SecondLayer(Data* pdata) :_data(pdata) {
        cout << "SecondLayer(Data *data)" << endl;
    }
    ~SecondLayer() {
        cout << "~SecondLayer()" << endl;
        if (_data) {
            delete _data;
            _data = nullptr;
        }
    }
private:
    Data* _data;//数据成员是你要操作数据类的指针对象
};

// 智能指针类：不考虑内存回收，本身具有内存回收的功能
// 间接管理对象，管理的是 SecondLayer 对象指针
class ThirdLayer {
public:
    ThirdLayer(SecondLayer* ps1) :_sl(ps1) {
        cout << "ThirdLayer(SecondLayer *SecondLayer)" << endl;
    }
    // 重载箭头访问运算符
    SecondLayer& operator->() { return *_sl; }

    // 重载解引用运算符（返回 Data 对象）
    Data operator*() {
        return **_sl; // 调用 SecondLayer 的 operator*()
    }

    ~ThirdLayer() {
        cout << "~ThirdLayer()" << endl;
        if (_sl) {
            delete _sl;
            _sl = nullptr;
        }
    }
private:
    SecondLayer* _sl;
};

void test() {
    SecondLayer s1(new Data(10));
    cout << "s1->getData() = " << s1->getData() << endl;
    // 还原：s1.operator->()->getData();

    cout << "(*s1).getData() = " << (*s1).getData() << endl;
    // 还原：s1.operator*().getData();
}

void test2() {
    ThirdLayer t1(new SecondLayer(new Data(11)));
    cout << "t1->getData() = " << t1->getData() << endl;
    //还原：tl.operator->().operator->()->getData();涉及多层的函数重载

    cout << "(*t1).getData() = " << (*t1).getData() << endl; 
    // 还原：t1.operator*().getData()
}
//--------------------------------------------------------------------------
//SecondLayout是一个栈对象，栈对象随着生命周期的结束，而调用析构函数，去回收new的空间。
//虽然它不是指针，但通过重载，却比指针好用，不用考虑delete,相当于智能指针！以后会用到很多
```





## 运算符重载总结

对于运算符重载时采用的形式的建议：

- 所有的一元运算符，建议以成员函数重载

- 运算符 `= () [] -> *` ，必须以成员函数重载

- 运算符 `+= -= /= *= %= ^= &= != >>= <<=` 建议以成员函数形式重载

- 其它二元运算符，建议以非成员函数重载



## 函数对象

注意：将重载了函数调用运算符的类创建的对象称为函数对象

```c++
template <typename T>
struct CompareList
{
    bool operator()(const T &lhs, const T &rhs) const
    {
        cout << "bool operator()(const T &, const T &) const" << endl;
        return lhs < rhs;
    }
};
//每次使用时，CompareList com
// com(lhs,rhs) 等价于 com.operator()(lhs,rhs)
```



