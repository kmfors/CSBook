

## I/O 流机制

在 C++ 中，I/O 操作通过 **流**(stream)的概念实现。**流** 本质上是一个字节序列，表示数据在源与目标之间的流动。根据数据方向的不同，可分为两类：

1. **输入流**(input stream)：当字节从外部设备（如键盘、磁盘、网络等）流向程序内存时，称为输入操作。程序通过输入流读取这些数据。
2. **输出流**(output stream)：当字节从程序内存流向外部设备（如显示器、打印机、磁盘、网络等）时，称为输出操作。程序通过输出流写入这些数据。

从 C++程序的角度看，I/O 操作的核心是与字节流的交互：

- 程序需要确保正确地生成要输出的字节数据
- 程序需要正确地解析输入的字节数据

流抽象的一个重要优势是它隐藏了底层设备的细节。程序员只需关注与流的交互，而不必关心数据具体来自或去向何种物理设备（如磁盘、网络等），这种抽象通过 C++的流类层次结构实现。



### 流的类型

在 C++中，I/O 流类型主要分为三类，它们都定义在标准库头文件中，形成一个层次化的类体系：

1. 标准 I/O 流（控制台 I/O），对系统的标准设备的输入和输出。
2. 文件 I/O 流，以磁盘文件为对象进行输入和输出。
3. 字符串 I/O 流，对内存中指定的空间进行输入和输出。



### 流类型之间的关系

`ios` 是抽象基类，由它派生出 `istream` 类和 `ostream` 类，`iostream` 类支持输入输出操作，`iostream` 类是从 `istream` 类和 `ostream` 类通过多重继承而派生的类。类 `ifstream` 继承了类 `istream`，类 `ofstream` 继承了类 `ostream`，类 `fstream` 继承了类 `iostream`。

![image-20250509091842678](/1_store/1_asset/image-20250509091842678.png)

### 流的状态

在 C++标准库中，流的状态由 `iostate` 类型表示，它是一个 **位掩码类型**，包含以下四种状态标志（所有编译器实现均遵循此标准）：

#### 状态标志定义

| 状态标志  | 值（通常） | 描述                           |
| --------- | :--------- | :----------------------------- |
| `goodbit` | `0x00`     | 流处于正常状态（无错误）       |
| `eofbit`  | `0x01`     | 到达文件末尾（End-Of-File）    |
| `failbit` | `0x02`     | 发生逻辑错误（可恢复）         |
| `badbit`  | `0x04`     | 发生系统级错误（通常不可恢复） |

#### 关键行为说明

- **`badbit`**：表示 **底层硬件或系统级故障**（如磁盘损坏、无权限访问文件）
  - **流通常不可继续使用**（即使调用 `clear()` 也可能无法恢复）
- **`failbit`**：由 **逻辑错误** 触发（如尝试读取数字时遇到字母字符）
  - 流缓冲区仍有效，可通过 `clear()` 重置状态后继续使用
  - **与 `eofbit` 的关系**：读取到文件末尾时，会同时设置 `eofbit` 和 `failbit`
- **`eofbit`**：仅表示 **到达数据源末尾**（不一定是错误）
  - 单独设置时仍可继续操作流（如检查状态）
- **`goodbit`**：等价于所有错误位均未设置（`good() == true`）
  - **注意**：`goodbit` 本身的值是 `0`，而非一个独立的标志位



#### 状态检测方法

| 方法        | 等效条件                     | 典型使用场景             |
| :---------- | ---------------------------- | :----------------------- |
| `good()`    | `state() == goodbit`         | 检查流是否完全可用       |
| `eof()`     | `state() & eofbit`           | 检测是否到达文件末尾     |
| `fail()`    | `state() & (failbit|badbit)` | 检查是否可恢复错误       |
| `bad()`     | `state() & badbit`           | 检测硬件/系统级错误      |
| `operator!` | `fail()`                     | 条件判断（如 `if(!cin)`） |



