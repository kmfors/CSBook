





# 数组

Q1：请描述数组的内存模型
	     连续的一片内存空间，并且这片内存空间被划分为大小相等的小空间

Q2：为什么要划分大小相等的小空间？ ---> 实现随机访问数组的元素
		 以及是根据什么划分的？        --->  数组只能存储同一种类型的数据

**Q3：为什么在大多数的语言当中，数组的索引都是从0开始的？**

​		设一维数组A[n]存放在n个连续的存储单元中，每个数组元素占一个存储单元（一个存储单元为C个连续字节）.如果数组元素A[0]的首地址是L，则A[1]的首地址是L+C，A[2]的首地址是L+2C，… …，依次类推寻址方式
​		**i_addr = base_addr + i * sizeof (elem_type)（从0开始）** 
​		**i_addr = L + i * C （从0开始）**

而若从1开始：**i_addr = base_addr + ( i - 1 ) * sizeof (elem_type)（从1开始）** |   **i_addr = L + ( i - 1 ) * C （从1开始）**则会进行一些不必要的计算；如果初地址不置用，从第二个地址用，会浪费一个空间内存。



## 数组的声明

数组的声明一般是：`元素类型 数组名[大小];`   `int arr[10];`

## 数组的初始化

数组的初始化一般有以下几种方式：

1. `int arr[10] = {1,2,3,4,5,6,7,8,9,10};`该数组长度为10，并为每一个下标元素赋予初始值
2. `int arr[10] = {1,2,3,4,5};`该数组长度为10，只为前5个下标元素赋予初始值
3. `int arr[10] = {0}; `该数组长度为10，默认将该数组中的每个元素赋予初始值0
4. `int arr[] = {1,2,3,4,5,6,7,8,9,10};`编译器会根据初始化的长度，决定数组的长度

> 注意：` int arr[10] = {1,2,3,4,5,6,7,8,9,10,11};`编译器报错，元素数量超出数组长度！



## 数组使用 sizeof 运算符

有些时候，数组的长度会由我们声明，我们可以知道这个数组的长度是多少。但对于一些不透明的数组我们反而不知道它们的长度是多少。但我们可以由数组的内存模型可以知晓，它们之间的关系

**数组A的长度 = 数组A的字节大小 / 数组A中单个元素的字节大小**

利用 **sizeof** 运算符我们可以得出某一数组的大小，再根据该数组的首元素的大小，求出数组长度

**A_Length = sizeof (A) / sizeof (A[0])**

可以定义一个宏函数，方便以后每次计算数组的长度：**#define  SIZE (A)  ( sizeof (A) / sizeof (A[0]) ）**

**A_Length = SIZE (A)**

> 注意：VS 不可直接定义 **int arr[0]** , 指定的下标数必须大于 0



## 多维数组

二维数组的逻辑内存模型如图：
![[C基础知识-9.png]]

但二维数组的内存空间是连续的（行优先存储） 如图：
![[C基础知识-10.png]]


二维数组的本质：元素为一维数组的数组

### 二维数组的初始化

1. `int matrix[3][4] = { {1,2,3,4},{2,2,3,4},{3,2,3,4} };` 正常赋值
2. `int matrix[3][4] = { {1,2,3,4},{2,2,3,4} };`  初始化只满足列数，其余元素初始化0
3. `int matrix[3][4] = { {1,2,3},{2,2,3},{3,2,3} };`  初始化只满足行数，其余元素初始化0
4. `int matrix[3][4] = { {1,2,3},{2,2,3} };`   初始化时既不满足行数也不满足列数，其余元素初始化0；**但根据下标取值时极易出错，地址连续的**
5. `int matrix[3][4] = { 1,2,3,4,5,6,7,8,9,10,11,12 };`  不推荐！特别容易搞混，尤其长度特别长的
6. `int matrix[3][4] = { 0 };`  初始化时所有元素初始化为 0
7. **`int matrix[][4] = { {1,2,3,4},{2,2,3,4},{3,2,3,4} };`**只能省略第 1 维度（多维数组本质上还是一维数组，后面的维度指示元素的类型）

### 指针和多维数组

`int zippo [4][2];`   由一维数组知，zippo为该数组首元素的地址，所以zippo的值与&zippo[0] 的值相同。zippp[0] 是一个占用 1 个 int 大小对象的地址，zippo则是一个占用 2个 int 大小对象的地址。 
`*zippo[0]` 则为 `zippo[0][0]` 的值，`*zippo` 则为`*zippo[0]`的值，`**zippo[0]` 则为`zippo[0][0]`的值，所以二维数组zippo需要解地址两次才能获得原始值，
![[C基础知识-11.png]]![[C基础知识-12.png]]



### 指向多维数组的指针

指向多维数组的指针称为 **数组指针**

### 二维数组的传参

**1.基本形式：二维数组在栈上分配，各行地址空间连续**

```c
	...
    int array[M][N];
　　//array[][N] ={{...},...,{...}}; is ok　　
　　//array[][] = {{...},...,{...}}; is wrong
	...   
	//使用array[m][n]

```

这种分配是在栈上进行的，能够保证所有元素的地址空间连续。这样，`arry[i][j]` 和`((array+i)+j)`是一样的，程序是知道`array+i`的i实际上偏移了
`i*N`个单位，这也导致了在二维数组`array[3][3]`中，使用下标`array[2][1]`和`array[1][4]`是访问的同一个元素，尽管后者的下标对于一个3*3矩阵来说是非法的，但这并不影响访问。

这种形式，无论是数组定义还是函数都不够泛用，两个维度在编译前就定好了，唯一可以做的就是把维度M、N声明为宏或者枚举类型，但这仍不能避免每次修改后都要重新编译。

**2.数组传参形式：二维数组在栈上分配，各行地址空间连续，函数参数使用指针形式**

当把这种**二维数组的指针直接作为参数传递时，数组名退化为指针，函数并不知道数组的列数**，N对它来说是不可见的，即使使用`*(*(array +i) +j)`，第一层解引用失败。这时，编译器会报warning，运行生成的文件会发生segment fault。那么，为了指导这个函数如何解引用，也就是人为地解引用，需要把这个二维数组的**首元素地址**传给函数，于是就变成了下面的形式：

```c
#include <stdio.h>
//1.测试二维数组，第一种方式 int func(int Card[][n])  OK的！
int func(int Card[][2]);
int main(void) {
    int Card[5][2] = { {0,1},{2,3},{4,5},{6,7},{8,9} };
    func(Card);
    return 0;
}
int func(int Card[][2]) {
    int n = 0;
    n = Card[0][1];
    printf("%d", n);
    return 0;
}
```



## 常量数组

格式为： **const int arr[6] = {1,2,3,4,5,6}**   arr里面的元素不会发生改变

数组的元素不能发生改变，作用：存放一些静态数据

rand() 直接调用产生随机数，缺点每次程序运行产生的随机数都一样

调用srand(seed)，播种seed的值为100，rand() 直接调用产生随机数，缺点每次程序运行产生的随机数都一样.看似没有效果



# 函数

## 函数

函数：封装一系列的语句

**一、函数的定义**

参数的传递（值传递）**C语言中只有值传递，没有引用传递**

值传递：在被调函数中是不能够修改主调函数的参数的

Q：如何在被调函数中修改主调函数中的参数？（传指针）

**1.当数组作为参数传递时会退化成指针**，而这个指针指向数组第一个元素的指针

