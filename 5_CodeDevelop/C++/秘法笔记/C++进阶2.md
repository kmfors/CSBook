# C++ 输入输出流

C++的 I/O 发生在流中，**流**是字节序列。如果字节流是从设备（如键盘、磁盘驱动器、网络连接等）流向内存，这叫做**输入操作**。如果字节流是从内存流向设备（如显示屏、打印机、磁盘驱动器、网络连接等）这叫做**输出操作**。就C++程序而言， I/O操作可以简单地看作是从程序移进或移出字节，程序只需要关心是否正确地输出了字节数据，以及是否正确地输入了要读取字节数据，特定I/O设备的细节对程序员是隐藏的。

## 输入输出机制

C++的 I/O 发生在流中，**流**是字节序列。如果字节流是从设备（如键盘、磁盘驱动器、网络连接等）流向内存，这叫做**输入操作**。如果字节流是从内存流向设备（如显示屏、打印机、磁盘驱动器、网络连接等）这叫做**输出操作**。

就C++程序而言， I/O操作可以简单地看作是从程序移进或移出字节，程序只需要关心是否正确地输出了字节数据，以及是否正确地输入了要读取字节数据，特定I/O设备的细节对程序员是隐藏的。



## C++常用流类型

1.  对系统指定的标准设备的输入和输出。即从键盘输入数据，输出到显示器屏幕。这种输入输出称为标准的输入输出，简称**标准I/O**。
2.  以外存磁盘文件为对象进行输入和输出，即从磁盘文件输入数据，数据输出到磁盘文件。以外存文件为对象的输入输出称为文件的输入输出，简称**文件I/O**
3.  对内存中指定的空间进行输入和输出。通常指定一个字符数组作为存储空间（实际上可以利用该空间存储任何信息）。这种输入和输出称为字符串输入输出，简称**串I/O**。

C++标准库提供了一组丰富的具有输入/输出功能的流类型。常用流类如下:



<img src="https://pnglist.obs.cn-east-3.myhuaweicloud.com/typora/202312052126494.png" alt="image-20230327193638975" style="zoom:50%;" />

## 流类型之间的关系

`ios`是抽象基类，由它派生出`istream`类和`ostream`类，`iostream`类支持输入输出操作，`iostream`类是从`istream`类和`ostream`类通过多重继承而派生的类。类`ifstream`继承了类`istream`，类`ofstream`继承了类`ostream`，类`fstream`继承了类`iostream`.

<img src="https://pnglist.obs.cn-east-3.myhuaweicloud.com/typora/202312052126667.png" alt="image-20230327193756290" style="zoom:50%;" />

## 流的状态

IO操作与生俱来的一个问题是可能会发生错误，一些错误是可以恢复的，另一些是不可以的。在C++标准库中，用iostate来表示流的状态，不同的编译器iostate的实现可能不一样，不过都有四种状态：

- `badbit`表示发生**系统级的错误**，如不可恢复的读写错误。通常情况下一旦`badbit`被置位，流就无法再使用了。

- `failbit`表示发生**可恢复的错误**，如期望读取一个数值，却读出一个字符等错误。这种问题通常是可以修改的，流还可以继续使用。

- 当**到达文件的结束位置**时，`eofbit`和 `failbit`都会被置位。

- `goodbit`被置位表示**流未发生错误**。如果`badbit`、 `failbit`和`eofbit`任何一个被置位，则检查流状态的条件会失。

  

## 管理流的状态

```c++
bool bad() const; //若流的badbit置位，则返回true;否则返回false
bool fail() const; //若流的failbit或badbit置位，则返回true;
bool eof() const; //若流的eofbit置位，则返回true;
bool good() const; //若流处于有效状态，则返回true;
iostate rdstate() const; //获取流的状态
void setstate(iostate state); //设置流的状态
//clear的无参版本会复位所有错误标志位*(重置流的状态)
void clear(std::ios_base::iostate state = std::ios_base::goodbit);
```

- badbit，一种系统级别的错误，该种错误不能恢复，可以使用函数bad进行判断。
- failbit，是一种可以恢复的错误，可以重新置位流的状态，可以使用函数fail进行判断。
- eofbit，是一种异常，表示文件指针到达了文件末尾，可以使用函数eof进行判断。
- goodbit，是一种流的正常状态，可以使用函数good进行判断。

## 流的通用操作

输入输出流同时还提供了一些其他通用的操作:

```c++
//-------以下输入流操作-----------------------------------------------------------------------------
int_type get();//读取一个字符
istream & get(char_type & ch);

//读取一行数据
istream & getline(char_type * s, std::streamsize count, char_type delim ='\n');

//读取count个字节的数据
istream & read(char_type * s, std::streamsize count);

//最多获取count个字节，返回值为实际获取的字节数
std::streamsize readsome(char_type * s, std::streamsize count);

//读取到前count个字符或在读这count个字符进程中遇到delim字符就停止，并把读取的这些东西丢掉
istream & ignore(std::streamsize count = 1, int_type delim = Traits::eof());

//查看输入流中的下一个字符, 但是并不将该字符从输入流中取走
//不会跳过输入流中的空格、回车符; 在输入流已经结束的情况下，返回 EOF。
int_type peek();

//获取当前流中游标所在的位置
pos_type tellg();

//偏移游标的位置
basic_istream & seekg(pos_type pos);
basic_istream & seekg(off_type off, std::ios::seekdir dir);
//----以下为输出流操作----------------------------------------------------------------------------------
//往输出流中写入一个字符
ostream & put(char_type ch);

//往输出流中写入count个字符
ostream & write(const char_type * s, std::streamsize count);

//获取当前流中游标所在的位置
pos_type tellp();

//刷新缓冲区
ostream & flush();

//偏移游标的位置
ostream & seekp(pos_type pos);
ostream & seekp(off_type off, std::ios_base::seekdir dir);
```



## 缓冲区

**缓冲区**又称为缓存，它是内存空间的一部分。也就是说，在内存空间中预留了一定的存储空间，这些存储空间用来缓冲输入或输出的数据，这部分预留的空间就叫做缓冲区。缓冲区根据其对应的是输入设备还是输出设备，分为**输入缓冲区**和**输出缓冲区**。

### 为什么要引入缓冲区呢？

比如我们从磁盘里取信息，我们先把读出的数据放在缓冲区，计算机再直接从缓冲区中取数据，等缓冲区的数据取完后再去磁盘中读取，这样就可以减少磁盘的读写次数，再加上计算机对缓冲区的操作大大快于对磁盘的操作，故应用缓冲区可大大提高计算机的运行速度。又比如，我们使用打印机打印文档，由于打印机的打印速度相对较慢，我们先把文档输出到打印机相应的缓冲区，打印机再自行逐步打印，这时我们的CPU可以处理别的事情。

因此**缓冲区**就是一块内存区，它用在输入输出设备和CPU之间，用来缓存数据。**它使得低速的输入输出设备和高速的CPU能够协调工作**，避免低速的输入输出设备占用CPU，解放出CPU，使其能够高效率工作。

从上面的描述中，不难发现缓冲区向上连接了程序的输入输出请求，向下连接了真实的 I/O 操作。作为中间层，必然需要分别处理好与上下两层之间的接口，以及要处理好上下两层之间的协作。

### 缓冲区的类型

缓冲区分为三种类型：全缓冲、行缓冲和不带缓冲。

- 全缓冲：在这种情况下，当填满标准I/O缓存后才进行实际I/O操作。全缓冲的典型代表是对磁盘文件的读写。
- 行缓冲：在这种情况下，当在输入和输出中遇到换行符时，执行真正的I/O操作。这时，我们输入的字符先存放在缓冲区，等按下回车键换行时才进行实际的I/O操作。典型代表是键盘输入数据。
- 不带缓冲：也就是不进行缓冲，标准出错情况**cerr/stderr**是典型代表，这使得出错信息可以直接尽快地显示出来。

### 缓冲区的刷新

- 程序正常结束的时候，会刷新缓冲区。
- 缓冲区满的时候（在ubuntu1804上缓冲区的大小正好是1024）