### 流的状态检测

```c++
bool bad() const; // 若流的 badbit 置位，则返回 true;否则返回 false
bool fail() const; // 若流的 failbit 或 badbit 置位，则返回 true;
bool eof() const; // 若流的 eofbit 置位，则返回 true;
bool good() const; // 若流处于有效状态，则返回 true;

iostate rdstate() const; // 获取流的状态
void setstate(iostate state); // 设置流的状态

// clear 的无参版本会复位所有错误标志位（重置流的状态）
void clear(std::ios_base::iostate state = std::ios_base::goodbit);
```

- `badbit`，一种系统级别的错误，该种错误不能恢复，可以使用函数 `bad()` 进行判断。
- `failbit`，是一种可以恢复的错误，可以重新置位流的状态，可以使用函数 `fail()` 进行判断。
- `eofbit`，是一种异常，表示文件指针到达了文件末尾，可以使用函数 `eof()` 进行判断。
- `goodbit`，是一种流的正常状态，可以使用函数 `good()` 进行判断。



```shell
流状态判断流程：
	good()
	│
	├─ true → 正常状态
    │
    └─ false → 检查具体原因：
       ├─ eof() → 文件末尾
       ├─ fail() → 可恢复错误
       └─ bad() → 系统级错误
```



### 流的通用操作

**输入流操作**：

```c++
int_type get();//读取一个字符
istream & get(char_type & ch);

//读取一行数据
istream& getline(char_type * s, std::streamsize count, char_type delim ='\n');

//读取count个字节的数据
istream& read(char_type * s, std::streamsize count);

//最多获取count个字节，返回值为实际获取的字节数
std::streamsize readsome(char_type * s, std::streamsize count);

//读取到前count个字符或在读这count个字符进程中遇到delim字符就停止，并把读取的这些东西丢掉
istream& ignore(std::streamsize count = 1, int_type delim = Traits::eof());

//查看输入流中的下一个字符, 但是并不将该字符从输入流中取走
//不会跳过输入流中的空格、回车符; 在输入流已经结束的情况下，返回 EOF。
int_type peek();

//获取当前流中游标所在的位置
pos_type tellg();

//偏移游标的位置
basic_istream& seekg(pos_type pos);
basic_istream& seekg(off_type off, std::ios::seekdir dir);

```

**输出流操作**：

```c++
//往输出流中写入一个字符
ostream& put(char_type ch);

//往输出流中写入count个字符
ostream& write(const char_type * s, std::streamsize count);

//获取当前流中游标所在的位置
pos_type tellp();

//刷新缓冲区
ostream& flush();

//偏移游标的位置
ostream& seekp(pos_type pos);
ostream& seekp(off_type off, std::ios_base::seekdir dir);
```

**ignore 函数的使用：** 很好的解决了缓冲区遗留字符的问题

```c++
//读取到前count个字符或在读这count个字符进程中遇到delim字符就停止，并把读取的这些东西丢掉
istream & ignore(std::streamsize count = 1, int_type delim = Traits::eof());
//举简单例子
cin.ignore(1024,'\n')	//对输入缓冲区内的前1024个字节内，如果遇到字符\n,就将\n之前的内容丢掉
    
//查看源码,等价于
#include <limits> //不要忘记加头文件
cin.ignore(std::numeric_limits<std::streamsize>::max(),'\n')//max()当前的缓冲区的最大值
```



## 缓冲区

**缓冲区**(Buffer)是内存中预留的一块特定区域，用于临时存储输入/输出数据，目的是 **协调不同速度设备之间的数据传输**。在 C++ I/O 系统中：

1. **缓冲区的本质**

   - 是内存空间的一部分（通常是 `streambuf` 类管理的区域）
   - 由标准库自动分配和管理（程序员通常不直接操作）