缺点：丢失了类型信息，丢失了数组的长度

优点：可以避免大量复制数组，可以修改原数组中的元素，使函数调用更加灵活

所以函数参数数组的声明应该是`int a[],int n`额外再添加一个参数，这个参数传递数组的长度

2.二维数组作为指针传递时其格式为 `int a[][3],int n` 额外再添加一个参数，这个参数传递数组的行数。同时理解a是`int [*][3]`类型的指针，所以另一种的写法还可以是`int (*a)[3],int n`

Q：有没有一个办法可以传递一个列数不固定的数组？有  `int* arr[],int n`只不过这种是指针数组，这个数组里存储的是指针类型的元素，所以arr的类型为 `int**`

 返回值类型  ---> 返回值类型不能是数组类型

程序如何终止?
程序的开始，操作系统调用main函数；程序的终止，main函数的返回

Q：如何不在main中终止？   `void exit (int exit,code);`退出状态码，作用和main函数的返回值一致：0 代表正常退出、1 表示异常退出

局部变量与外部变量

- 局部变量：定义在函数里面的变量，默认自动存储期限，但可以通过关键字static改为静态存储期限
- 外部变量：文件作用域，从变量的定义开始，到文件的末尾

(1) 存储期限：

- 自动存储期限，存放在栈里面的数据，具有自动存储期限，变量的生命周期随着栈帧的入栈而开始 ，随着栈帧的出栈而消亡
- 静态存储期限：拥有“永久”的存储单元，在程序的整个执行期间都存在

如图所示：函数foo里调用自身，main函数调用后，局部变量未初始化，值默认随机，输出地址e534。而foo函数每一次的调用，又将重新声明局部变量所以这就造成了默认值固定，但地址是随机的
![[C基础知识-13.png]]



如图所示：局部变量前面添加 static ，生命周期改为静态存储期限。值虽然变化，但是其局部变量本身的地址不会发生变化。
![[C基础知识-14.png]]



在 C 语言中，函数由一个函数头和一个函数主体组成。下面列出一个函数的所有组成部分：

- **返回类型：**一个函数可以返回一个值。**return_type** 是函数返回的值的数据类型。有些函数执行所需的操作而不返回值，在这种情况下，return_type 是关键字 **void**。
- **函数名称：**这是函数的实际名称。函数名和参数列表一起构成了函数签名。
- **参数：**参数就像是占位符。当函数被调用时，您向参数传递一个值，这个值被称为实际参数。参数列表包括函数参数的类型、顺序、数量。参数是可选的，也就是说，函数可能不包含参数。
- **函数主体：**函数主体包含一组定义函数执行任务的语句。

**函数参数**

如果函数要使用参数，则必须声明接受参数值的变量。这些变量称为函数的**形式参数**。

形式参数就像函数内的其他局部变量，在进入函数时被创建，退出函数时被销毁。

当调用函数时，有两种向函数传递参数的方式：