<img src="https://pnglist.obs.cn-east-3.myhuaweicloud.com/typora/202312052126058.png" alt="image-20230327223856396" style="zoom: 80%;" />

查看cout源码：`/usr/include/c++/7/bits/ostream.tcc`  利用软件查看！



### 流缓冲区

在C++中，流的缓冲区之基类是定义在 头文件当中的 streambuf.在头文件 当中，定义着两个类: ios_base和 ios。ios_base是所有I/O类的祖先，提供了**状态信息**、**控制信息**、**内部存储**、**回调**等设施。ios继承自ios_base，额外提供了与streambuf的接口；同时允许多个ios对象绑定同一个streambuf对象。

由于ios_base没有提供与streambuf的接口，ios才是标准库内所有I/O类（模板）事实上的最近共同祖先。ios的成员函数rdbuf是读取和设置流对象（ios的对象）绑定缓冲区的成员函数，它有两个不同的重载形式，分别如下：

```c++
//返回与之关联的streambuf；如果没有，则返回nullptr
streambuf *rdbuf() const;
//重新设置streambuf
//如果有，与先前绑定的streambuf解绑，再绑定传入的streambuf；
//如果传入的是nullptr，则流对象不与任何缓冲区对象绑定
streambuf *rdbuf(streambuf * sb);
```

streambuf本身不可以直接创建对象，它是一个抽象类型；但它向下派生出了2个派生类型，如我们在类图中看到的filebuf和stringbuf，这两个类分别是作为文件流和字符串流的子对象的，可以直接创建对象，后面我们会再次看到。



## 流状态的使用

```c++
#include <iostream>
using std::cin;
using std::cout;
using std::endl;
void printStreamStatus(){
    cout << "cin.badbit = " << cin.bad() << endl
         << "cin.failbit = " << cin.fail() << endl
         << "cin.eofbit = " << cin.eof() << endl 
         << "cin.goodbit = " << cin.good() << endl;//为true，则说明是一个正常的流
}
void test(){
    int number = 0;
    printStreamStatus();
    cin >> number;
    printStreamStatus();
    //cin.clear(); 重置流的状态
    cout << "number = " << number << endl;
    printStreamStatus();
}
int main(void){
    test();
    return 0;
}
```

<img src="../asset/image-20231205212703985.png" alt="image-20231205212703985" style="zoom:80%;" />

**ignore 函数的使用：**很好的解决了缓冲区遗留字符的问题

```c++
//读取到前count个字符或在读这count个字符进程中遇到delim字符就停止，并把读取的这些东西丢掉
istream & ignore(std::streamsize count = 1, int_type delim = Traits::eof());
//举简单例子
cin.ignore(1024,'\n')	//对输入缓冲区内的前1024个字节内，如果遇到字符\n,就将\n之前的内容丢掉
    
//查看源码,等价于
#include <limits> //不要忘记加头文件
cin.ignore(std::numeric_limits<std::streamsize>::max(),'\n')//max()当前的缓冲区的最大值
```





# C++ 标准IO

标准输入流cin，标准输出流cout，cerr（标准错误输出，不带缓冲），clog（标准错误输出，带缓冲）

重置流的状态clear()函数，清空缓冲区ignore();

## 标准输入流

cin 标准输入流，它从标准输入设备(键盘)获取数据，程序中的变量通过 **流提取符>>** 从流中提取数据。

流提取符>>从流中提取数据时通常跳过输入流中的空格、tab键、换行符等空白字符。只有在输入完数据再按回车键后，该行数据才被送入键盘缓冲区，形成输入流，提取运算符>>才能从中提取数据。需要**注意**保证从流中读取数据能正常进行。

```c++
#include <iostream>
using std::cin;
using std::cout;
using std::endl;
void printStreamStatus(){
    cout << "cin.badbit = " << cin.bad() << endl
         << "cin.failbit = " << cin.fail() << endl
         << "cin.eofbit = " << cin.eof() << endl 
         << "cin.goodbit = " << cin.good() << endl;//为true，则说明是一个正常的流
}
void test2(){
    int number = 0;
    //真值表达式while(cin)，要么是真的要么是假的
    //采用逗号表达式，取最后一个逗号的右边的对象
    while(cin >> number, !cin.eof()){	//eof()不等于ture,说明没有到末尾，如果等于true到达末尾跳出
        if(cin.bad()){ //如果发生不能恢复的系统级别错误
            cerr << "The stream is bad" << endl;
            return ;
        }
        else if(cin.fail()){//可以恢复的错误
            cin.clear();//重置流的状态
            cin.ignore(std::numeric_limits<std::streamsize>::max(),'\n');
            cout << "please input int data " << endl;
        }
        else{
            cout << "number = " << number << endl;
        }
    }
}
int main(void){
    test();
    return 0;
}
```



## 标准输出流

ostream类定义了3个全局输出流对象，即cout,cerr,clog，平常用的最多的就是cout,即**标准输出**。cout将数据输出到终端，它与标准C输出stdout关联。cerr是标准错误流（非缓冲），clog也是标准错误流（带缓冲）。注意：在C语言中，标准输入、标准输出和标准错误分别用0, 1, 2文件描述符代表。

```c++
void test5()
{	//cout不使用 << 运算符，可以调用put和write函数打印
	cout.put('a');	//等价于cout << 'a' ;
	cout.put('\n');
	char str[] = "hello,world";
	cout.write(str, sizeof(str));
	int x = 0x61626364;
	cout.write((char*)&x, sizeof(x)) << endl;
}
```

**cerr 与 log的区别：**

它们俩都是标准错误流，区别在于cerr不经过缓冲区，直接向终端输出信息，而clog中的信息是存放在缓冲区的，缓冲区满后或遇到endl向终端输出。



## 输出缓冲区

输出缓冲区内容刷新的意思是：输出缓冲区的内容写入到真实的输出设备或者文件。

如下几种情况会导致输出缓冲区内容被刷新：

1. 程序正常结束（有一个收尾操作就是清空缓冲区）；
2. 缓冲区满（包含正常情况和异常情况）；
3. 使用操纵符显式地刷新输出缓冲区，如：endl、flush、ends(没有刷新功能)；
4. 使用unitbuf操纵符设置流的内部状态；
5. 输出流与输入流相关联，此时在读输入流时将刷新其关联的输出流的输出缓冲区。

### 演示缓冲区满

```c++
void test7(){
	//在Ubuntu18.04上演示
	for(size_t idx = 0; idx < 1024; ++idx){
		cout << 'a';
	}
	sleep(5);
	cout << 'b';
}
```

### 使用操纵符

endl: 用来完成换行，并刷新缓冲区

ends: 在输入后加上一个空字符，但是没有刷新缓冲区（这个需要注意，很多书上说可以刷新缓冲区）

flush: 用来直接刷新缓冲区的

unitbuf: 在每次执行完写操作后都刷新输出缓冲区

nounitbuf: 让流回到正常的缓冲方式

```c++
void test8()
{
cout << "hello, world!" << endl;//立刻换行输出
cout << "hello, Garen";//不确定啥时候会输出
sleep(5);
cout << "hello, Ashe" << ends;
cout << "hello, Master Yi" << flush;
cout << unitbuf << "hello, Wukong" << nounitbuf;
}
```

### 输出流与输入流相关联

当一个输入流被关联到一个输出流时，任何试图从输入流读取数据的操作都会先刷新关联的输出流。标准库将cout和cin关联在一起，可以测试

```c++
void test9()
{
auto stream = cin.tie();
cout << "stream:" << stream << endl;
cout << "&cout:" << &cout << endl;
cin.tie(nullptr);//解除关联
}
```

**交互式系统通常应该关联输入流和输出流。这意味着所有输出，包括用户提示信息，都会在读操作之前被打印出来。**

用来关联流的操作是tie:

```c++
ostream *tie () const; //返回指向绑定的输出流的指针。
ostream *tie (ostream *os); //将os指向的输出流绑定的该对象上，并返回上一个绑定的输出流指针。
```