2. **分类方式**
   *按数据流向*：

   - **输入缓冲区**：暂存从输入设备（如键盘、文件）读取的数据
   - **输出缓冲区**：暂存准备写入输出设备（如屏幕、文件）的数据

   *按缓冲策略*：

   - **全缓冲**（如文件 I/O）：缓冲区满时才执行实际 I/O
   - **行缓冲**（如控制台）：遇到换行符 `\n` 时刷新
   - **无缓冲**（如 `cerr`）：立即输出

3. **关键特性**：

   - 减少直接 I/O 操作次数（提升性能）
   - 支持 `flush()` 强制刷新缓冲区
   - 程序正常终止时会自动刷新所有缓冲区



### 为什么要引入缓冲区呢？

比如我们从磁盘里取信息，我们先把读出的数据放在缓冲区，计算机再直接从缓冲区中取数据，等缓冲区的数据取完后再去磁盘中读取，这样就可以减少磁盘的读写次数，再加上计算机对缓冲区的操作大大快于对磁盘的操作，故应用缓冲区可大大提高计算机的运行速度。

因此 **缓冲区** 就是一块内存区，它用在输入输出设备和 CPU 之间，用来缓存数据。**它使得低速的输入输出设备和高速的 CPU 能够协调工作**，避免低速的输入输出设备占用 CPU，解放出 CPU，使其能够高效率工作。

从上面的描述中，不难发现缓冲区向上连接了程序的输入输出请求，向下连接了真实的 I/O 操作。作为中间层，必然需要分别处理好与上下两层之间的接口，以及要处理好上下两层之间的协作。

### 缓冲区的类型

缓冲区分为三种类型：全缓冲、行缓冲和不带缓冲。

- 全缓冲：在这种情况下，当填满标准 I/O 缓存后才进行实际 I/O 操作。全缓冲的典型代表是对磁盘文件的读写。
- 行缓冲：在这种情况下，当在输入和输出中遇到换行符时，执行真正的 I/O 操作。这时，我们输入的字符先存放在缓冲区，等按下回车键换行时才进行实际的 I/O 操作。典型代表是键盘输入数据。
- 不带缓冲：也就是不进行缓冲，标准出错情况 **cerr/stderr** 是典型代表，这使得出错信息可以直接尽快地显示出来。

### 缓冲区的刷新

- 程序正常结束的时候，会刷新缓冲区。
- 缓冲区满的时候（在 ubuntu1804 上缓冲区的大小正好是 1024）

```c++
cout << "123" << endl;  // 可以刷新并且换行
cout << "456" << flush; // 可以刷新但是不能换行
cout << "789" << ends;  // 不能刷新也不能换行
```



### 流缓冲区

在 C++中，流的缓冲区的基类是定义在 `streambuf` 头文件中，定义着两个类: `ios_base` 和 `ios`。

`ios_base` 是所有 I/O 类的祖先，提供了 **状态信息**、**控制信息**、**内部存储**、**回调** 等设施。`ios` 继承自 `ios_base`，额外提供了与 `streambuf` 的接口；同时允许多个 `ios` 对象绑定同一个 `streambuf` 对象。

由于 `ios_base` 没有提供与 `streambuf` 的接口，`ios` 才是标准库内所有 I/O 类（模板）事实上的最近共同祖先。`ios` 的成员函数 `rdbuf` 是读取和设置流对象（`ios` 的对象）绑定缓冲区的成员函数，它有两个不同的重载形式，分别如下：

```c++
//返回与之关联的streambuf；如果没有，则返回nullptr
streambuf *rdbuf() const;
//重新设置streambuf
//如果有，与先前绑定的streambuf解绑，再绑定传入的streambuf；
//如果传入的是nullptr，则流对象不与任何缓冲区对象绑定
streambuf *rdbuf(streambuf * sb);
```

`streambuf` 本身不可以直接创建对象，它是一个抽象类型；但它向下派生出了 2 个派生类型，如我们在类图中看到的 `filebuf` 和 `stringbuf`，这两个类分别是作为文件流和字符串流的子对象的，可以直接创建对象，后面我们会再次看到。