|                           调用类型                           |                             描述                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| [传值调用](https://www.nowcoder.com/tutorial/10002/4eff7542c37245d69893e4245df2ff22) | 该方法把参数的实际值复制给函数的形式参数。在这种情况下，修改函数内的形式参数不会影响实际参数。 |
| [引用调用](https://www.nowcoder.com/tutorial/10002/918dd239f2e84cc79b9c3949f4951218) | 通过指针传递方式，形参为指向实参地址的指针，当对形参的指向操作时，就相当于对实参本身进行的操作。 |

默认情况下，C 使用**传值调用**来传递参数。一般来说，这意味着函数内的代码不能改变用于调用函数的实际参数。

**传值调用**：

向函数传递参数的**传值调用**方法，把参数的实际值复制给函数的形式参数。在这种情况下，修改函数内的形式参数不会影响实际参数。

默认情况下，C 语言使用*传值调用*方法来传递参数。一般来说，这意味着函数内的代码不会改变用于调用函数的实际参数。

```c
/* 函数定义 */
void swap(int x, int y)
{
   int temp;
 
   temp = x; /* 保存 x 的值 */
   x = y;    /* 把 y 赋值给 x */
   y = temp; /* 把 temp 赋值给 y */
 
   return;
}
-------------------------------------------------------------------------------------------------------------------
include <stdio.h>
 
/* 函数声明 */
void swap(int x, int y);
 
int main ()
{
   /* 局部变量定义 */
   int a = 100;
   int b = 200;
 
   printf("交换前，a 的值： %d\n", a );
   printf("交换前，b 的值： %d\n", b );
 
   /* 调用函数来交换值 */
   swap(a, b);
 
   printf("交换后，a 的值： %d\n", a );
   printf("交换后，b 的值： %d\n", b );
 
   return 0;
}
```

**引用调用**：

通过引用传递方式，**形参为指向实参地址的指针**，当对形参的指向操作时，就相当于对实参本身进行的操作。

传递指针可以让多个函数访问指针所引用的对象，而不用把对象声明为全局可访问

```c
/* 函数定义 */
void swap(int *x, int *y)
{
   int temp;
   temp = *x;    /* 保存地址 x 的值 */
   *x = *y;      /* 把 y 赋值给 x */
   *y = temp;    /* 把 temp 赋值给 y */
 
   return;
}
--------------------------------------------------------------------------------
#include <stdio.h>
 
/* 函数声明 */
void swap(int *x, int *y);
 
int main ()
{
   /* 局部变量定义 */
   int a = 100;
   int b = 200;
 
   printf("交换前，a 的值： %d\n", a );
   printf("交换前，b 的值： %d\n", b );
 
   /* 调用函数来交换值
    * &a 表示指向 a 的指针，即变量 a 的地址
    * &b 表示指向 b 的指针，即变量 b 的地址
   */
   swap(&a, &b);
 
   printf("交换后，a 的值： %d\n", a );
   printf("交换后，b 的值： %d\n", b );
 
   return 0;
}
```





## C 判断

|                             语句                             |                             描述                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| [if 语句](https://www.nowcoder.com/tutorial/10002/ec2d054a7b8b487d9ef4cd8d3c91da18) |  一个 **if 语句** 由一个布尔表达式后跟一个或多个语句组成。   |
| [if...else 语句](https://www.nowcoder.com/tutorial/10002/9a2175a9a24e4590850e85574fb5ea1f) | 一个 **if 语句** 后可跟一个可选的 **else 语句**，else 语句在布尔表达式为假时执行。 |
| [嵌套 if 语句](https://www.nowcoder.com/tutorial/10002/0030605fe48849aa8fa30c00b2ef1df3) | 您可以在一个 **if** 或 **else if** 语句内使用另一个 **if** 或 **else if** 语句。 |
| [switch 语句](https://www.nowcoder.com/tutorial/10002/eb3008215f7b47e6a823362dc8229c50) |   一个 **switch** 语句允许测试一个变量等于多个值时的情况。   |
| [嵌套 switch 语句](https://www.nowcoder.com/tutorial/10002/700b32239a014410a74f36ff283f3c69) |  您可以在一个 **switch** 语句内使用另一个 **switch** 语句。  |

条件运算符(三元运算符)

**条件运算符 ? :**，可以用来替代 **if...else** 语句。它的一般形式如下：

```
Exp1 ? Exp2 : Exp3;
```

其中，Exp1、Exp2 和 Exp3 是表达式。请注意，冒号的使用和位置。

? 表达式的值是由 Exp1 决定的。如果 Exp1 为真，则计算 Exp2 的值，结果即为整个 ? 表达式的值。如果 Exp1 为假，则计算 Exp3 的值，结果即为整个 ? 表达式的值

## C 循环

### 循环类型

C 语言提供了以下几种循环类型。点击链接查看每个类型的细节。

|                           循环类型                           |                             描述                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| [while 循环](https://www.nowcoder.com/tutorial/10002/63b25b48a25549a4b811c4903dcdf053) | 当给定条件为真时，重复语句或语句组。它会在执行循环主体之前测试条件。 |
| [for 循环](https://www.nowcoder.com/tutorial/10002/80a4cd76e41143b6912466e60829118a) |        多次执行一个语句序列，简化管理循环变量的代码。        |
| [do...while 循环](https://www.nowcoder.com/tutorial/10002/ee919d49db034bb0bec48ef7c1164a4d) |  除了它是在循环主体结尾测试条件外，其他与 while 语句类似。   |
| [嵌套循环](https://www.nowcoder.com/tutorial/10002/acfd1344821642e188489bf84ab3f8e0) | 您可以在 while、for 或 do..while 循环内使用一个或多个循环。  |

### 循环控制语句

循环控制语句改变你代码的执行顺序。通过它你可以实现代码的跳转。

C 提供了下列的循环控制语句。点击链接查看每个语句的细节。

|                           控制语句                           |                             描述                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| [break 语句](https://www.nowcoder.com/tutorial/10002/74770c2c0fa349769e50ad892fc25fc5) | 终止**循环**或 **switch** 语句，程序流将继续执行紧接着循环或 switch 的下一条语句。 |
| [continue 语句](https://www.nowcoder.com/tutorial/10002/d9fc09376deb4728a19e0458736911a9) |  告诉一个循环体立刻停止本次循环迭代，重新开始下次循环迭代。  |
| [goto 语句](https://www.nowcoder.com/tutorial/10002/817a2b56d82c4ad8b0f006002f7a6855) | 将控制转移到被标记的语句。但是不建议在程序中使用 goto 语句。 |







# 指针

## 指针基础

计算机的最小寻址单位：字节
变量的地址：变量第一个字节的地址。   指针，简单来说就是地址

指针变量：存放地址的变量，有时候也把指针变量称为指针。

Q：指针变量只是存放变量的首地址，那怎么通过指针访问指针指向的对象呢？

​		声明时需要指名指针的基础类型（指针指向对象的类型） 

指针的基本操作：取地址符号&   解地址符号 *　　int *p = &i     i 是直接访问（访问一次内存） *p 是直接访问（访问两次内存）

野指针问题：野指针是指未初始化的指针，或者指向未知区域的指针。若对野指针进行解引用运算，会导致未定义的行为！

指针变量的赋值：

- 使用取地址运算符，`p = &i;` 
- 通过另外一个指针变量赋值，`p = q;`

注意：`p = q` 是将q保存的地址赋值给p ,或者p指向q的引用地址

指针作为参数传递：可以在被调函数中访问和修改主调函数中的参数

## 指针和数组

当指针指向数组元素时，可以通过指针的算数运算访问数组的其他元素

- 指针加上一个整数，其结果为指针
- 指针减去一个整数，其结果为指针
- 两个指针相减，这两个指针应该指向同一个数组的元素！其结果为整数

指针的比较运算（两个指针应该指向同一个数组的元素）

- `p < q  <==> p - q < 0`
- `p == q  <==> p - q <== 0`
- `p > q  <==> p - q > 0`

用指针处理数组
![[C基础知识-15.png]]



*和++ / -- 的组合：

- `*p++ , *(p++)` ：表达式的值为 `*p`，副作用为 p 自增
- `(*p)++` ：表达式的值为 `*p`，副作用为 `(*p)` 自增
- `*++p , *(++p)` ：表达式的值为 `*(p+1)`，副作用为 p 自增
- `++*p , ++(*p)` ：表达式的值为 `(*p)+1`，副作用为 `p` 自增

数组名可以作为指针使用
![[C基础知识-16.png]]


指针也可以作为数组名使用（可以对指针进行 [ ]  运算）
![[C基础知识-17.png]]

## 字符指针

字符指针：指向字符型数据的指针变量。每个字符串在内存中都占用一段连续的存储空间，并有唯一确定的首地址。即将字符串的首地址赋值给字符指针，可让字符指针指向一个字符串

```c
char *ptr = "Hello";//将保存在常量存储区的"Hello"的首地址赋值给ptr
与
char *ptr;
ptr = "Hello";//是等价的，注意不能理解为将字符串赋值给ptr
```



# 字符串

C语言中没有专门的字符串类型，C语言中的字符串依赖字符数组存在！（C语言的字符串是一个逻辑类型）

字符串字面值：在编译时，编译器会将两个相邻字符串合并成一个
相邻：两个字符串字面值之间仅有空白字符

**C没有专门用于存储字符串的变量类型，字符串都被存储在char类型的数组中。数组由连续的存储单元组成，字符串中的字符被存储在相邻的存储单元中**

**数组是同类型数据元素的有序序列**

**以`'\0'`作为结束标志，作为字符串结束的标志。`\0`作为一个特殊字符，它的ASCII值为0，不是'0'**



## 字符串字面量

字符串字面量 (string literal) 是用一对双引号括起来的字符序列：

```c
"To C or not to C, that's the question."
```

字符串常量也被称为字符串常量。它作为一种静态存储类型， 在程序开始运行时被分配地址，一直存在到程序结束，它本身只代表首元素的地址。



## 字符串变量

字符串变量也称为字符数组

字符串变量的初始化方式，列举

- `char name[10] = "Allen";`  （浪费内存空间）
- `char name[5] = "Allen";` （结尾没有 \0 ，不是个字符串）
- **`char name[] = "Allen";`（推荐使用！）**

**字符数组**的初始化和**字符指针**的初始化：

`name1`是个数组，赋值是个初始化式，元素可以修改；`name2`是个指针，赋值是个字符串字面值为常量，不能够修改。如图
![[C基础知识-18.png]]


本质的原因：二者在内存中存储区域不同，**字符数组的字符串存储在全局数据区或栈区**，**字符指针指向的字符串存储在常量区**。
【全局数据区和栈区 的字符串有读取和写入的权限，而 常量区字符串只有读取权限，没有写入权限，这就导致了字符数组在定义后可读取和修改每个字符而 字符指针（字符串常量） 一旦定义后便不可修改，对它的赋值都是错误的（可整体赋值）】



## 读写字符串



### printf 和 puts 写字符串

转换说明 %s 允许 printf 写字符串。如：

```c
char str[] = "Are we having fun yet?";
printf("%s\n", str);
```

printf 函数会逐个写字符串中的字符，直到遇到空字符。(如果字符数组中没有空字符， printf 会一直写，直到在内存的某个位置找到空字符为止)。

如果只想显示字符串的一部分，可以使用转换说明 %.ps ，这里 p 是要显示的字符数量，如：

```c
printf("%.6s\n", str);
```

字符串跟数一样，可以在指定的字段内显示。转换说明 **%ms 会在大小为 m 的字段内显示字符串**。(对于超过 m 个字符的字符串， printf 会显示整个字符串，不会截断)。如果字符串少于 m 个字符，则会在字段内右对齐。如果要左对齐，可以在 m 前加一个减号。

C 函数库还提供了puts 函数，用来输出字符串

```c
puts(str);
```

> 注意：puts(会在字符串后面自动添加换行符 \n )   `puts(str)` 等价于 `printf("%s\n",str)`



### scanf 和 gets 读字符串

转换说明 %s 允许 scanf 把字符串读入字符数组：

```c
scanf("%s", str);
```

在 scanf 函数调用中，不需要在str 前添加运算符 & ，因为 str 是数组名，编译器在把它传递给函数时会把它当作指针来处理。

读字符串时， **scanf 会跳过前面的空白字符，然后读入字符并存储到 str 中，直到遇到空白字符为止**。scanf 始终会在字符串末尾存储一个空字符。

**用 scanf 读入的字符串永远不会包含空白字符**。因此， scanf 通常不会读入一整行输入。空格符和制表符会使 scanf 停止读入。

为了读入一整行输入，我们可以使用 gets 函数。类似于 scanf ， gets 会把读入的字符存储到**字符数组**中，然后存储一个空字符。然而，这两个函数在其他方面有很大的不同：

- gets 不会跳过前面的空白字符 (scanf 会跳过)
- gets 会一直读入直到遇到换行符才停止 (scanf 会在任意空白字符处停止)
- gets 函数会忽略掉换行符，用空字符代替换行符

> 注意事项：scanf 和 gets 函数都不会检查数组是否越界。因此，使用这两类函数是不安全的。
>
> 二者的共同的缺陷是：不会检查数组是否越界！
>
> 加深印象：键盘先输入，读取的字符进入输入缓冲区，等待写入。scanf进行写入操作，若遇到空白字符，scanf便认为写入结束！



## C 字符串库

一些编程语言提供了一些运算符，可以对字符串进行复制、比较、拼接等操作，但 C 语言的运算符根本无法操作字符串。**C 语言是把字符串当作数组来处理，它们都不能用运算符进行复制和比较**

幸运的是，C 语言提供了丰富的字符串函数。这些函数的原型都包含在 **<string.h>** 头文件中。

在下面的例子中，都假设 str、str1 和 str2 都是用作字符串的字符数组，介绍几种常用的字符串函数

### strlen 函数

strlen 函数：获取给定字符串的长度

```c
size_t strlen( const char *str );//strlen 函数返回字符串 str 的长度：str中第一个空字符之前的字符个数
```

```c
int len;
len = strlen("abc"); /* len is now 3 */
len = strlen(""); /* len is now 0 */
strcpy(strl, "abc");
len = strlen(strl); /* len is now 3 */
```

注意：`const char *str 与  char * const str`的区别

<img src="https://gitee.com/demoCanon/pic-bed/raw/master/img/202302041553943.png" alt="image-20230204155256748" style="zoom:67%;" />

### strcpy 函数

`strcpy(str1,str2)`：字符串赋值函数，C语言中字符串赋值不能直接使用 ’=’ 进行赋值，必须使用`strcpy(str1,str2)`函数进行赋值。

```c
char *strcpy(char *str1, const char *str2);
```

strcpy 函数把字符串 str2 复制给字符串 s1。准确地讲，是 strcpy 把 str2 指向的字符串复制到 str1 指向的数组中。 也就是说， strcpy 函数把 str2 中的字符复制到str1 中直到遇到s2 中 的第一个空字符为止（该空字符也需要复制）。

strcpy 函数返回str1 （即指向目标字符串的指针）。**这一过程不会改变str2 指向的字符 串，因此将其声明为const** 。

strcpy 函数的存在弥补了不能使用赋值运算符复制字符串的不足。比如，我们可以这样调用 strcpy 函数：

```c
strcpy(str2, "abcd");
strcpy(str1, str2);
```

大多数情况下我们会忽略strcpy 函数的返回值，但有时候strcpy 函数调用是一个更大的表达式的一部分，这时其返回值就比较有用了。例如，可以把一系列strcpy 函数调用连起来：

```c
strcpy(str1, strcpy(str2, "abcd"));
```

在 strcpy(str1, str2) 的调用中，strcpy 函数无法 检查str2 指向的字符串的大小是否真的适合str1 指向的数组。假设str1 指向的字符串长度为 ，如果str2 指向的字符 串中的字符数不超过 ，那么复制操作可以完成。但是，如 果str2 指向更长的字符串，那么结果就无法预料了。（因为 strcpy 函数会一直复制到第一个空字符为止，所以它会越过 str1 指向的数组的边界继续复制。）**不会检查数组是否越界**

尽管执行会慢一点，但是调用 strncpy 函数。仍是一种 更安全的复制字符串的方法。strncpy 类似于strcpy ，但它还有 第三个参数可以用于限制所复制的字符数。为了将str2 复制到str1 ，可以使用如下的strncpy 

```c
strncpy(str1, str2, sizeof(str1));
-----------------------------------------
strncpy(str1, str2, sizeof(str1) - 1);
str1[sizeof(str1)-1] = '\0';
```



### strcat 函数

```c
char *strcat(char *str1, const char *str2);
```

strcat 函数把字符串str2 的内容追加到字符串str1 的末尾，并且返回字符串str1 （指向结果字符串的指针）。

```c
char str1[100]="Hello";
strcat(str1, "world!");//输出Helloworld!
```

如果str1 指向的数组没有大到足以容纳str2 指向的字符 串中的字符，那么调用strcat(str1, str2) 的结果将是不可预测的。考虑下面的例子：

```c
char str1[6] = "abc";
strcat(str1, "def"); /*** WRONG ***/
```

strncat 函数比strcat 更安全，但速度也慢一 些。与 strncpy 一样，它有第三个参数来限制所复制的字符数。下面是调用的形式：

```c
strncat(str1, str2, sizeof(str1) - strlen(str1) - 1) ; //减1是预留空间给空字符
```



### strcmp 函数

```c
int strcmp(const char *str1, const char *str2);
```

strcmp 函数比较字符串str1 和字符串str2 ，然后根据str1 是小于、等于或大于str2 ， 函数返回一个负数、0、正数的值。
例如，为 了检查str1 是否小于str2 ，可以这样写：

```c
if (strcmp(str1, str2) < 0) /* is str1 < str2? */
...
```

类似于字典中单词的编排方式，**strcmp 函数利用字典顺序进行字符串比较**。更精确地说，只要满足下列两个条件之一，那么strcmp 函 数就认为str1 是小于str2 的：

- str1 与str2 的前 i 个字符一致，但是str1 的第 i + 1 个字符小于str2 的第 i + 1个字符

  例如，"abc" 小于"bcd" ，"abd" 小 于"abe" 

- s1 的所有字符与s2 的字符一致，但是s1 比s2 短

  例如，"abc" 小于"abcd" 



## 字符串惯用法

### 自编写strlen函数

第一种写法：

```c
size_t my_strlen(const char* str) {
	size_t n = 0;
	while (*str != '\0') {
		n++; 
		str++;
	}
	return n;
}
```

第二种写法：记录下首地址，利用字符串地址的差得出字符串长度为整数

```c
size_t my_strlen(const char* str) {
	const char* p = str;
	while (*str) {
		str++;
	}
	return str-p;
}
```

搜索字符串的末尾

```
while(*str){
	str++;
}
while(*str++)
	;
```



### 自编写strcat函数

第二种方式复制字符串，包括空字符
![[C基础知识-19.png]]


## 字符串数组

如何表示字符串数组？

1. 采用二维数组的方式，如果要排序则需复制字符串的内容

   ```
   char a[][6] = {"one","two","three","four","five","six"}
   ```

   缺陷是浪费内存空间，不灵活，如图会存在大量 '\0' 
   ![[C基础知识-20.png]]
1. 字符指针数组，灵活，排序只需要交换指针即可

   ```
   char* a[] = {"one","two","three","four","five","six"}
   ```
![[C基础知识-21.png]]

## 命令行参数

程序的开始：操作系统调用 main， 操作系统 ---> 通过命令行参数 ---> 应用程序

程序的结束：从 main 函数中返回 exit， 应用程序 --->退出状态码 ---> 操作系统

```c
#include<stdio.h>
int main(int argv,char* argvs[]) { //argvs为字符指针数组

	for (int i = 0; i < argv; i++) {
		printf("%s\n", argvs[i]);
	}

	return 0;
}
```

第一种方式：调用exe文件 空格 输入一行字符串，回车后，会以空格为分界符打印内容
![[C基础知识-22.png]]
第二种方式，直接在VS上添加更改参数

![[C基础知识-23.png]]




# 结构、联合和枚举


## 结构体

结构体的性质：1. 结构体的成员可以是不同类型的数据；2. 我们是通过成员的名称来选择成员，而不是位置。

Tips: 把结构体作为参数或者作为返回值，都会涉及结构体变量的复制。这样会增加程序的开销，特别是当结构体很大的时候。**为了避免这些开销，我们往往会传递一个结构体指针而不是一个结构体；类似地，我们也会返回一个结构体指针而不是一个结构体。**



### 结构体变量的声明

```c
struct student { //struct提示是个结构体，student为结构体标签
	int age;
	char name[25];
	bool sex;
	int China;
	int Math;
	int English;
};
//创建对象
struct student s1;
struct student s2;
```

### 结构体变量内存分布

结构体变量的成员在内存中是按声明顺序依次存储的，如图所示
![[C基础知识-24.png]]

![[C基础知识-25.png]]

这个结构体中最长的基本单位是：int 型 4个字节，以它为校准
第一个int型占用 4 个字节，内存地址分布为：fbd4~fbd7 ，4被4整除
第二个 char类型数组，占用 25 个字节，内存地址分布为：fbd8~fbf0 ，**(4+25)不被4整除，需要填补3个空字节，为32才能够被4整除。 fbf1~fbf3**
第三个 bool 型占用 1个字节，刚好填入前面空出的第一个地址中 fbf1，因此空的填补占 fbf2~fbf3
第四个 int 型占用4个字节，刚好顺着空的填补后面，内存地址分布为 fbf4~fbf7
...

**填充的目的是对齐！**

- **结构体变量的首地址是其最长基本类型成员的整数倍；**
- **结构体每个成员相对于结构体首地址的偏移量（offset）都是成员大小的整数倍，如有需要编译器会在成员之间加上填充字节（internal adding）；**
- **结构体的总大小为结构体最宽基本类型成员大小的 整数 倍，如有需要，编译器会在最末一个成员之后加上填充字节（trailing padding）。**
- **结构体内类型相同的连续元素将在连续的空间内，和数组一样。**
- **如果结构体内存在长度大于处理器位数的元素，那么就以处理器的倍数为对齐单位；否则，如果结构体内的元素的长度都小于处理器的倍数的时候，便以结构体里面最长的数据元素为对齐单位。**

### 结构体变量初始化

```c
struct student s1 = { 23,"zhangsan",1,80,80,80 };
struct student s2 = { 20,"lisi",0,85,99,89 };
```

> 注意：1. 初始化式 (initializer) 中的数据的顺序必须和结构体成员定义的顺序一致；
>
> 2. 和数组一样，初始化式中成员的个数可以少于结构体中的成员个数，“剩余的” 成员会被初始化为 0



### 结构体基本操作

**访问结构成员**： 通过成员名称获取成员  `s1.age;     s1.name;`

结构体的成员代表着内存中的一块存储空间，它是左值 (lvalue)。它可以出现在赋值语句的左边，并且可以当作自增、自减运算符的操作数

```c
part1.number = 9527;
part1.onhand++; /* 注意运算符的优先级 */
```



**结构体赋值：** 

```c
part2 = part1; //把 part1 内存空间的数据 copy 到 part2 内存空间中。
```

> 注意：当结构体作为参数 或 返回值，会复制整个结构体的数据。为了避免复制数据，我们往往会传递指向结构体的指针

为了避免复制数据，我们往往会传递指向结构体的指针

```C
void printf_info(struct student* s) {
	printf("%d %s %s %d %d %d\n",
		(*s).age,
		(*s).name,
		(*s).sex,
		(*s).China,
		(*s).Math,
		(*s).English);
}
```

为了方便使用指向结构体的指针，C提供 `->` 运算符

```c
void printf_info(struct student* s) {
	printf("%d %s %s %d %d %d\n",
		s->age,
		s->name,
		s->sex,
		s->China,
		s->Math,
		s->English);
}
```



**结构体别名**：  使用 typedef 给结构体起别名

```c
typedef struct student { 
	int age;
	char name[25];
	bool sex;
	int China;
	int Math;
	int English;
} Stduent;
Student s1;
```



## 枚举

在很多程序中，有些变量只能取一些离散的值。比如，扑克牌的花色只有 {黑桃、红桃、梅花、方块}。显然，我们可以用 int 类型来表示这些值，为了增加代码的可读性，我们还可以添加一些宏定义：

```c
#define SUIT int
#define DIAMONDS 0
#define HEARTS 1
#define SPADES 2
#define CLUBS 3
SUIT s = HEARTS;
```

但是这种解决方案还是有一些问题：

1. 没有显示地表明 DIAMONDS、HEARTS、SPADES、CLUBS 隶属于同一种类型；
2. 如果离散值比较多，为每个值添加一个宏定义将会很繁琐
3. 预处理器会删除我们定义的名字，如： DIAMONDS、HEARTS、SPADES、CLUBS ，所以在调试期间我们是没办法使用这些名字的。

因此，C 语言提供了一种特殊的类型——枚举类型，用来解决这些问题

**枚举定义**：与结构体相似

```c
enum SUIT {
	SPADE,   //它的值为0
	HEART,	 //它的值为1
	CLUB,	 //它的值为2
	DIAMOND	 //它的值为3
};
enum SUIT suit1 = CLUB;  //枚举变量suit1 值为club为2
enum SUIT suit2 = HEART; //枚举变量suit2 值为club为1
```

同样也可以给枚举类型起别名

枚举类型没有自己独立的命名空间；也就是说，枚举常量的名字要与同一作用域内的其他标识符不相同。

在系统内部，C 语言会把枚举变量和枚举常量当作一个整数。默认情况下，编译器会把 0, 1, 2, ... 依次赋值给枚举常量。在我们上面的例子中，DIAMONDS, HEARTS, SPADES, CLUBS 分别表示 0, 1, 2, 3。

当然，我们自由地为枚举常量选择整数值：

```c
enum suit {DIAMONDS = 40, HEARTS = 30, SPADES = 20, CLUBS = 9527};
```



当没有为枚举常量指定值的时候，它的值比前一个枚举枚举常量的值大 1 (默认情况下，第一个枚举常量的值为 0)。

```c
enum EGA_colors {BLACK, LT_GRAY = 7, DK_GRAY, WHITE = 15}; //BLACK 的值为 0，DK_GRAY 的值为 8
```



联合和结构体非常类似，它也可以有一个或多个成员，并且这些成员可以是不同类型。不一样的是，编译器只会给联合中最大的成员分配足够的内存空间。联合中的成员在内存空间上是相互重叠的。这样的结果就是给一个成员赋值，会影响到联合中其他成员的值。



# 指针的高级应用



## 动态内存分配

动态内存分配在 C 语言编程中有着举足轻重的作用，因为它是链式结构的基础。我们可以把动态分配的内存链接在一起，形成链表、树、图等灵活的数据结构。虽然动态内存分配适用于所有的数据类型，但它主要还是应用于字符串，数组和结构体中。其中，动态分配的结构体是最值得关注的，因为它们可以被链接成链表、树或其他数据结构。

### 内存分配函数

我们可以使用下面三个函数进行动态内存分配，这些函数都声明在 <stdlib.h> 头文件中。

- **`void* malloc(size_t size)`**  分配 size 个字节的内存块，不对内存块进行清零；如果无法分配指定大小的内存块，返回空指针
- **`void* calloc(size_t nmemb, size_t size)`**   为有 nmemb 个元素的数组分配内存块，其中每个元素占 size 个字节，并且对内存块进行清零；如果无法分配指定大小的内存块，返回空指针
- **`void* realloc(void *ptr, size_t new_size)`**    调整先前分配内存块的大小。如果重新分配内存大小成功，返回指向新对象的指针，否则返回空指针

这三个函数当中， **malloc 是常用的，也是效率最高的**。当我们调用这些函数申请堆内存空间的时候，函数是无法知道我们打算往内存中存储什么类型数据的，所以**函数返回 void * 类型的指针。 void * 类型的值是"通用"指针，指针本质上就是内存地址**。

**void*** ：通用指针，可以指向任意类型的对象，并且也可以转换其他类型的指针

**返回值**：如果申请成功，会返回void* 类型的指针，如果申请失败会返回空指针

**空指针**：不指向任何对象的指针。所以不能对空指针进行解引用运算，否则会报NullPointer Exception（在 C 语言中，用宏NULL表示空指针，其值为0）

------

**malloc 函数**：

```c
int* arr = malloc(n * sizeof(int));//给数组arr分配 n*4 个字节的内存空间
```

**calloc 函数**：

```c
int* arr = calloc(n , sizeof(int));//为有 n个元素的数组分配内存块，其中每个元素占 4个字节，并且对内存块进行清零（数组元素值为0）
```

**realloc 函数**

```c
void* realloc(void *ptr, size_t new_size); //ptr必须是以前通过 malloc ，calloc，realloc 返回的指针。new_size为需要的新的空间大小
```

如果申请内存空间失败，需要进行一个错误处理

```c
if (arr == NULL) { //注意顺序不要弄反，推荐这种写法
		printf("malloc failed\n");
		exit(1); //1 表示细节
}
```

### 动态内存分布

**我们定义的指针变量本身是在栈里，通过动态分配内存，分配的空间是在堆空间，指针变量是指向的 堆空间**

**realloc** 函数的参数new_size 如果为负数，则是截断原来字符串的一部分。new_size为正数，原数组会原地扩容，扩容的部分是未初始化的。
但是realloc 在原地扩容的时候也会采取另一种方式，就是新找一个内存空间，分配(原数组+新添加的)内存大小，把原数组的值赋过去，再通过 free 函数，释放原数组，返回新的数组的地址，而里面的ptr则是无效了。新添加的部分还是未初始化的数据。

C 标准有几条关于 realloc 函数的规则：

1. 如果申请新内存块不成功，那么 realloc 函数会返回空指针；并且旧内存块的数据不会发生改变
2. 如果新内存块比旧内存块大，那么超过的那部分内存是不会被初始化的。
3. 如果 realloc 的第一个参数为空指针，那么它的行为和 malloc 一样
4. 如果 realloc 的第二个参数为 0，那么它会释放 ptr 指向的内存块

**在 C 语言中，字符串是以 '\0' 结尾的，因此参数为 n+1 而非 n。通用指针类型 void * 可以自动转换为任意指针类型，因此不需要强制类型转换**

当新内存块比旧内存块小的时候，我们应该直接缩小旧内存块的大小，这样可以避免 copy 数据。同理，当新内存块比旧内存块大的时候，我们应该试图原地扩大旧内存块；如果这样不可行，再在别处申请内存块，并把旧内存块里的数据 copy 到新内存块。

> 因为 realloc 函数可能将内存块移动到其他地方，所以当 realloc 函数的返回值不为 NULL 的时候，一定要更新所有指向旧内存块的指针。





### 释放内存空间

对程序而言，不可再被访问的内存块被称为**垃圾** (garbage)。如果程序中留有垃圾，这种现象被称为**内存泄漏** (memory leak)。一些语言提供垃圾收集器 (garbage collector) 用于自动定位和回收垃圾。但是C 语言要求程序员自己负责垃圾的回收，它提供了 **free 函数**.

malloc , calloc 和 realloc 函数都是在堆上申请内存空间的，如果频繁地调用这些函数，那么堆上的空间总会被消耗殆尽。更为糟糕的是，如果丢失了对这些内存块的跟踪，那么这些内存块就无法再被程序使用，也就出现了所谓的内存泄漏现象。

如何避免内存泄露？ 及时释放无用的内存空间    **`void free( void* ptr );`**

**当传入一个ptr的指针时，内存空间的首地址的前一个附带有额外的信息，指定要复制的长度**

**但同时也会发生新的指问题，悬空指针**

```c
char *p = malloc(4);
...
free(p);
...
strcpy(p, "abc"); /* DANGEROURS! */
```

> 试图访问或者修改已经被释放掉的内存会导致未定义的行为，它可能会产生灾难性的后果

动态内存分配是链表的基础



## 指向指针的指针

第一种方式：值传递

```c
#include<stdio.h>
#include<stdlib.h>
typedef struct node {
	int data;
	struct node* next;
} Node;

Node* add_to_list(Node* list, int data);

int main(void) {
	Node* list = NULL;//空链表
	list = add_to_list(list, 3); //3
	list = add_to_list(list, 2);//2 ---> 3
	list = add_to_list(list, 1);//1-->2-->3

	return 0;
}
Node* add_to_list(Node* list, int data) {
	Node* newnode = malloc(sizeof(Node));//在堆上创建一个结点
	if (newnode == NULL) {
		exit(1); //结束这个进程
	}
	//初始化结点
	newnode->data = data;
	//头插法
	newnode->next = list;//list指向第一个结点，头插法，现在要求新结点插入到第一个结点之前

	return newnode;
}

```

如果没有返回值，直接调用函数，会形成如图所示的结果，main函数中的list仍然指向为空
![[C基础知识-26.png]]

```c

#include<stdio.h>
#include<stdlib.h>
typedef struct node {
	int data;
	struct node* next;
} Node;

void add_to_list(Node** plist, int data);

int main(void) {
	Node* list = NULL; 
	add_to_list(&list, 3); //3
	add_to_list(&list, 2);//2 ---> 3
	add_to_list(&list, 1);//1-->2-->3

	return 0;
}
void add_to_list(Node** plist, int data) {
	Node* newnode = malloc(sizeof(Node));//在堆上创建一个结点
	if (newnode == NULL) {
		exit(1); //结束这个进程
	}
	//初始化结点
	newnode->data = data;
	//头插法
	newnode->next = *plist;//list指向第一个结点，头插法，现在要求新结点插入到第一个结点之前

}

```

![image-20230207002243917](C:/Users/kicei/AppData/Roaming/Typora/typora-user-images/image-20230207002243917.png)

使用二级指针的原则：如果想要修改指针变量的值---->传递二级指针

如果想要修改的是指针变量指向的对象--->传递一级指针，，。

## 指向函数的指针



# 文件

## 流

在 C 语言中，**流** (stream) 表示任意输入的源或任意输出的目的地。流是一个抽象的概念，它即可以表示存储硬盘上的文件，也可以表示网络端口或者打印设备。流这个概念可以很好地屏蔽硬件设备之间的差异，使得 C 语言可以像读写文件一样读写任意的设备。

流模型：顺序读写文件时，不需要程序员手动移动文件的位置
![[C基础知识-27.png]]


## 文件类型

文本文件：以 **字符** 为基本单位

二进制文件：以 **字节** 为基本单位

文本文件具有两个独特的性质：

- **文本文件有行的概念**。文本文件被划分为若干行，并且每一行的结尾都以特殊字符进行标记。在Windows 系统中，是以回车符和换行符 (\r\n) 进行标记的；在 Unix 和 Macintosh 系统中是以换行符 (\n) 标记的。
- **文本文件可能包含一个特殊的 "文件末尾"标记**。一些操作系统允许在文本文件的末尾使用一个特殊的字节作为标记。在 Windows 系统中，这个标记为 '\x1a' (Ctrl+Z)。Ctrl+Z不是必需的，但如果存在，它就标志着文件的结束，其后的所有字节都会被忽略。大多数其他操作系统 (包括 UNIX) 是没有文件末尾字符。

在写入数据时，我们需要考虑是以文本形式存储还是以二进制的形式存储。
比如，存储整数 32767，一种选择是写入字符 '3', '2', '7', '6', '7'，需要 5 个字节。
![[C基础知识-28.png]]

另一个选择是以二进制形式存储这个数，这种方法只需要两个字节。![[C基础知识-29.png]]

文本形式可以方便人类阅读和编辑；二进制形式可以节省空间，并且转换效率高。



## 标准流

C 语言对流的访问是通过**文件指针**实现的，它的类型为 FILE*。并且在 <stdio.h> 头文件中提供了 3 个标准流。这 3 个标准流可以直接使用——我们不需要对其进行声明，也不用打开或者关闭它们。![[C基础知识-30.png]]

不需要手动创建，也不需要手动销毁

## 打开关闭文件

### 打开文件

读写文件之前，我们需要使用 fopen 函数打开文件

```c
FILE *fopen( const char *filename, const char *mode );
```

filename 定位文件路径位置、mode 确定访问模式

文件路径：

- 绝对路径：从根目录开始到文件的位置
- 相对路径：从当前工作目录开始到文件所在的位置

mode 打开模式：

| 文件访问<br />模式字符串 | 含义 |      解释       | 文件存在动作 | 文件不存在动作  |
| :----------------------: | :--: | :-------------: | :----------: | :-------------: |
|           "r"            | 只读 | 打开文件以读取  |    从头读    | 打开失败返回EOF |
|           "w"            | 只写 | 创建文件以写入  |   销毁内容   |   创建新文件    |
|           "a"            | 追加 |   追加到文件    |   结尾追加   |   创建新文件    |
|           "r+"           | 读写 | 打开文件以读/写 |    从头读    |      错误       |
|           "w+"           | 写读 | 创建文件以读/写 |   销毁内容   |   创建新文件    |
|           "a+"           | 读写 | 打开文件以读/写 |   结尾追加   |   创建新文件    |

<img src="https://gitee.com/demoCanon/pic-bed/raw/master/img/202302101849623.png" alt="image-20230210184937510" style="zoom:77%;" />

### 关闭文件

fclose 可以关闭程序不再使用的文件

```c
int fclose(FILE* stream);
```

如果成功关闭，fclose 返回零；否则返回 EOF(-1)。 注意：当不再使用某个文件时，一定要及时关闭该文件。
读到文件末尾或失败返回 EOF 

## 文件的读写

文件的读写包括 ：文本文件的读写、二进制文件的读写

读写文本文件：

- 连续单个字符的读写：fgetc   fputc
- 连续一行行的读写：fgets   fputs
- 序列化和反序列化：fscanf   fprintf

读写二进制文件：fread   fwrite



### 文本文件的读写

**（1）连续单个字符的读写：fgetc   fputc**

```c
int fgetc( FILE* stream );//如果读取成功，返回读取的字符；如果读到文件末尾，或者读取失败，返回 EOF

int fputc( int ch, FILE* stream );//向输出流中写入一个字符，如果写入成功，返回写入的字符；如果写入失败，返回 EOF
```

```c
FILE* src = fopen("a.txt", "r");//打开文件只读，为文件的操作创造一个流(通道)
if (src == NULL) {
		printf("open failed\n");
		exit(1);
}
FILE* dst = fopen("b.txt", "w"); //打开文件读写，为文件的操作创造一个流(通道)
	
//连续单个字符的读写入
char ch;
while ((ch = fgetc(src)) != EOF) {
	fputc(ch, dst);
}//src对应a文件的流、dst对应b文件的流
// 关闭文件
fclose(src);
fclose(dest);
```



**（2）连续一行行的读写：fgets   fputs**

`char* fgets( char* str, int count, FILE* stream );`
str 指向待写入字符数组， count表示最多能写入字符的个数 (通常就是数组的长度)， stream 就是输入流。如果读数据成功，则返回str ；否则返回 NULL 



`int fputs( const char* str, FILE* stream );`
str 指向要写出的字符串 (以'\0' 结尾的字符串)， stream 代表输出流。如果写数据成功，返回一个非负整数；否则返回 EOF

```c
char* fgets( char* str, int count, FILE* stream );//从文件流获取一个字符串,str为数组，count为数组长度

int fputs( const char* str, FILE* stream );//将一个字符串写入文件流
```

```c
char buff[100];

while (fgets(buff, 100, src) != NULL) {
	fputs(buff, dst);
}
```



**（3）序列化和反序列化：fscanf   fprintf**

序列化：将内存中的一个对象，将它转换成字符表现类型、二进制，输送到文件流

反序列化：把“文件”中的数据读取到程序中，并生成对应的对象

```c
int fscanf(FILE* stream, const char* format, ...);
```

scanf 是从标准输入 (stdin) 中读取数据，而 fscanf 可以从任何一个流中读取数据。也就是说，当 fscanf 的第一个参数为 stdin 时，它的效果等价于 scanf



```c
int fprintf(FILE* stream, const char* format, ...);
```

printf 始终是向标准输出 (stdout) 写入内容的，而 fprintf 可以向任何一个流中写入内容。也就是说，当 fprintf 的第一个参数为 stdout 时，它的效果等价于 printf

### 序列化

```c
#define _CRT_SECURE_NO_WARNINGS

#include<stdio.h>
#include<stdlib.h>

typedef struct {
	int no;
	char name[26];
	int chinese;
	int math;
	int english;
} Student;

int main(void) {
	Student s1 = { 1, "liuyifei", 100, 100, 100 };
	// 序列化
    FILE* fp = fopen("session.dat", "a");
	if (fp == NULL) {
		fprintf(stderr, "open %s failed\n", "session.dat");//向错误流输出错误信息
		exit(1);
	}

	fprintf(fp, "%d %s %d %d %d\n",
		s1.no,
		s1.name,
		s1.chinese,
		s1.math,
		s1.english);
    fclose(fp);
    //一个文件只能对应一个文件流(指定操作)，如果一个文件有着两个流，不同文件流的操作对这个文件是写还是读就不确定，不能一边写一边读吧
	return 0;
}
```



### 反序列化

```c
// 反序列化
FILE* fp = fopen("session.dat", "r");//只读
if (fp == NULL) {
	fprintf(stderr, "open %s failed\n", "session.dat");
	exit(1);
}
	
Student s2;
fscanf(fp, "%d%s%d%d%d",//   把fp流中的数据读取到程序中，并生成对应的对象
	&s2.no,
	s2.name,
	&s2.chinese,
	&s2.math,
	&s2.english);

```



### 读写二进制文件

fread 和 fwrite 主要是用来处理二进制文件的，fread 每次可以读取一大块数据，fwrite 每次可以写入一大块数据。很方便！

```c
size_t fread(void* ptr, size_t size, size_t nmemb, FILE* stream);
size_t fwrite(void* ptr, size_t size, size_t nmemb, FILE* stream);
```

**fread 从 stream 中读取数据，并填充到 ptr 指向的数组中**。数组元素的大小为 size，最多可以填充nmemb 个元素。
fread 返回实际读取元素 (不是字节) 的数量，返回值应该小于等于 nmemb。

**fwrite 把 ptr 指向数组中的元素写入到 stream 中**。数组元素的大小为 size，写入 nmemb 个元素。
fwrite 返回实际写入的元素 (不是字节) 的数量。如果出现写入错误，那么返回值将会小于 nmemb。

<img src="https://gitee.com/demoCanon/pic-bed/raw/master/img/202302101955944.png" alt="image-20230210195518803" style="zoom:55%;" /><img src="https://gitee.com/demoCanon/pic-bed/raw/master/img/202302101955814.png" alt="image-20230210195558695" style="zoom:55%;" />

```c
#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <stdlib.h>

#define N 100

int main(void) {
	// 打开文件
	FILE* src = fopen("xixi.png", "rb");//只读二进制
	if (src == NULL) {
		printf("open %s failed\n", "xixi.png");
		exit(1);
	}

	FILE* dst = fopen("mn.png", "wb");//写入二进制
	if (dst == NULL) {
		fclose(src);
		printf("open %s failed\n", "mn.png");
		exit(1);
	}

	
	char buff[4096];
	int n;//实际读的元素个数
	while ((n = fread(buff, 1, 4096, src)) != 0) {

		fwrite(buff, 1, n, dst);//读多少个，写多少个

		if (feof(src)) break; //若已抵达流尾则为非零值，否则为 ​0(未到流尾)

		if (ferror(src)) break; //若文件流已出现错误则为非零值，否则为 ​0(没有出错)
	}

	// 关闭文件
	fclose(src);
	fclose(dst);

	return 0;
}
```



### 二进制序列化

```c
#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <stdlib.h>

typedef struct {
	int no;
	char name[26];
	int chinese;
	int math;
	int english;
} Student;

int main(void) {

	Student s1 = { 1, "liuyifei", 100, 100, 100 };
	// 序列化
	FILE* fp = fopen("session.dat", "wb"); //二进制写入
	if (fp == NULL) {
		fprintf(stderr, "open %s failed\n", "session.dat");
		exit(1);
	}

	fwrite(&s1, sizeof(s1), 1, fp); //获取对象地址

	// 关闭文件
	fclose(fp);
	return 0;
}
```



### 二进制反序列化

```c
#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <stdlib.h>

typedef struct {
	int no;
	char name[26];
	int chinese;
	int math;
	int english;
} Student;

int main(void) {

	// 反序列化
	FILE* fp = fopen("session.dat", "rb");
	if (fp == NULL) {
		fprintf(stderr, "open %s failed\n", "session.dat");
		exit(1);
	}

	Student s2;
	fread(&s2, sizeof(s2), 1, fp);

	// 关闭文件
	fclose(fp);
	return 0;
}
```



## 文件定位

每个流都有相关联的文件位置。在执行读写操作时，文件位置会自动推进，并按照顺序访问文件。顺序访问是很好的，但是有时候，我们可能需要跳跃地访问文件。为此，<stdio.h> 提供了几个函数来支持这种能力：

```c
int fseek(FILE* stream, long int offset, int whence);//寻找
long int ftell(FILE* stream);
void rewind(FILE* stream);//回到文件的开头
```



**fseek**：fseek 可以改变与 stream 相关联的文件位置。其中 whence 表示参照点，参照点有 3 个选择：

- SEEK_SET：文件的起始位置。
- SEEK_CUR：文件的当前位置。
- SEEK_END：文件的末尾位置。

offset 表示偏移量 (可能为负)，它是以字节进行计数的。比如：移动到文件的起始位置，可以这样写：`fseek(fp, 0L, SEEK_SET);`
移动到文件的末尾 `fseek(fp, 0L, SEEK_END);`        往回移动 10 个字节  `fseek(fp, -10L, SEEK_CUR);`

通常情况下，fseek 会返回 0；如果发生错误 (比如，请求的位置不存在)，那么 fseek 会返回非 0 值



**ftell**：ftell 以长整数形式返回当前文件位置；如果发生错误，ftell 返回 -1L。

返回流 `stream` 的文件位置指示器。

**若流以二进制模式打开，则由此函数获得的值是从文件开始的字节数**。

若流以文本模式打开，则由此函数返回的值未指定，且仅若作为 fseek() 的输入才有意义。

ftell 一般的用法是：记录当前位置，方便以后返回

```c
long int filePos = ftell(fp);
...
fseek(fp, filePos, SEEK_SET);
```



**rewind**：rewind 会将文件位置设置为起始位置，回到文件的开头

类似 `fseek(fp, 0L, SEEK_SET);`



## 错误处理

1. 通过函数的返回值
2. **errno**（系统调用发生错误）

系统调用：操作系统提供给用户程序的接口

```c
#define _CRT_SECURE_NO_WARNINGS
#include <errno.h>
#include <stdio.h>
#include <string.h>

int main(void) {
	printf("%d\n", errno);                  //errno是个全局变量，定义在errno.h里  打印值为0
	FILE* fp = fopen("xxxxxx.jpg", "r");	//出错
	printf("%d\n", errno);					//errno是个全局变量，定义在errno.h里  打印值为2
	printf("%s\n", strerror(errno));		//打印出具体的错误原因
    
	perror("main");							//main函数出错，报告错误信息的定位

	return 0;
}
```

​	 	

## 写入内存





```c
 	FILE* fp = fopen(path, "r+");//二进制只读
    if (fp == NULL) {
        printf("open failed\n");
        return NULL;
    }

    fseek(fp, 0L, SEEK_END);//文件定位器，定位到末尾，偏移量为0

    long file_size = ftell(fp);//定义长整型，接收ftell二进制返回值即整个文件的字节数

    char* buffer = (char*)calloc(file_size, sizeof(char));//根据文件大小，在内存上申请空间大小
    if (buffer == NULL) {
        return NULL;
    }

    fseek(fp, 0L, SEEK_SET);//文件定位器，定位到开头，偏移量为0，开始从头扫描
    fread(buffer, 1, file_size, fp);//二进制读取
    run(buffer);//运行
    fclose(fp);//关闭文件流
    free(buffer);//释放内存
    buffer = NULL;//指向为空
```