# C++文件IO

所谓“文件”，一般指存储在**外部介质**上数据的集合。一批数据是以文件的形式存放在外部介质上的。操作系统是以文件为单位对数据进行管理的。要向外部介质上存储数据也必须先建立一个文件（以文件名标识），才能向它输出数据。根据文件中数据的组织形式，可分为**ASCII文件**和**二进制文件**。

**文件流**是以**外存文件**为输入输出对象的数据流。**文件输入流**是从外存文件流向内存的数据，**文件输出流**是从内存流向外存文件的数据。每一个文件流都有一个内存缓冲区与之对应。**文件流**本身不是文件，而只是以文件为输入输出对象的流。若要对磁盘文件输入输出，就必须通过文件流来实现。

`include <fstream>` 

C++对文件进行操作的流类型有三个: ifstream（文件输入流）, ofstream（文件输出流）, fstream（文件输入输出流），他们的构造函数形式都很类似:

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

**如果想将某个成员函数删除或者不提供，可以在类中将该函数设置为私有的（C++98）或者将该函数删除=delete(C++11)**

根据不同的情况，对文件的读写操作，可以采用不同的文件打开模式。文件模式在GNU GCC7.4源码实现中，是用一个叫做openmode的枚举类型定义的，它位于ios_base类中。文件模式一共有六种，它们分别是:

- in: 输入，文件将允许做读操作；如果文件不存在，打开失败
- out: 输出，文件将允许做写操作；如果文件不存在，则直接创建一个
- app: 追加，写入将始终发生在文件的末尾
- ate: 末尾，读操作始终发生在文件的末尾
- trunc: 截断，如果打开的文件存在，其内容将被丢弃，其大小被截断为零
- binary: 二进制，读取或写入文件的数据为二进制形式



## 文件输入流（读入内存）

### 输入流使用说明

输入流写法步骤：

- 当文件不存在时，打开文件失败，`ifstream ifs` ，构造函数多种，需要选取
- 从文件读数据并输出到屏幕，`ifs >> word` ，常搭配while，默认以空格分隔
- 从文件一次读取一行使用getline函数

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





### 读取文件内容1.0

```c++
#include <string>
#include <iostream>
#include <fstream>	//文件流头文件
using std::string;
using std::ifstream;//文件输入流数据类型
using std::cout;
using std::endl;
using std::cerr;


void test()
{
    //对于文件输入流而言，如果文件不存在，那么就会报错；如果文件存在，才可以进行后续的操作
    ifstream ifs("PointTmp.cc");//定义一个文件输入流对象，并调用构造函数
    if(!ifs.good())	//出现错误退出
    {
        cerr << "ifstream is not good" << endl;
        return ;
    }

    string word; //定义一个字符串
    //对于文件输入流而言，默认情况以空格为分隔符
    while(ifs >> word)  //等价于while(ifs >> word,!ifs.eof())
    {
        cout << word <<" ";;
    }
    cout << endl;

    ifs.close();//关闭文件
}


int main(int argc, char **argv)
{
    test();
    return 0;
}


```

<img src="https://pnglist.obs.cn-east-3.myhuaweicloud.com/typora/202312052127336.png" alt="image-20230328195315473" style="zoom: 67%;" />



### 读取文件内容2.0

1.0的版本上的文件内容读取，并不美观友好，我们试着让程序每读取一行打印一行内容。

```c++
#include <string>
#include <iostream>
#include <fstream>
using std::string;
using std::ifstream;
using std::cout;
using std::endl;
using std::cerr;

void test2(){
    ifstream ifs("test1.cc");
    if(!ifs.good()){
        cerr << "ifstream is not good" << endl;
        return ;
    }

    string line[80];
    size_t idx = 0;
    
    while(getline(ifs, line[idx]))//读取每一行的结果放在string数组中
    {
        cout << line[idx] << endl;;
        ++idx;
    }
    cout << endl;
    //可以指定文件内容的某一行的输出
    cout << "line[42] = " << line[42] << endl;

    ifs.close();//关闭文件
}

int main(int argc, char **argv){
    test2();
    return 0;
}


```

<img src="https://pnglist.obs.cn-east-3.myhuaweicloud.com/typora/202312052127747.png" alt="image-20230328201228654" style="zoom: 67%;" />



### 读取文件内容3.0

这一次使用容器进行文件内容的存储，可以随机打印某一行内容。

```c++
#include <iostream>
#include <fstream>
#include <string>
#include <vector>

using std::cout;
using std::endl;
using std::cerr;
using std::cin;
using std::ifstream;
using std::string;
using std::vector;

void test3(){
   
    ifstream ifs("test1.cc");
    if(!ifs.good()){
        cerr << "ifstream is not good" << endl;
        return ;
    }

    //动态的扩容
    vector<string> vec; //创建一个string类型的容器
    vec.reserve(60);//预留空间
    string line;
    
    while(getline(ifs, line)){
        vec.push_back(line); //将读取的一行内容添加到容器内
    }
    cout << endl;
    //可以对容器进行下标读取元素的操作
    cout << "vec[42] = " << vec[42] << endl;

    ifs.close();//关闭文件
}

int main(int argc, char **argv){
    test3();
    return 0;
}
```





## 容器 vector

vector扩容原理

当vector中元素的个数与空间的大小相同的时候，`size() == capacity()`会按照`2 * size()`进行扩容，然后将老的空间中的元素拷贝到新的空间’然后再将老的空间去进行回收（包括元素与内存都需要清理），然后再将新的元素插入到新的空间的后面。

```c++
#include <iostream>
#include <vector>

using std::cout;
using std::endl;
using std::vector;

void printCapacity(const vector<int> &con)
{
    //vector扩容原理
    //当vector中元素的个数与空间的大小相同的时候，size()==capacity()
    //会按照2 * size()进行扩容，然后将老的空间中的元素拷贝到新的空间
    //然后再将老的空间去进行回收（包括元素与内存都需要清理），然后
    //再将新的元素插入到新的空间的后面
    cout << "con.size() = " << con.size() << endl
         << "con.capacity() = " << con.capacity() << endl;
}

void test()
{
    vector<int> vec;
    vec.reserve(20);
    printCapacity(vec);

    cout << endl;
    vec.push_back(1);
    printCapacity(vec);

    cout << endl;
    vec.push_back(2);
    printCapacity(vec);

    cout << endl;
    vec.push_back(3);
    printCapacity(vec);

    cout << endl;
    vec.push_back(4);
    printCapacity(vec);

    cout << endl;
    vec.push_back(5);
    printCapacity(vec);

    cout << endl;
    vec.push_back(6);
    printCapacity(vec);

    cout << endl;
    vec.push_back(7);
    printCapacity(vec);

    cout << endl;
    vec.push_back(5);
    printCapacity(vec);

    cout << endl;
    vec.push_back(5);
    printCapacity(vec);
}

int main(int argc, char **argv)
{
    test();
    return 0;
}


```





## 文件输出流（内存写出）

### 输出流使用说明

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



根据不同的情况，对文件的读写操作，可以采用不同的文件打开模式。文件模式在GNU GCC7.4源码实现中，是用一个叫做openmode的枚举类型定义的，它位于ios_base类中。文件模式一共有六种，它们分别是:

- in: 输入，文件将允许做读操作；如果文件不存在，打开失败
- out: 输出，文件将允许做写操作；如果文件不存在，则直接创建一个
- app: 追加，写入将始终发生在文件的末尾
- ate: 末尾，读操作始终发生在文件的末尾
- trunc: 截断，如果打开的文件存在，其内容将被丢弃，其大小被截断为零
- binary: 二进制，读取或写入文件的数据为二进制形式

### 写入文件内容1.0