## 标准 IO

标准输入流 cin，标准输出流 cout，cerr（标准错误输出，不带缓冲），clog（标准错误输出，带缓冲）

重置流的状态 clear()函数，清空缓冲区 ignore();

### 标准输入流

`cin` 是标准输入流，它从标准输入设备(键盘)获取数据，程序中的变量通过 **流提取符 >>** 从流中提取数据。

流提取符 >> 从流中提取数据时通常跳过输入流中的空格、tab 键、换行符等空白字符。只有在输入完数据再按回车键后，该行数据才被送入键盘缓冲区，形成输入流，提取运算符 >> 才能从中提取数据。需要注意要保证从流中读取数据是能正常进行。

```c++
#include <iostream>
#include <limits>

using std::cin;
using std::cout;
using std::endl;
using std::cerr;

void test(){
    int number = 0;
    // 真值表达式 while(cin)，要么真、要么假
    // 采用逗号表达式，取最后一个逗号的右边的对象
    while(cin >> number, !cin.eof()){	
        if(cin.bad()){          // 无法恢复的系统级别错误
            cerr << "The stream is bad" << endl;
            return ;
        } else if(cin.fail()){  // 可以恢复的错误
            cin.clear();        //重置流的状态
            // 一次性清空缓冲区中所有残留字符，遇到换行符停止忽略
            cin.ignore(std::numeric_limits<std::streamsize>::max(),'\n');
            cout << "please input int data " << endl;
        } else{
            cout << "number = " << number << endl;
        }
    }
}
```



### 标准输出流

ostream 类定义了 3 个全局输出流对象，即 cout, cerr, clog，平常用的最多的就是 cout, 即 **标准输出**。cout 将数据输出到终端，它与标准 C 输出 stdout 关联。cerr 是标准错误流（非缓冲），clog 也是标准错误流（带缓冲）。注意：在 C 语言中，标准输入、标准输出和标准错误分别用 0, 1, 2 文件描述符代表。

```c++
void test5()
{	//cout不使用 << 运算符，可以调用put和write函数打印
	cout.put('a');	//等价于cout << 'a' ;
	cout.put('\n');
    
	char str[] = "hello,world";
	cout.write(str, sizeof(str));
    
	int x = 0x61626364;
    // 将整数的内存字节当作字符输出
	cout.write((char*)&x, sizeof(x)) << endl;
}
```

**cerr 与 log 的区别：**

它们俩都是标准错误流，区别在于 cerr 不经过缓冲区，直接向终端输出信息，而 clog 中的信息是存放在缓冲区的，缓冲区满后或遇到 endl 向终端输出。



### 输出缓冲区

输出缓冲区内容刷新的意思是：输出缓冲区的内容写入到真实的输出设备或者文件。

如下几种情况会导致输出缓冲区内容被刷新：

1. 程序正常结束（有一个收尾操作就是清空缓冲区）；
2. 缓冲区满（包含正常情况和异常情况）；
3. 使用操纵符显式地刷新输出缓冲区，如：`endl`、`flush`、`ends`(没有刷新功能)；
4. 使用 `unitbuf` 操纵符设置流的内部状态；
5. 输出流与输入流相关联，此时在读输入流时将刷新其关联的输出流的输出缓冲区。

#### 演示缓冲区满

```c++
void test7(){
	// 在 Ubuntu18.04 上演示
	for(size_t idx = 0; idx < 1024; ++idx){
		cout << 'a';
	}
	sleep(5);
	cout << 'b';
}
```

#### 使用操纵符

endl: 用来完成换行，并刷新缓冲区

ends: 在输入后加上一个空字符，但是没有刷新缓冲区（这个需要注意，很多书上说可以刷新缓冲区）

flush: 用来直接刷新缓冲区的

unitbuf: 在每次执行完写操作后都刷新输出缓冲区

nounitbuf: 让流回到正常的缓冲方式

```c++
void test8()
{
    std::cout << "hello, world!" << std::endl;    //立刻换行输出
    std::cout << "hello, Garen";             // 不确定什么时候会输出
    //sleep(5);
    std::cout << "hello, Ashe" << std::ends;
    std::cout << "hello, Master Yi" << std::flush;
    std::cout << std::unitbuf << "hello, Wukong" << std::nounitbuf;
}
```

### 输出流与输入流相关联

当一个输入流被关联到一个输出流时，任何试图从输入流读取数据的操作都会先刷新关联的输出流。标准库将 cout 和 cin 关联在一起，可以测试

```c++
void test9() {
    auto stream = cin.tie();
    cout << "stream:" << stream << endl;
    cout << "&cout:" << &cout << endl;
    cin.tie(nullptr); // 解除关联
}
```

**交互式系统通常应该关联输入流和输出流。这意味着所有输出，包括用户提示信息，都会在读操作之前被打印出来。**

用来关联流的操作是 tie:

```c++
ostream *tie () const; //返回指向绑定的输出流的指针。
ostream *tie (ostream *os); //将os指向的输出流绑定的该对象上，并返回上一个绑定的输出流指针。
```



## 文件 IO

所谓“文件”，通常指存储在 **外部存储介质** 上的数据集合。数据以文件形式存储在外部介质中，操作系统亦以文件为单位进行数据管理。要向外部介质存储数据时，必须先创建文件（通过文件名标识），才能执行数据写入操作。根据数据的组织形式，文件可分为 **文本文件（ASCII 文件）** 和 **二进制文件**。

**文件流** 是以 **外存文件** 为操作对象的数据传输通道。其中：

- **文件输入流** 实现数据从外存文件到内存的传输
- **文件输出流** 完成数据从内存到外存文件的传输

每个文件流都对应一个内存缓冲区。需注意的是，**文件流** 本身并非物理文件，而是以文件为操作对象的抽象数据流。对磁盘文件的所有 I/O 操作都必须通过文件流实现。

C++对文件进行操作的流类型有三个: `ifstream`（文件输入流）, `ofstream`（文件输出流）, `fstream`（文件输入输出流），他们的构造函数形式都很类似:

```c++
ifstream();
explicit ifstream(const char *filename, openmode mode = in);
explicit ifstream(const string &filename, openmode mode = in);

ofstream();
explicit ofstream(const char *filename, openmode mode = out);
explicit ofstream(const string &filename, openmode mode = out);

fstream();
explicit fstream(const char *filename, openmode mode = in|out);
explicit fstream(const string &filename, openmode mode = in|out);
```

**如果想将某个成员函数删除或者不提供，可以在类中将该函数设置为私有的（C++98）或者将该函数删除 = delete(C++11)**

根据不同的使用场景，C++ 文件操作支持多种打开模式。在 `GNU GCC 7.4` 的实现中，这些模式通过 `ios_base` 类中的 `openmode` 枚举类型定义，包含以下六种模式：

- **in**（输入模式）：允许读取文件内容。若文件不存在，则打开失败（`failbit` 被置位）
- **out**（输出模式）：允许写入文件内容。若文件不存在，则创建新文件；若文件已存在且未指定 `app/ate` 模式，默认覆盖原有内容
- **app**（追加模式）：所有的写入将始终发生在文件的末尾（等效于 `out | ate`）
- **ate**（初始末尾定位模式）：打开文件后立即将读写位置定位到文件末尾（后续操作可自由移动位置）
- **trunc**（截断模式）：若文件已存在，则清空其内容（文件大小归零）。必须与 `out` 模式同时使用
- **binary**（二进制模式）：禁用文本转换，直接以二进制形式读写数据



### 文件输入流（读入内存）

输入流写法步骤：

- 当文件不存在时，打开文件失败，`ifstream ifs` ，构造函数多种，需要选取
- 从文件读数据并输出到屏幕，`ifs >> word` ，常搭配 while，默认以空格分隔
- 从文件一次读取一行使用 `getline` 函数