```c++
#include <iostream>
#include <fstream>
#include <string>

using std::cout;
using std::endl;
using std::cerr;
using std::cin;
using std::ofstream;
using std::ifstream;
using std::fstream;
using std::string;

void test()
{
    //文件输出流而言，当文件不存在的时候，就创建文件
    //当文件存在的时候，文件输出流会将文件清空
    ofstream ofs("wd.txt");
    if(!ofs.good()) //文件流打不开就报错
    {
        cerr << "ofstream is not good" << endl;
        return;
    }

    ifstream ifs("PointTmp.cc");
    if(!ifs.good()) //文件流打不开就报错
    {
        cerr << "ifstream is not good" << endl;
        return;
    }

    string line;
    while(getline(ifs, line)) //将数据从ifs读入内存
    {
        ofs << line << endl; //将数据从内存写入ofs
    }

    ofs.close();
    ifs.close();
}
int main(void){
    test();
    return 0;
}
```

运行结果：wd.txt写入内容



### 写入文件内容2.0

每次在文件的末尾添加数据

```c++
int test(void){
	std::ofstream ofs("text1.txt", std::ios::app); //app: 追加，写入将始终发生在文件的末尾
	if(!ofs){
		cerr << "ofstream error!" << endl;
		return -1;
	}
	cout << ofs.tellp() << endl;	//输出当前位置指示器的值(单位是字节数)
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



## 文件输入输出流fstream

### 输入流使用说明

**对于文件输入输出流而言，当文件不存在的时候，就会报错，所以文件必须要存在**。

```c++
//std::basic_ofstream类
class basic_iostream;
    
//构造函数，explicit 是防止隐式转换的
explicit basic_iostream( std::basic_streambuf<CharT,Traits>* sb );
    
//将basic_ofstream<char>类类型定义别名为ofstream
typedef basic_iofstream<char>       iofstream;

-----------------------------------------------------------------------
#include <fstream>	//文件流头文件
using std::iofstream;//文件输入输出流数据类型
```



### 文件输入输出1.0

```c++
#include <iostream>
#include <fstream>
#include <string>

using std::cout;
using std::endl;
using std::cerr;
using std::cin;
using std::ofstream;
using std::ifstream;
using std::fstream;
using std::string;

void test2()
{
    //对于文件输入输出流而言，当文件不存在的时候，就会报错，所以文件必须要存在
    fstream fs("text.txt");
    if(!fs)
    {
        cerr << "fstream is not good" << endl;
        return;
    }

    //利用fs的输出特性，从键盘输入五个数据存到text.txt中
    int number = 0;
    for(size_t idx = 1; idx != 6; ++idx)
    {
        cin >> number;
        fs << number << " ";
    }

    //使用tellg/tellp查看文件对象的位置
    //g = get    p = put
    cout << "fs.tellg() = " << fs.tellg() << endl;//二者都一样，没有区别
    cout << "fs.tellp() = " << fs.tellp() << endl;

    //文件流对象的偏移操作（文件指针的偏移）
    //seekp/seekg
    /* fs.seekg(0); */
    fs.seekg(0,std::ios::beg);//将文件位置器置为开始位置

    //利用fs的输入特性，从文件text.txt中读出到屏幕上
    for(size_t idx = 1; idx != 6; ++idx){
        fs >> number;
        cout << number << " ";
    }
    fs.close();
}