使用说明：

```c++
//std::basic_ifstream类
class basic_ifstream : public std::basic_istream<CharT, Traits>
    
//构造函数，explicit 是防止隐式转换的
explicit basic_ifstream( const char* filename,
                         std::ios_base::openmode mode
                             = std::ios_base::in );//in输入
    
//将basic_ifstream<char>类类型定义别名为ifstream
typedef basic_ifstream<char>       ifstream;

-----------------------------------------------------------------------
#include <fstream>	//文件流头文件
using std::ifstream;//文件输入流数据类型
```





#### 读取文件内容 1.0

```c++
void test() {
    
    std::ifstream ifs("PointTmp.cc"); // 定义一个文件输入流对象，并调用构造函数
    if(!ifs.good())	{
        std::cerr << "ifstream is not good" << std::endl;
        return ;
    }

    std::string word; 
    //对于文件输入流而言，默认情况以空格为分隔符
    while(ifs >> word)  // 等价于 while(ifs >> word, !ifs.eof())
    {
        std::cout << word <<" ";;
    }
    std::cout << std::endl;

    ifs.close();//关闭文件
}
```



#### 读取文件内容 2.0

1.0 的版本上的文件内容读取，并不美观友好，我们试着让程序每读取一行打印一行内容。

```c++
void test2() {
    std::ifstream ifs("test1.cc");
    if(!ifs.good()) {
        std::cerr << "ifstream is not good" << std::endl;
        return ;
    }

    std::string line[80];
    size_t idx = 0;
    
    // 读取的每一行结果放在string数组中
    while(getline(ifs, line[idx])) 
    {
        std::cout << line[idx] << std::endl;;
        ++idx;
    }
    std::cout << std::endl;
    // 可以指定文件内容的某一行的输出
    std::cout << "line[42] = " << line[42] << std::endl;

    ifs.close(); // 关闭文件
}
```



#### 读取文件内容 3.0

这一次使用容器进行文件内容的存储，可以随机打印某一行内容。

```c++
void test3() {
   
    std::ifstream ifs("test1.cc");
    if(!ifs.good()){
        std::cerr << "ifstream is not good" << std::endl;
        return ;
    }

    //动态的扩容
    std::vector<std::string> vec;
    vec.reserve(60); // 预留空间

    std::string line;
    while(getline(ifs, line)) {
        // 将读取的一行内容添加到容器内
        vec.push_back(line); 
    }
    std::cout << std::endl;
    std::cout << "vec[42] = " << vec[42] << std::endl;

    ifs.close();//关闭文件
}
```



### 文件输出流（内存写出）

```c++
//std::basic_ofstream类
class basic_ofstream : public std::basic_ostream<CharT, Traits>
    
//构造函数，explicit 是防止隐式转换的
explicit basic_ofstream( const char* filename,
                         std::ios_base::openmode mode
                             = std::ios_base::out );//默认out输出
    
//将basic_ofstream<char>类类型定义别名为ofstream
typedef basic_ofstream<char>       ofstream;

-----------------------------------------------------------------------
#include <fstream>	//文件流头文件
using std::ofstream;//文件输出流数据类型
```

在 C++文件操作中，可以根据需求选择不同的文件打开模式。这些模式在 `GNU GCC 7.4` 的实现中是通过 `ios_base` 类中的 `openmode` 枚举类型定义的，主要包括以下六种模式：



根据不同的情况，对文件的读写操作，可以采用不同的文件打开模式。文件模式在 GNU GCC7.4 源码实现中，是用一个叫做 openmode 的枚举类型定义的，它位于 ios_base 类中。文件模式一共有六种，它们分别是:

- in: 输入，文件将允许做读操作；如果文件不存在，则打开失败（设置 `failbit`）
- out: 输出，允许写入文件内容；若文件不存在，则创建新文件；若文件已存在且为只读 `app` 或 `ate` 模式，默认覆盖原有内容。
- app: 追加，写入将始终发生在文件的末尾，隐含 `out` 模式特性。
- ate: 末尾，打开文件后立即定位到文件末尾，后续操作可以自由移动读写位置。
- trunc: 截断，若文件已存在，则清空其内容。必须与 `out` 模式配合使用。
- binary: 二进制，读取或写入文件的数据为二进制形式

#### 写入文件内容 1.0

```c++
void test()
{
    // 若文件不存在，则创建；若文件存在，清空文件
    std::ofstream ofs("today.log");
    if(!ofs.good()) {
        std::cerr << "ofstream is not good" << std::endl;
        return;
    }

    std::ifstream ifs("PointTmp.cc");
    if(!ifs.good()) {
        std::cerr << "ifstream is not good" << std::endl;
        return;
    }

    std::string line;
    while(getline(ifs, line)) //将数据从ifs读入内存
    {
        ofs << line << std::endl; //将数据从内存写入ofs
    }

    ofs.close();
    ifs.close();
}
```



#### 写入文件内容 2.0

每次在文件的末尾添加数据

```c++
void test(void) {
    // app: 追加，写入将始终发生在文件的末尾
	std::ofstream ofs("text1.txt", std::ios::app);
	if(!ofs){
		std::cerr << "ofstream error!" << std::endl;
		return;
	}
    // 输出当前位置指示器的值(单位是字节数)
	std::cout << ofs.tellp() << std::endl;	

	ofs << "that's new line" << std::endl;
	ofs.close();
}
```

随机存取：利用对文件指针的操作可以实现随机存取数据。对于文件指针的偏移操作涉及到三个值:

```c++
ios_base::beg // 文件开头
ios_base::cur // 当前指针位置
ios_base::end // 文件结尾位置
```



### 文件输入输出流

**对于文件输入输出流而言，当文件不存在的时候，就会报错，所以文件必须要存在**。

```c++
//std::basic_ofstream类
class basic_iostream;
    
//构造函数，explicit 是防止隐式转换的
explicit basic_iostream( std::basic_streambuf<CharT,Traits>* sb );
    
//将basic_ofstream<char>类类型定义别名为ofstream
typedef basic_iofstream<char>       iofstream;

-----------------------------------------------------------------------
#include <fstream>	// 文件流头文件
using std::iofstream; // 文件输入输出流数据类型
```



#### 文件输入输出 1.0

```c++
void test2()
{
    std::fstream fs("text.txt");
    if(!fs) {
        std::cerr << "fstream is not good" << std::endl;
        return;
    }

    //利用fs的输出特性，从键盘输入五个数据存到text.txt中
    int number = 0;
    for(size_t idx = 0; idx < 5; ++idx)
    {
        std::cin >> number;
        fs << number << " ";
    }

    // 使用 tellg / tellp 查看文件对象的位置
    // g = get    p = put
    std::cout << "fs.tellg() = " << fs.tellg() << std::endl;
    std::cout << "fs.tellp() = " << fs.tellp() << std::endl;

    // 文件流对象的偏移操作（文件指针的偏移）
    // seekp / seekg
    // fs.seekg(0);
    fs.seekg(0, std::ios::beg); //定位到开始位置

    //利用fs的输入特性，从文件text.txt中读出到屏幕上
    for(size_t idx = 0; idx < 5; ++idx){
        fs >> number;
        std::cout << number << " ";
    }
    std::cout << std::endl;
    fs.close();
}
```

注意：换行符也算一个字节，所以文件里共有 10 个字节的内容

#### 例子补充

每次在文件的末尾添加数据：

```c++
int test(void){
	std::ofstream ofs("text1.txt", std::ios::app);
	if(!ofs){
		cerr << "ofstream error!" << endl;
		return -1;
	}
	cout << ofs.tellp() << endl;
	ofs << "that's new line" << std::endl;
	ofs.close();
	return 0;
}
```