int main(void){
    test2();
    return 0;
}
```

注意：换行符也算一个字节，所以文件里共有10个字节的内容
<img src="https://pnglist.obs.cn-east-3.myhuaweicloud.com/typora/202312052127780.png" alt="image-20230328212324641" style="zoom: 67%;" />



## 文件的模式

根据不同的情况，对文件的读写操作，可以采用不同的文件打开模式。文件模式在GNU GCC7.4源码实现中，是用一个叫做openmode的枚举类型定义的，它位于ios_base类中。文件模式一共有六种，它们分别是:

in: 输入，文件将允许做读操作；如果文件不存在，打开失败

out: 输出，文件将允许做写操作；如果文件不存在，则直接创建一个

app: 追加，写入将始终发生在文件的末尾

ate: 末尾，读操作始终发生在文件的末尾

trunc: 截断，如果打开的文件存在，其内容将被丢弃，其大小被截断为零

binary: 二进制，读取或写入文件的数据为二进制形式

---



简单读文件可以用ifstream的相关方法>>，简单写文件可以用ofstream的相关方法<<.

每次读取文件中的一行数据:

```c++
void test(){
	ifstream ifs("test.txt");
	if(!ifs.good()){
		cerr << ">> ifstream open file error!\n";
		return;
}
	string line;
	vector<string> vec;
	while(getline(ifs, line){
		//cout << line << endl;
		vec.push_back(line);
	}
	ifs.close();
	ofstream ofs("a.txt");
	if(!ofs.good()){
		cerr << ">> ofstream open file error!\n";
		return;
	}
	for(auto &elem : vec){
		ofs << elem << '\n';
	}
	ofs.close();
}
```

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

复制文件应该用ios::binary(二进制模式)，原因是使用二进制文件模式时，程序将数据从内存传递给文件，将不会发生任何隐藏的转换，而默认状态下是文本模式，复制的内容可能会发生改变。

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





# C++字符串IO

头文件 `include <sstream>`



## 字符串输出流ostringstream

数值转换为字符串并输出

```c++
#include <iostream>
#include <sstream>
#include <string>
#include <fstream>

using std::cout;
using std::endl;
using std::cerr;
using std::ostringstream;
using std::istringstream;
using std::stringstream;
using std::string;
using std::ifstream;


string int_toString(int value){
    ostringstream oss;
    oss << value; //将内存数据输出到字符串流中
    //字符串流调用str()函数，将int型转换为string型
    return oss.str();
}
void test(){
    string s1 = int_toString(10);
    cout << "s1 = " << s1 << endl;
}


int main(void){
    test();
    return 0;
}
```



## 字符串输入流istringstream

举例：读配置文件，结合ifstream进行

```c++
//const string &filename = "myconf.conf";//string("myconf.conf")
void readConfig(const string &filename)
{
    ifstream ifs(filename);
    if(!ifs){
        cerr << "open file " << filename << " error!" << endl;
        return;
    }

    string line;
    while(getline(ifs, line)) //从ifs读入到line中
    {
        istringstream iss(line);//将字符串流的数据输入到内存中
        string key;
        string value;
        //逗号表达式
        while(iss >> key >> value, !iss.eof()){	//从流中依次取出数据
            cout << key << "---->" << value << endl;//再打印数据
        }
    }
    ifs.close();
}

void test3()
{
    readConfig("myconf.conf");
}
```



## 字符串输入输出流stringstream

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



## 提问：getline与字符串流的搭配

读文件流为什么要搭配读字符串流使用呢？尤其是在读文件的时候，经常使用读文件读一行数据，再用读字符串对一行数据进行处理。

原因是读取磁盘文件里的数据速度不快，如果每次都是读取一行数据（读取磁盘）到内存里，再去处理数据，处理完数据后再读一行数据（读取磁盘）整个过程是很慢的。所以需要使用读字符串流接收数据，即从磁盘里读的数据先放在内存中，再对数据进行处理时直接从内存中拿取处理，分开读取与处理的工作，提高效率。



# 移动语义

## 概念理解

可以取地址的是左值，不能取地址的就是右值。**右值短暂的存在于栈上**。

右值包括：临时对象、匿名对象、字面值常量

const左值引用可以绑定到左值与右值上面。正因如此，也就无法区分传进来的参数是左值还是右值。

右值引用只能绑定到右值，不能绑定到左值。所以可以区分出传进来的参数到底是左值还是右值，进而可以区分。

字面值常量：10   是个右值

字符串常量："hello"   是个左值



```c++
String s3 = "hello";
String s4 = String("world");
```

移动语义的背景：

`String("world")` 是一个临时对象，临时对象开辟出一块空间，s4深拷贝后，临时对象开辟的空间立马销毁。但是如果临时对象开辟的空间直接转移给s4，s4也不用深拷贝了，这不就省下开销了吗，对吧。

那如何将临时对象开辟的空间转移给s4呢？那肯定先要识别临时对象是个右值还是右值，如果是右值就进行操作。

结论：非const的左值引用是不能绑定右值的；而const的左值引用既可以绑定左值，也可以绑定右值，但是无法区分左值、右值

C++98困境：不能完成通过代码区分左值与右值

C++11提出概念：右值引用

```c++
int &&rref = 10;
int &&rref = a ;//不可以，报错
```

右值引用可以绑定到右值上，但是不能绑定左值。



## 右值引用

右值引用的作用是识别出右值，但是其本身是一个左值。但是右值引用可不可以是一个右值呢？

左值：可以进行取地址的就是左值。
右值：不能进行取地址的就是右值。右值包括：临时变量、临时对象、匿名变量、匿名对象、字面值常量（10）

左值引用：int &ref;左值引用可以绑定到左值，不能绑定到右值。可以识别左值。
const左值引用：既可以绑定到左值，也可以绑定到右值，所以，const左值引用不能区分出左值与右值。
右值引用：int &&rref;右值引用可以绑定到右值，但是不能绑定到左值。可以识别右值。

右值引用的特点是：可以识别右值，可以绑定到右值。但是右值引用本身既可以是左值，也可以是右值。比如：当我们将右值引用作为函数参数的时候，在移动构造函数与移动赋值函数中就是**左值**，但是如果将函数的返回类型设置为右值引用的时候，那么右值引用本身就是一个**右值**了。



## 移动构造函数

将拷贝构造函数以及赋值运算符函数称为具有复制控制语义的函数。
将移动构造函数以及移动赋值运算符函数称为具有移动语义的函数。

1、移动语义的函数优先于具有拷贝语义的函数的执行
2、具有移动语义的函数如果不写的话，编译器是不会自动生成，必须手写

```c++
String(String &&rhs)
: _pstr(rhs._pstr)//将左值指向右值的空间
{
	cout << "String(String &&)" << endl;
	rhs._pstr = nullptr; //右值空间被转移后，使命完成，为避免共同指向销毁，右值指向为空
}
```



## 移动赋值运算符函数

```c++
String &operator=(String &&rhs)
{
	cout << "String &operator=(String &&)" << endl;
	if(this != &rhs)//1、自移动
	{
		delete [] _pstr;//2、释放左操作数
		_pstr = nullptr;
		_pstr = rhs._pstr;//3、浅拷贝，指向临时开辟的空间
		rhs._pstr = nullptr; //改变临时的指向，防止临时销毁后空间也跟着销毁
	}
	return *this;//4、返回*this
}
```



## std::move 函数

将左值转换为右值，在内部其实上是做了一个强制转换，static_cast<T &&>(lvaule)

```c++
//s1是左值，现在将s1转换成右值，右值s1 = s1
s1 = std::move(s1);
cout << "s1 = " << s1 << endl;// 执行为空
//在移动赋值运算函数中，s1指向nullptr，那右边的s1也就为空了，再去给s1赋值，那其实大家都是空指针了嘛，所以打印为空
```

几个问题注意一下：

1、`String("world") = String("world")` 这里不存在自复制，因为这里虽然字符串一样，但是确实两个不一样的临时对象，且这样做无意义

2、自移动或者自复制这一步操作还是要有的，就是为了避免出现 `s1 = std::move(s1) ` 这样的操作。



# 编译器优化技术RVO

RVO(Return Value Optimization)

当函数需要返回一个对象的时候，如果自己创建一个临时对象用户返回，那么这个临时对象会消耗一个构造函数(Constructor)的调用、一个复制构造函数的调用(Copy Constructor)以及一个析构函数(Destructor)的调用的代价。

而如果稍微做一点优化，就可以将成本降低到一个构造函数的代价

http://blog.csdn.net/zwvista/article/details/6845020

http://www.cnblogs.com/xkfz007/articles/2506022.html



# 资源管理

C 的难点背景：

```c++
void UseFile(char const* fn){
	FILE* f = fopen(fn, “r”);  // 获取资源
    // 使用资源
    if (!g()) { fclose(f); return; }//关闭文件
    // ...   
    if (!h()) { fclose(f); return; }//关闭文件
    // ...  
    fclose(f);           // 释放资源
}

```

用于释放资源的代码需要在不同的位置重复书写多次。如果再加入异常处理，fclose(f) 情况会变得更加复杂。

为了应对这种问题，c++11提出了 RAII技术(Resource Acquisition Is Initialization)。

RAII 是一种由 C++创造者 Bjarne Stroustrup 提出的， **利用栈对象生命周期管理程序资源（包括内存、文件句柄、锁等）的技术**。



## RAII 思想

利用栈对象的生命周期管理资源（内存资源、文件描述符、文件、锁）。

#### 四个基本特征

1、在构造函数中初始化资源，或者称为托管资源
2、在析构函数中释放资源
3、提供若干访问资源的方法（比如：读写文件）
4、一般不允许复制或者赋值。（对象语义）

**对象语义**：不允许复制或者赋值。

- 1、将拷贝构造函数或者赋值运算符函数=delete
- 2、将拷贝构造函数或者赋值运算符函数设置为私有的
- 3、可以使用继承的思想，将基类拷贝构造函数与赋值运算符函数删除（设为私有），让派生类继承基类

**值语义**：可以进行复制或者赋值。

使用 RAII 时，一般在资源获得的同时构造对象， 在对象生存期间，资源一直保持有效；对象析构时，资源被释放。

关键：要保证资源的释放顺序与获取顺序严格相反。

```c++
#include <iostream>
#include <string>

using std::cout;
using std::endl;
using std::string;

class SafeFile{
public:
    //在构造函数中初始化资源(托管资源)
    SafeFile(FILE *fp)
    : _fp(fp){
        cout << "SafeFile(FILE *)" << endl;
    }

    //提供若干访问资源的方法（对文件进行写操作）
    //用C的方法向文件中写数据
    void write(const string &msg){
        fwrite(msg.c_str(), 1, msg.size(), _fp);
    }

    //在析构函数中释放资源
    ~SafeFile(){
        cout << "~SafeFile()" << endl;
        if(_fp){
            fclose(_fp);
            cout << "fclose(_fp)" << endl;
        }
    }

private:
    FILE *_fp;
};

void test(){
    string msg = "hello,world\n";
    SafeFile sf(fopen("wd.txt", "a+"));//sf是栈对象
    sf.write(msg);
    //栈对象随着声明周期的结束，会自动的调用析构函数，完成对文件的close操作
}

int main(int argc, char **argv){
    test();
    return 0;
}


```



## RAII 实现

```c++
#include <iostream>

using std::cout;
using std::endl;

class Point
{
public:
    Point(int ix = 0, int iy = 0)
    : _ix(ix)
    , _iy(iy)
    {
        cout << "Point(int = 0, int = 0)"  << endl;
    }

    void print() const
    {
        cout << "(" << _ix
             << ", " << _iy
             << ")" << endl;
    }

    ~Point()
    {
        cout << "~Point()" << endl;
    }
private:
    int _ix;
    int _iy;
};

template <typename  T> //T加相当于任何数据类型有
class RAII
{
public:
    //1、在构造函数中初始化资源
    RAII(T *data)
    : _data(data)
    {
        cout <<"RAII(T *)" << endl;
    }

    //2、在析构函数中释放资源
    ~RAII()
    {
        cout << "~RAII()" << endl;
        if(_data)
        {
            delete _data;
            _data = nullptr;
        }
    }

    //3、提供若干访问资源的方法
    T *operator->()
    {
        return _data;
    }

    T &operator*()
    {
        return *_data;
    }

    T *get() const
    {
        return _data;
    }

    //重置
    void reset(T *data)
    {
        if(_data)
        {
            delete _data;
            _data = nullptr;
        }
        _data = data;
    }

    //4、不允许复制或者赋值
    RAII(const RAII &rhs) = delete;
    RAII &operator=(const RAII &rhs) = delete;
private:
    T *_data;
    /* int *_data; */
    /* double *_data; */
    /* string *_data; */
    /* Point *_data; */
};

void test()
{
    //利用栈对象的生命周期结束的时候，会执行析构函数，完成资源的清理
    RAII<int> raii(new int(10));//栈对象raii，在尖括号中声明T是int类型,托管堆申请的空间

    cout << endl;
    //pt本身不是一个指针，但是pt的使用就与一个指针完全一样
    //可以托管资源（new int(10),new Point(1, 2)）,但是可以
    //不用直接执行delete就可以将资源进行回收,资源的回收靠的
    //就是栈对象pt的销毁执行析构函数
    RAII<Point> pt(new Point(1, 2));//栈对象pt，在尖括号中声明T是Point类型，托管堆申请的空间
    pt->print();
    (*pt).print();
    /* pt.operator->()->print(); */
}

int main(int argc, char **argv)
{
    test();
    return 0;
}


```



# 智能指针

C++11提供了以下几种智能指针,位于头文件`<memory>`，它们都是类模板.

智能指针(Smart Pointer)：

- 是存储指向动态分配（堆）对象指针的类
- 在面对异常的时候格外有用，因为他们能够确保正确的销毁动态分配的对象
- RAII类模拟智能指针，可以利用一个对象在离开一个域中会调用析构函数的特性，在构造函数中完成初始化，在析构函数中完成清理工作，将需要操作和保护的指针作为成员变量放入RAII中。

## auto_ptr的使用

```c++
#include <iostream>
#include <memory>

using std::cout;
using std::endl;
using std::auto_ptr;

void test(){
    int *pInt = new int(10);
    auto_ptr<int> ap(pInt);
    cout << "*ap = " << *ap << endl;
    cout << "*pInt = " << *pInt << endl;

    cout << endl;
    auto_ptr<int> ap2(ap);
    cout << "*ap2 = " << *ap2 << endl;
    cout << "*ap = " << *ap << endl; //报错，Segmentation fault (core dumped)
}

int main(){
    test();
    return 0;
}
```

看一眼关于auto_ptr的源码

```c++
auto_ptr(auto_ptr& __a) __STL_NOTHROW : _M_ptr(__a.release()) {}
template <class _Tp> class auto_ptr {
private:
  _Tp* _M_ptr;
public:
    //auto_ptr ap2(ap);
    //auto_ptr &__a = ap
    auto_ptr(auto_ptr& __a)
    : _M_ptr(__a.release())
    {
        
    }
    
    _Tp* release(){	//将ap托管的空间转移给了ap2
    	_Tp* __tmp = _M_ptr;
    	_M_ptr = 0; //ap自身指向为空
    	return __tmp;
  	}
    
};
```

auto_ptr的缺点：一般都不使用这种指针

表面上执行的是拷贝操作，但是底层己经将右操作数 ap 所托管的堆空间的控制权己经交给了左操作数ap2，并且ap底层的数据成员己经被置位空了，该拷贝操作存在隐患，所以该拷贝操作是存在缺陷的。**在拷贝构造或赋值操作时,会发生所有权的转移**。



## unique_ptr 的使用

明确表明是独享所有权的智能指针，**无法进行复制与赋值**。

保存指向某个对象的指针，当它本身被删除释放的时候，会使用给定的删除器释放它指向的对象。

**具有移动语义，可以作为容器的元素（容器中可以存unique_ptr 的类型）**

```c++
void test()
{
	unique_ptr<int> up(new int(10));
	cout << "*up = " << *up << endl;
	cout << "up.get() = " << up.get() << endl;//获取托管的指针的值
	
    cout << endl ;
 /* unique_ptr<int> up2(up);//error,独享资源的所有权,不能进行拷贝复制 */
	
    cout << endl ;
	unique_ptr<int> up3(new int(10));
 /* up3 = up;//error,不能进行赋值操作 */
//独享所有权的智能指针，对托管的空间独立拥有。在语法层面就将auto_ptr的缺陷摒弃掉了，具有对象语义
/*------------------------------------------------------------------------------------------*/
	
    cout << endl ;
	unique_ptr<int> up4(std::move(up));//通过移动语义move转移up的所有权，初始化智能指针up4
	cout << "*up4 = " << *up4 << endl;
	cout << "up4.get() = " << up4.get() << endl;
	
    cout << endl ;
	unique_ptr<Point> up5(new Point(3, 4));
	vector<unique_ptr<Point>> vect;
    vect.push_back(std::move(up5));//不能传左值，但是可以传右值
    //或者直接构建右值，显式调用unique_ptr的构造函数
	vect.push_back(unique_ptr<Point>(new Point(1, 2)));
	
}
```



## shared_ptr 的使用

是一个**共享所有权的智能指针，允许对象之间进行复制或者赋值**，展示出来的就是值语义。使用**引用计数**的观点。当对象之间进行复制或者赋值的时候，引用计数会加+1，当最后一个对象销毁的时候，引用计数减为0的时候，会负责回收托管的空间。



``` c++
void test()
{
    shared_ptr<int> sp(new int(10));
    cout << "*sp = " << *sp << endl;
    cout << "sp.get() = " << sp.get() << endl;
    cout << "sp.use_count() = " << sp.use_count() << endl; //引用计数

    cout << endl;
    shared_ptr<int> sp2 = sp;//拷贝复制ok
    cout << "*sp = " << *sp << endl;
    cout << "*sp2 = " << *sp2 << endl;
    cout << "sp.get() = " << sp.get() << endl;
    cout << "sp2.get() = " << sp2.get() << endl;
    cout << "sp.use_count() = " << sp.use_count() << endl;
    cout << "sp2.use_count() = " << sp2.use_count() << endl;

    cout << endl;
    shared_ptr<int> sp3(new int(20));
    sp3 = sp;//赋值ok
    cout << "*sp = " << *sp << endl;
    cout << "*sp2 = " << *sp2 << endl;
    cout << "*sp3 = " << *sp3 << endl;
    cout << "sp.get() = " << sp.get() << endl;
    cout << "sp2.get() = " << sp2.get() << endl;
    cout << "sp3.get() = " << sp3.get() << endl;
    cout << "sp.use_count() = " << sp.use_count() << endl;
    cout << "sp2.use_count() = " << sp2.use_count() << endl;
    cout << "sp3.use_count() = " << sp3.use_count() << endl;

    cout << endl;
    vector<shared_ptr<Point>> vec; //作为容器的元素
    shared_ptr<Point> sp4(new Point(10, 20));
    vec.push_back(sp4); //可以传左值
    vec.push_back(std::move(sp4)); //也可以传右值
    vec.push_back(shared_ptr<Point>(new Point(1, 3)));//类名声明一个临时对象指针
}

```

<img src="https://pnglist.obs.cn-east-3.myhuaweicloud.com/typora/202312052127583.png" alt="image-20230412153509786" style="zoom:67%;" />

**问题：存在循环引用？**

该智能指针在使用的时候，会使得引用计数增加，从而会出现**循环引用**的问题？

两个shared_ptr智能指针互指，导致引用计数增加，不能靠对象的销毁使得引用计数变为0，从而导致内存泄漏。。。

```c++
#include <iostream>
#include <memory>

using std::cout;
using std::endl;
using std::shared_ptr;

class Child;//类的前向声明

class Parent{
public:
    Parent(){
        cout << "Parent()" << endl;
    }

    ~Parent(){
        cout << "~Parent()" << endl;
    }
    shared_ptr<Child> parent;
};

class Child{
public:
    Child(){
        cout << "Child()" << endl;
    }

    ~Child(){
        cout << "~Child()" << endl;
    }
    shared_ptr<Parent> child;
};

void test()
{
    shared_ptr<Parent> pParent(new Parent());
    shared_ptr<Child> pChild(new Child());
    cout << "pParent.use_count() = " << pParent.use_count() << endl;
    cout << "pchild.use_count() = " << pChild.use_count() << endl;

    cout << endl;
    pParent->parent = pChild;
    pChild->child = pParent;
    cout << "pParent.use_count() = " << pParent.use_count() << endl;
    cout << "pchild.use_count() = " << pChild.use_count() << endl;
    //此代码存在内存泄漏
}

int main(int argc, char **argv){
    test();
    return 0;
}
```

<img src="https://pnglist.obs.cn-east-3.myhuaweicloud.com/typora/202312052127767.png" alt="image-20230412155106117"  />

循环引用导致的原因是shared_ptr是强引用的智能指针，每一次指向都会使得引用计数进行累加。但是单独的靠智能指针这种栈对象的销毁又不能将引用计数减为0，所以就会出现循环引用而不能完成空间回收的问题。就需要有一种弱引用的智能指针，可以执行空间，但是不会将引用计数累加，就需要让shared_ptr与weak_ptr结合使用。

解决循环引用的办法是使得其中一个改为weak_ptr，不会增加引用计数，这样可以使用对象的销毁而打破引用计数减为0的问题

修改： shared_ptr pChild;改为 weak_ptr pChild;即可解决循环引用的问题

```c++
parentPtr->pParent = childPtr;//sp = sp
childPtr->pChild = parentPtr;//wp = sp,weak_ptr不会导致引用计数加1
```



## weak_ptr的使用

与shared_ptr相比，weak_ptr 被称为弱引用的智能指针。weak_ptr不会导致引用计数增加，但是它不能直接获取资源，必须通过lock函数从wp提升为sp，从而判断共享的资源是否已经销毁。

```c++
bool expired() const;
//如果引用计数use_count()==0,那么该函数的返回值就是true；如果use_count() != 0,那么该函数的返回值就是false
```

```c++
td::shared_ptr<T> lock() const;
//将weak_ptr提升为一个share_ptr，然后再来判断shared_ptr，进而知道weak_ptr指向的空间还在还是不在。
```

shared_ptr能够用解引用运算符去查看自己所指向的空间还在不在，但是weak_ptr则没有这个东西，怎么去查看呢？

```c++
void test()
{
    /* weak_ptr<int> wp(new int(20)); //error,没有普通类型的构造函数 */
    shared_ptr<int> sp(new int(10));
    weak_ptr<int> wp;
    wp = sp;
    cout << "sp.use_count() = " << sp.use_count() << endl;
    cout << "wp.use_count() = " << wp.use_count() << endl;

    //如果引用计数为0，不在返回true; 否则返回false
    //若被管理对象已被删除则为 true ，否则为 false
    bool flag = wp.expired(); 
    cout << "wp.expired() = " << flag << endl;
}

//那为了试验一下expired函数的作用，需要将sp给释放掉，但智能指针的释放肯定不是delete
//而是随着作用域的的生命周期结束而释放掉

//第一种方式
void test1()
{
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

```c++
//第2种方式
void test2()
{
    weak_ptr<int> wp;
    {
        shared_ptr<int> sp(new int(10));
        wp = sp;
        cout << "sp.use_count() = " << sp.use_count() << endl;
        cout << "wp.use_count() = " << wp.use_count() << endl;

        shared_ptr<int> sp2 = wp.lock();
        if(sp2)
        {
            cout << "提升成功" << endl;
            cout << "*sp2 = " << *sp2 << endl;
            cout << "wp.use_count() = " << wp.use_count() << endl;
        }
        else{
            cout << "提升失败，wp指向的空间已经被回收了" << endl;
        }
    }
    cout << "--------离开作用域后，share_ptr释放----------" << endl;

    shared_ptr<int> sp2 = wp.lock();
    if(sp2)
    {
        cout << "提升成功" << endl;
        cout << "*sp2 = " << *sp2 << endl;
    }
    else{
        cout << "提升失败，wp指向的空间已经被回收了" << endl;
    }
}
```



## 删除器

问题引入：

```c++
void test(){
    string msg = "hello,world\n";
    //FILE *fp = fopen("wd.txt", "a+");
    unique_ptr<FILE> up(fopen("wd.txt", "a+"));
    fwrite(msg.c_str(), 1, msg.size(), up.get()); //get函数可以从智能指针获取到裸指针
    //flose(up.get());
}
```

以上代码我们运行的时候很显然会报错，原因在于，unique_ptr是一个智能指针，虽然fp看上去是一个指针，但实际上并没有做new操作。当程序结束后，虽然程序结束了但并没有去做关闭文件的操作，导致报错。

看一下unique_ptr 的定义

<img src="https://pnglist.obs.cn-east-3.myhuaweicloud.com/typora/202312052128983.png" alt="image-20230515204303850" style="zoom: 80%;" />

原来unique_ptr在使用的时候默认用的是 `std::default_delete<T>` 删除器，内部默认设置的delete函数。所以在程序结束时，我们是无法对一个未new的文件进行delete操作的。

<img src="../asset/image-20231205212820398.png" alt="image-20231205212820398" style="zoom:80%;" />



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
   	//传入一个删除器FILECloser, 是仿照默认删除器写的
    unique_ptr<FILE, FILECloser> up(fopen("wd.txt", "a+"));
    fwrite(msg.c_str(), 1, msg.size(), up.get());
}
```

默认的删除器执行的是delete或者delete []的形式，所以针对FILE或者socket这种创建的资源，需要重新改写删除器，使用 fclose 或 close。

其他智能指针除了 share_ptr 具有删除器的操作，其他都没有。但 share_ptr 的删除器与 unique_ptr的删除器不太一样。

<img src="https://pnglist.obs.cn-east-3.myhuaweicloud.com/typora/202312052128555.png" alt="image-20230515210458901" style="zoom:80%;" />

**share_ptr的删除器是在构造函数里传入的。那写一下关于share_ptr删除器的使用**

```c++
void test2(){
    //要想从类型调用到对象，可以直接显式使用构造函数，比如：使用无参构造函数
    string msg = "hello,world\n";
    //注意share_ptr构造函数传入的删除器是一个对象，类名使用括号为匿名对象,调用的是无参构造函数
   	share_ptr<FILE> sp(fopen("wh.txt", "a+"), FILECloser());
    fwrite(msg.c_str(), 1, msg.size(), sp.get());
}
```



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

3、share_ptr也会出现跟第一种一样的情况，只是说当用裸指针给share_ptr初始化时，引用计数是不会加1的，因为它们两个指针没有关系。所以我们要给另一个share_ptr初始化时，要用share_ptr指针取初始化。

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

5、注意函数的返回类型式裸指针也不行，也一样会出错。然而当我们将返回类型改为了share_ptr类型后，还是报错，这是为什么？
因为在return的时候我们的this也就是sp所在的对象给匿名托管了，两个不同的智能指针托管同一块空间必然报错。

<img src="https://pnglist.obs.cn-east-3.myhuaweicloud.com/typora/202312052128739.png" alt="image-20230516194513288" style="zoom:67%;" />

解决办法：使用shared_from_this

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



# 模板

1、简化程序，少写代码，维持结构的清晰，大大提高程序的效率。

2、解决强类型语言的**严格性**和**灵活性**之间的冲突。。

 2.1、带参数的宏定义(原样替换)

 2.2、函数重载(函数名字相同，参数不同)

 2.3、模板(将数据类型作为参数)



## 模板定义

```c++
template <typename T> //模板参数列表
template <class T>	//模板参数列表
```

注意：class与typename在此没有任何区别，完全一致。class出现的时间比较早，通用性更好一些，typename是后来才添加了，对于早期的编译器可能识别不了typename关键字。



## 模板类型

函数模板与类模板。通过参数实例化构造出具体的函数或者类，称为模板函数或者模板类。

函数模板和类模板的区别在于：

1，函数模板有自动类型推导，但是类模板没有自动类型推导，必须指定模板参数类型；

2，函数模板不可以有默认参数，但是类模板允许有默认参数，当类模板中有默认参数的时候，就可以把显示指定类型的参数去掉。



## 函数模板

template <模板参数表> //模板参数列表，**此处模板参数列表不能为空**

返回类型 函数名（参数列表）

 { //函数体 }

```c++
template <typename T>//模板参数列表
T add(T x, T y)
{
	cout << "T add(T, T)" << endl;
	return x + y;
}
```

 					模板参数推导 (在实参传递的时候进行推导)
函数模板------------------------------------------------------------------------模板函数
												实例化

https://gitee.com/demoCanon/cpp-gpt/blob/master/Project/week4/functionTemplate.cc



### 实例化：隐式实例化与显示实例化

```c++
cout << "add(ia, ib) = " << add(ia, ib) << endl;//隐式实例化，没有明确说明类型，T只能靠编译器推导

cout << "add(da, db) = " << add<double>(da, db) << endl;//显示实例化，显式写出参数类型T为double
```



### 函数模板、普通函数间的关系

1、函数模板与普通函数是可以进行重载的

2、普通函数优先于函数模板执行

3、函数模板与函数模板之间也是可以进行重载的



### 模板头文件与实现文件

模板**不能**写成头文件与实现文件形式(类型inline函数)，或者说不能将声明与实现分开，这样会导致编译报错。

分开可以编译，但是在链接的时候是有问题的。

建议在头文件给出模板的声明后，立即include引入实现文件

<img src="https://pnglist.obs.cn-east-3.myhuaweicloud.com/typora/202312052128112.png" alt="image-20230410171120223" style="zoom:67%;" />



### 模板的特化：偏特化与全特化

```c++
template <> //全部特化出来就是全特化
//因为模板的所有参数都以特殊形式写出来了，所以模板列表写为空
const char *add(const char *pstr1, const char *pstr2)
{
	size_t len = strlen(pstr1) + strlen(pstr2) + 1;
	char *ptmp = new char [len]();
	strcpy(ptmp, pstr1);
	strcat(ptmp, pstr2);
	return ptmp;
}
```

**注意：函数模板是支持全特化的，但是不同的编译器，不同的语法版本，可能不支持偏特化，不支持部分特化。**

### 函数模板的参数类型

1、类型参数，class T 这种就是类型参数

2、非类型参数 常量表达式，整型：bool/char/short/int/long/size_t,	注意：float/double这些就不是整型

https://gitee.com/demoCanon/cpp-gpt/blob/master/Project/week4/templatePara.cc#



### 成员函数模板

```c++
#include <iostream>

using std::cout;
using std::endl;

class Point{
public:
    Point(double dx = 0.0, double dy = 0.0)
    : _dx(dx)
    , _dy(dy)
    {
        cout << "Point(double,double)" << endl;
    }

    //普通函数与成员函数都可以设置为模板形式
    template <typename T = double>
    T func(){
        return (T)_dx;
    }

    ~Point(){
        cout << "~Point()" << endl;
    }

private:
    double _dx;
    double _dy;
};

void test(){
    Point pt(1.1, 2.2);

    cout << "pt.func() = " << pt.func<int>() << endl;//使用传入参数int

    cout << "pt.func() = " << pt.func() << endl;//使用类型参数默认值double
}


int main()
{
    test();
    return 0;
}


```





## 可变模板参数

可以表示0个到任意个参数，参数的个数不确定，参数的类型也不确定。可以类比printf进行理解。

https://gitee.com/demoCanon/cpp-gpt/blob/master/Project/week4/variTemplate.cc

<img src="https://pnglist.obs.cn-east-3.myhuaweicloud.com/typora/202312052129328.png" alt="image-20230410195807447" style="zoom:67%;" />

<img src="https://pnglist.obs.cn-east-3.myhuaweicloud.com/typora/202312052129718.png" alt="image-20230410195821495" style="zoom:67%;" />

<img src="https://pnglist.obs.cn-east-3.myhuaweicloud.com/typora/202312052129360.png" alt="image-20230410195837134" style="zoom:67%;" />

```c++
template<typename ...Args>
//Args对接收的众多参数进行打包
void Mylogger::info(const char *msg,Args ...args){
    char buff[1024] = {0};
    //sprintf可以自动识别可变模板参数args，后面紧跟...表示解包
    sprintf(buff,"%s  %s : %s : %d",msg,args...);//可以对args解包，但是必须匹配一个转换符
    _mycat.info(buff); 
}

//...代表多参数，##__VA_ARGS__代表可变参数宏
#define logError(msg,...) Mylogger::getInstance()->error(msg,##__VA_ARGS__)
#define logInfo(msg,...) Mylogger::getInstance()->info(msg,##__VA_ARGS__)
#define logWarn(msg,...) Mylogger::getInstance()->warn(msg,##__VA_ARGS__)
#define logDebug(msg,...) Mylogger::getInstance()->debug(msg,##__VA_ARGS__)
//我可以传递很多参数，
void test(){
    logError("This is an error message",__FILE__,__FUNCTION__,__LINE__);
    logInfo("This is an error message",__FILE__,__FUNCTION__,__LINE__);
    logWarn("This is an error message",__FILE__,__FUNCTION__,__LINE__);
    logDebug("This is an error message",__FILE__,__FUNCTION__,__LINE__);
    /* Mylogger<Tt,Tc>::destory(); */
}
```



## 类模板

```c++
template <typename T>
class Stack
{
private:
    T *_data;
    //int *_data;
    //double *_data;
    //Point *_data;
};
```

注意：**成员模版不能声明为虚函数**。写成模板相当于写了好多个函数，如果再将这些函数设计为虚函数，而虚函数会入虚表，那么都不知道模板可以推导出多少个函数，那么虚表就不知道分配多少空间。

模板是可以进行嵌套的。

```c++
#include <iostream>
#include <string>

using std::cout;
using std::endl;
using std::string;

//泛型编程
template <typename T = int, size_t sz = 10>
class Stack
{
public:
    Stack()
    : _top(-1)
    , _data(new T[sz]())
    {

    }
    ~Stack();
    bool empty() const;
    bool full() const;
    void push(const T &value);
    void pop();
    T top() const;
private:
    int _top;
    T *_data;
};

template <typename T, size_t sz> //头必须写出来，但是默认值前面已经给出，无需初始化
Stack<T, sz>::~Stack() //用类声明的时候也需要将模板参数显式亮出来
{
    if(_data)
    {
        delete [] _data;
        _data = nullptr;
    }
}

template <typename T, size_t sz>
bool Stack<T, sz>::empty() const
{
    return -1 == _top;
}

template <typename T, size_t sz>
bool Stack<T, sz>::full() const
{
    return _top == sz - 1;
}

template <typename T, size_t sz>
void Stack<T, sz>::push(const T &value)
{
    if(!full())
    {
        _data[++_top] = value;
    }
    else
    {
        cout << "The stack is full, cannot push any data" << endl;
        return;
    }
}

template <typename T, size_t sz>
void Stack<T, sz>::pop()
{
    if(!empty())
    {
        --_top;
    }
    else
    {
        cout << "The stack is empty, cannot pop any data" << endl;
        return;
    }
}

template <typename T, size_t sz>
T Stack<T, sz>::top() const
{
    return _data[_top];
}

void test()
{
    Stack<int> st;
    cout << "栈是不是空的 ?" << st.empty() << endl;
    st.push(1);
    cout << "栈是不是满的 ?" << st.full() << endl;

    for(size_t idx = 2; idx != 15; ++idx)
    {
        st.push(idx);
    }
    cout << "栈是不是满的 ?" << st.full() << endl;

    while(!st.empty())
    {
        cout << st.top() << "  ";
        st.pop();
    }
    cout << endl;
    cout << "栈是不是空的 ?" << st.empty() << endl;
}

void test2()
{
    Stack<string, 20> st;

    cout << "栈是不是空的 ?" << st.empty() << endl;
    st.push(string("aa"));
    cout << "栈是不是满的 ?" << st.full() << endl;

    for(size_t idx = 1; idx != 15; ++idx)
    {
        st.push(string(2, 'a' + idx));
    }
    cout << "栈是不是满的 ?" << st.full() << endl;

    while(!st.empty())
    {
        cout << st.top() << "  ";
        st.pop();
    }
    cout << endl;
    cout << "栈是不是空的 ?" << st.empty() << endl;
}

int main(int argc, char **argv)
{
    test2();
    return 0;
}
```