随机存取：利用对文件指针的操作可以实现随机存取数据。对于文件指针的偏移操作涉及到三个值:

```c++
ios_base::beg //文件开头
ios_base::cur //当前指针位置
ios_base::end //文件结尾位置
```

对于短文件，如果想要一次性拿到所有的数据，该怎么操作？

```C++
void test12(){
	ifstream ifs("a.txt");
	if(!ifs.is_open()){
		cerr << "ifstream open file error!\n";
		return;
	}
	ifs.seekg(std::ios_base::end);
	auto length = ifs.tellg();
	ifs.seekg(std::ios_base::beg);
	char *buff = new char[length + 1]();
	ifs.read(buff, length + 1);
	string content(buff, length + 1);
	cout << content << endl;
	delete [] buff;
}
```

复制文件：

复制文件应该用 `ios::binary`(二进制模式)，原因是使用二进制文件模式时，程序将数据从内存传递给文件，将不会发生任何隐藏的转换，而默认状态下是文本模式，复制的内容可能会发生改变。

```c++
void test13(){
	fstream in("a.txt", std::ios::in|std::ios::binary);
	if(!in.is_open()){
		cerr << "fstream open file error!\n";
		return;
	}
	fstream out("a.txt", std::ios::out|std::ios::binary);
	if(!out.is_open()){
		cerr << "fstream open file error!\n";
		return;
	}
	out << in.rdbuf();//流的重定向
	out.close();
	in.close();
}
```





## 字符串 IO

头文件 `include <sstream>`

### 字符串输出流

字符串输出流 `ostringstream`

数值转换为字符串并输出

```c++
std::string int_toString(int value){
    std::ostringstream oss;
    oss << value; // 将内存数据输出到字符串流中
    //字符串流调用str()函数，将int型转换为string型
    return oss.str();
}
void test(){
    std::string s1 = int_toString(10);
    std::cout << "s1 = " << s1 << std::endl;
}
```



### 字符串输入流

字符串输入流 `istringstream` 。举例：读配置文件

```c++
//const string &filename = "myconf.conf"; 
void readConfig(const std::string &filename)
{
    std::ifstream ifs(filename);
    if(!ifs){
        std::cerr << "open file " << filename << " error!" << std::endl;
        return;
    }

    std::string line;
    while(getline(ifs, line)) //从ifs读入到line中
    {
        std::istringstream iss(line);//将字符串流的数据输入到内存中
        std::string key;
        std::string value;
        //逗号表达式
        while(iss >> key >> value, !iss.eof()){	//从流中依次取出数据
            std::cout << key << "---->" << value << std::endl;//再打印数据
        }
    }
    ifs.close();
}
```



### 字符串输入输出流

字符串输入输出流 `stringstream`

将数字转为字符串输出，然后字符串输入到指定格式输出到屏幕。

```c++
void test2()
{
    int number1 = 10;
    int number2 = 20;

    stringstream ss;

    ss << "number1= " << number1 //从内存输入到字符串流
       << " ,number2= " << number2 << endl;
    
    string s1 = ss.str();
    cout << s1 << endl;

    string key;
    int value;
    while(ss >> key >> value)	//再从流当中取出数据
    {
        cout << key << "---->" << value << endl;
    }
}
```



提问：getline 与字符串流的搭配，读文件流为什么要搭配读字符串流使用呢？尤其是在读文件的时候，经常使用读文件读一行数据，再用读字符串对一行数据进行处理。

原因是读取磁盘文件里的数据速度不快，如果每次都是读取一行数据（读取磁盘）到内存里，再去处理数据，处理完数据后再读一行数据（读取磁盘）整个过程是很慢的。所以需要使用读字符串流接收数据，即从磁盘里读的数据先放在内存中，再对数据进行处理时直接从内存中拿取处理，分开读取与处理的工作，提高效率。
