# 容器

Standard Template Library 标准模板库，有 6 大组件

1、容器（数据结构）：用来存数据的。

- 序列式容器：vector 、 deque  、list 
- 关联式容器：set 、 map
- 无序关联式容器：unordered_set 、 unordered_map

2、迭代器：看是一种指针，称为泛型指针。

3、算法：可以对容器中的数据进行相应的操作

4、适配器

- 容器适配器  优先级队列
- 算法适配器
- 迭代器的适配器（源码）

5、函数对象（仿函数）：做定制化操作

6、空间配置器 ( Allocator ) ：空间的申请与释放。（研究基本使用+原理+源码）

程序 = 容器（数据结构）+ 算法

---

dynamic 动态

deque 双端队列

## 序列式容器

<img src="/1_store/1_asset/202312052129693.png" alt="image-20230412091809472" style="zoom:67%;" />

### 初始化

三种序列式容器 vector、deque、list 完全支持五种初始化的方式

```c++
void test(){
    //-----------------------------初始化-------------------------------------
    //1、创建空对象 
    vector<int> number;

    //2、创建count个value
    vector<int> number2(10,3); //3是10个元素的初始化值

    //3、迭代器范围
    int arr[10] = {1,3,5,7,9,10,8,6,4,2};
    vector<int> number3(arr,arr + 10);//左闭右开的区间[,)这里的10指的是数组长度

    //4、拷贝构造函数与移动构造函数
    vector<int> number4_1 (number2); //拷贝构造函数
    vector<int> number4_2 (std::move(number2));//移动构造函数
    
    //5、大括号
    vector<int> number5 = {1,3,5,9,7,2,4,6,8};

//只需要将vector替换为对应的deque 与 list
```



### 遍历

4 种遍历方式

三种序列式容器，其中：list 是没有下标的，所以不支持下标访问运算符。其他的三种遍历的方式，三种容器都支持。

```c++
//只需要将vector替换为对应的deque 与 list
vector<int> number2(10,3);
void test(){
    //----------------------------遍历操作------------------------------------
#if 0 
    //1、使用下标进行遍历，只有list不支持下标访问
    for(size_t idx = 0; idx != number2.size();++idx){
        cout << number2[idx] << "  ";
    }
    cout << endl;
#endif
    
    //2、迭代器：泛型指针
    vector<int>::iterator it; //没有初始化迭代器it
    for(it = number.begin();it != number.end(); ++it){
        cout << *it << "  ";
    }
    cout << endl;
    
    //3、迭代器（对迭代器进行初始化）
    vector<int>::iterator it2 = number.begin();
    for(;it2 != number.end(); ++it2){
        cout << *it2 << "  ";
    }
    cout << endl;

    //4、for与auto,C++11
    for(auto &elem : number){ //&是引用的意思，减少拷贝操作
        cout << elem << "  ";
    }
    cout << endl;

}
```

注意：list 不支持下标

<img src="/1_store/1_asset/image-20231205212940582.png" alt="image-20231205212940582" style="zoom:80%;" />



```c++
//display 打印函数
template <typename Container>
void display(const Container &con)
{
    for(auto &elem : con)
    {
        cout << elem << "  ";
    }
    cout << endl;
}

```



### 在尾部进行插入与删除

三种容器在尾部进行插入与删除的使用是完全一样的。

```c++
void test()
{
    cout << endl << "----------------初始化vector容器-----------------" << endl;
    vector<int> number = {1, 3, 5, 9, 7, 4, 6, 2, 10, 3};
    display(number); //打印容器number的值                 
    cout << "number.size() = " << number.size() << endl //打印容器存储量
         << "number.capacity() = " << number.capacity() << endl ; //打印容器空间量

    cout << endl << "----------------在vector的尾部进行插入与删除------------------" << endl;
    cout << "在vector的尾部push_back插入2个元素，打印如下：" << endl;
    number.push_back(33);
    number.push_back(77);
    display(number);
    cout << "number.size() = " << number.size() << endl
         << "number.capacity() = " << number.capacity() << endl << endl;
    
    cout << "在vector的尾部pop_back删除元素，打印如下：" << endl;
    number.pop_back(); //删除尾部元素
    display(number);
    cout << "number.size() = " << number.size() << endl
         << "number.capacity() = " << number.capacity() << endl;
    
    cout << endl;
}

void test2(){
    cout << endl << "----------------初始化deque容器-----------------" << endl;
    deque<int> number = {1, 3, 5, 9, 7, 4, 6, 2, 10, 3};
    display(number);

    cout << endl << "在deque的尾部push_back进行插入" << endl;
    number.push_back(33);
    number.push_back(77);
    display(number);
    cout << endl << "在deque的尾部pop_back进行删除" << endl;
    number.pop_back();
    display(number);
    
    cout << endl;
}
    
void test3(){
    cout << endl << "----------------初始化list容器-----------------" << endl;
    list<int> number = {1, 3, 5, 9, 7, 4, 6, 2, 10, 3};
    display(number);

    cout << endl << "在list的尾部push_back进行插入" << endl;
    number.push_back(33);
    number.push_back(77);
    display(number);
    cout << endl << "在list的尾部pop_back进行删除" << endl;
    number.pop_back();
    display(number);
    
    cout << endl;
}
```



### 在头部进行插入与删除

**deque 与 list 支持在头部进行插入与删除**，但是 vector 没有在头部进行插入与删除的操作。

```c++ 
void test2(){
    cout << endl << "----------------初始化deque容器-----------------" << endl;
    deque<int> number = {1, 3, 5, 9, 7, 4, 6, 2, 10, 3};
    display(number);
	
    cout << endl << "在deque的头部push_front进行插入" << endl;
    number.push_front(100);
    number.push_front(200);
    display(number);
    cout << endl << "在deque的尾部pop_front进行删除" << endl;
    number.pop_front();
    display(number);
    
    cout << endl;
}
  
void test3(){
    cout << endl << "----------------初始化list容器-----------------" << endl;
    list<int> number = {1, 3, 5, 9, 7, 4, 6, 2, 10, 3};
    display(number);
	
    cout << endl << "在list的头部push_front进行插入" << endl;
    number.push_front(100);
    number.push_front(200);
    display(number);
    cout << endl << "在list的尾部pop_front进行删除" << endl;
    number.pop_front();
    display(number);
    
    cout << endl;
}
  
```

Q：为什么 vector 不支持在头部进行插入与删除？

A：因为 vector 中的元素是连续的，如果在头部删除一个元素，那么后面的元素都会前移；如果在头部进行插入一个元素，那么后面的所有的元素都会后移，这样会导致一个元素的插入与删除，使得整个 vector 中的元素都要变动位置，时间复杂度 O(N)
    

```c++
//Q:如何获取vector第一个元素的地址？
&number;  error,获取不到第一个元素的地址 */
&number[0];  ok,可以获取第一个元素的地址 */
&*number.begin();  ok,可以获取第一个元素的地址 */
int *pdate = number.data();  ok,可以获取第一个元素的地址 */
```

<img src="/1_store/1_asset/202312052130271.png" alt="image-20230412205943097" style="zoom:80%;" />

### vector 的原理图

<img src="/1_store/1_asset/202312052129201.png" alt="image-20230412210105184" style="zoom:67%;" />

`_M_start`：指向第一个元素的位置

`_M_finish`：指向最后一个元素的下一个位置

`_M_end_of_storage`：指向当前分配空间的最后一个位置的下一个位置

```c++
template <class _Tp, class _Alloc = __STL_DEFAULT_ALLOCATOR(_Tp) >
class vector
{
public:
    //类型萃取（重要）
    typedef _Tp value_type;
    typedef value_type* pointer;
    typedef const value_type* const_pointer;
    typedef value_type* iterator;
    typedef const value_type* const_iterator;
    typedef value_type& reference;
    typedef const value_type& const_reference;
    typedef size_t size_type;
    typedef ptrdiff_t difference_type;
    
    //为了严格表明_Base::allocator_type是一种类型，所以加上typename关键字进行强调
    typedef typename _Base::allocator_type allocator_type;
    
    //vector的下标访问运算符的重载
    reference operator[](size_type __n)
    { 
        return *(begin() + __n); 
    }
    
    void _M_range_check(size_type __n) const 
    {
       if (__n >= this->size())
           __stl_throw_range_error("vector");
    }

    //总结：通过下标获取元素，可以使用at，也可以直接使用下标，但是at比下标访问更安全，因为有范围检查
    reference at(size_type __n)
    {
		_M_range_check(__n); 
		return (*this)[__n]; 
    }
};
```

<img src="/1_store/1_asset/202312052130578.png" alt="image-20230412211434289" style="zoom:67%;" />

### deque 的原理图

<img src="/1_store/1_asset/202312052130230.png" alt="image-20230412211554701" style="zoom: 50%;" />

<img src="/1_store/1_asset/202312052130948.png" alt="image-20230412211628202" style="zoom: 50%;" />

注意：deque 是由很多小片段组成的，片段内部是连续的，片段之间是不连续。所以可以这么说，逻辑上是连续的，但是物理上是不连续。可以通过 **中控器数组** 进行调控。

### 在任意位置进行插入

三种序列式容器在任意位置插入元素的 **四种方法**

<img src="/1_store/1_asset/202312052130443.png" alt="image-20230413092532259" style="zoom:67%;" />

```c++ 
void test3(){
    list<int> number = {1, 3, 5, 9, 7, 4, 6, 2, 10, 3};
	cout << endl << "在list的任意位置进行插入元素" << endl;
    list<int>::iterator it = number.begin();
    ++it;
    ++it; //迭代器向后移动2个位置
    
    cout << "*it = " << *it << endl;
    number.insert(it, 100); //1、找一个位置，插入一个元素
    display(number);
    cout << "*it = " << *it << endl;

    cout << endl << endl;
    number.insert(it, 30, 200);//2、找一个位置，插入count30个相同元素
    display(number);
    cout << "*it = " << *it << endl;

    cout << endl << endl;
    vector<int> vec = {77, 99, 33, 55};
    number.insert(it, vec.begin(), vec.end()); //3、找一个位置，插入迭代器范围的元素
    display(number);
    cout << "*it = " << *it << endl;

    cout << endl << endl;
    number.insert(it, {111, 444, 666, 888, 333}); //4、找一个位置，插入一个大括号范围的元素
    display(number);
    cout << "*it = " << *it << endl;

}
```

总结：对于 list 而言，插入的时候，迭代器是跟着 **结点走的**，它在使用的时候，不会存在什么问题。但是对于 vector 而言，插入元素的大小会影响 vector 底层的扩容，有可能会导致迭代器失效。所以为了解决这个问题，每次操作迭代器时候，最好可以提前更新迭代器的位置，让迭代器指向新的空间的位置，这样就保证不会出错。

特意的，对于 deque 而言，去进行 insert 操作的时候，也有可能迭代器失效，所以最好还是可以每次使用迭代器的时候，像 vector 一样，将迭代器的位置进行更新，指向新的空间的位置。

插入一个链接



### vector 删除连续重复元素

vector 删除连续重复元素的隐患：

```c++
void test()
{
    vector<int> number = {1, 3, 6, 6, 6, 6, 7, 5};
    display(number);
    //这种方式不能删除连续重复元素
    for(auto it = number.begin(); it != number.end(); ++it)
    {
        if(6 == *it)
        {
            number.erase(it);
        }
    }
    display(number);
}
```



```
1 3 6 6 6 6 7 5
1 3 6 6 7 5
```



原因是：每删除一个元素，vector 的元素便向前移动位置，但是迭代器的位置不变，迭代器所在位置的数值发生变化，接着进入下一次的循环更新迭代器的位置，导致连续重复的元素逃过了判断，从而漏掉。

正确解决办法：

```c++
void test2()
{
    vector<int> number = {1, 3, 6, 6, 6, 6, 7, 5};
    display(number);
    for(auto it = number.begin(); it != number.end(); )  //不用在更新迭代器的位置
    {
        if(6 == *it)
        {
            number.erase(it);
        }
        else //如果不是该数字，才会去更新迭代器的位置
        {
            ++it; 
        }
    }
    display(number);
}

void test3()
{
    vector<int> number = {1, 3, 6, 6, 6, 6, 7, 5};
    display(number);
    for(auto it = number.begin(); it != number.end(); ++it)
    {
        while(6 == *it) //循环检测要删除的元素，如果不是连续元素就跳出（不更新迭代器的位置，在原位置上检测）
        {
            number.erase(it); 
        }
    }
    display(number);
}

```

deque 与 list 就不会出现，因为 vector 是属于动态数组的，deque 与 list 是属于链表的。

### 元素的清空

```c++ 
cout << endl << "清空vector容器中的元素" << endl;
number.clear();//只是将元素清空了，但是没有回收内存
number.shrink_to_fit();//将剩余的空间进行了回收
cout << "number.size() = " << number.size() << endl
     << "number.capacity() = " << number.capacity() << endl;
//------------------------------------------------------------------
cout << endl << "清空deque容器中的元素" << endl;
number.clear();//只是将元素清空了，但是没有回收内存
number.shrink_to_fit();//将剩余的空间进行了回收
cout << "number.size() = " << number.size() << endl;
//-----------------------------------------------------------------------
//对于list而言，元素删除了也会将结点删除，就不存在回收空间函数
cout << endl << "清空list容器中的元素" << endl;
number.clear();
cout << "number.size() = " << number.size() << endl;
```



### emplace_back 的使用

在容器末尾就地构造元素

```c++
void test()
{
    vector<Point> number;
    Point pt(1, 2);
    number.push_back(Point(1, 2));
    //number.push_back(1, 2); error!
    number.push_back((1, 2));
    number.push_back({1, 2});
    number.push_back(pt);
    number.emplace_back(3, 4); //免去大括号与Point声明
}
```

size()获取容器中元素个数，三者都有该函数

capacity()获取容器申请的空间大小，只有 vector 有这个函数。



## list 的特殊用法

### 排序函数 sort

<img src="/1_store/1_asset/202312052131692.png" alt="image-20230413134629671" style="zoom:67%;" />

存在两种形式的 sort 函数

```c++
void test()
{
    list<int> number = {1, 3, 5, 5, 9, 9, 8, 6, 2};
    display(number);

    cout << endl << "测试list的sort函数" << endl;
    number.sort();//1、默认情况，会按照从小到大的顺序进行排序
    
    //std::less<T> 如果lhs < rhs 就返回true,否则返回false
    std::less<int> les; //从小到大进行排序
    number.sort(std::less<int>()); //传递右值
    number.sort(les); //传递左值 
    
    //std::greater<T> 如果lhs > rhs 就返回true,否则返回false
    number.sort(std::greater<int>()); //从大到小进行排序
    display(number);
}
```

Compare 是个类，比较的是对象。

可以看一下源码：

<img src="/1_store/1_asset/202312052131172.png" alt="image-20230413153256848" style="zoom: 67%;" />

<img src="/1_store/1_asset/202312052131294.png" alt="image-20230413153428133" style="zoom:67%;" />

那 `_StrictWeakOrdering` 是个什么东西呢？ 翻译为：严格弱排序。

严格是说在判断的时候会用 "<"，而不是"<="，弱排序是因为，一旦"<"成立便认为存在"<"关系，返回ture，而忽略了"="关系和">" 区别，把它们归结为 false。

https://www.boost.org/sgi/stl/StrictWeakOrdering.html

严格弱排序是一个二元谓词，它对两个对象进行比较，如果第一个对象在第二个对象之前，则返回真。这个谓词必须满足严格弱排序的标准数学定义。准确的要求在下面说明，但它们的大致意思是，**严格弱排序的行为方式必须是 "小于 " 的行为方式**：如果 a 小于 b，则 b 不小于 a，如果 a 小于 b 且 b 小于 c，则 a 小于 c，以此类推。

<img src="/1_store/1_asset/202312052131882.png" alt="image-20230413154122470" style="zoom:67%;" />

所以我们了解到无参 sort 的排序是按照严格弱排序的，而有参 sort 给出了一个模板，_StrictWeakOrdering 告诉你，sort 也应该是弱排序，但毕竟是模板，可以传入的不一定非要是严格弱排序，也可以是其他数学定义的排序方式。所以也可以传入严格强排序。

```c++
template <class _Tp> //严格弱排序
struct less : public binary_function<_Tp,_Tp,bool> 
{
  bool operator()(const _Tp& __x, const _Tp& __y) const { return __x < __y; } // < 运算符重载
};
```

```c++
template <class _Tp> //严格强排序
struct greater : public binary_function<_Tp,_Tp,bool> 
{
  bool operator()(const _Tp& __x, const _Tp& __y) const { return __x > __y; } // > 运算符重载
};

```



### 实现 list 的 sort

```c++
//自定义函数对象（仿函数）
template <typename T>
struct CompareList //模仿less和greater的定义
{
    bool operator()(const T &lhs, const T &rhs) const
    {
        cout << "bool operator()(const T &, const T &) const" << endl;
        return lhs < rhs;
    }
};

void test()
{
    list<int> number = {1, 3, 5, 5, 9, 9, 8, 6, 2};
    display(number);

    cout << endl << "测试list的sort函数" << endl;
    number.sort(CompareList<int>()); //自己实现比较原则
    display(number);
}

```

函数对象：

函数对象是 **重载函数调用操作符** 的类的对象。即函数对象是行为类似函数的对象，又称 [仿函数](https://so.csdn.net/so/search?q=仿函数&spm=1001.2101.3001.7020)，是一个能被当做普通函数来调用的对象。





### reverse 反转序列 与 unique 去重元素

```c++
void test()
{
	list<int> number = {1, 3, 5, 5, 9, 9, 8, 6, 2, 5, 9, 5};
    display(number);

    cout << endl << "测试list的reverse函数" << endl;
    //reverse实现的是反转容器序列
    number.reverse();
    display(number);

    cout << endl << "测试list的unique函数" << endl;
    number.sort();
    number.unique();//需要先将元素进行排序，然后才能去除重复元素
    display(number);
    
}
```



### merge 合并 vector

两个链表要进行 merge 的时候，需要先各自进行相同的排序，然后再进行 merge，否则不排序后的合并没有意义。

```c++
void test()
{
	cout << endl << "---------测试list的merge函数-----------" << endl;
    list<int> number2 = {11, 99, 88, 33, 7};
    number2.sort();
    cout << "number2链表: ";
    display(number2);
    cout << "number 链表: ";
    display(number);
    //两个链表要进行merge的时候，需要先分别将两个链表进行从小
    //到大排序，然后再进行merge，否则就达不到合并后排序的效果
    cout << "合并操作: number.merge(number2); " << endl;
    number.merge(number2);
    cout << "number2链表: ";
    display(number2);
    cout << "number链表: ";
    display(number);

    

    cout << endl << "从大到小合并："<< endl;
    number.sort(std::greater<int>());
    number2.sort(std::greater<int>());
    number.merge(number2);
     cout << "number2链表: ";
    display(number2);
    cout << "number链表: ";
    display(number);
}
```

<img src="/1_store/1_asset/202312052131585.png" alt="image-20230413191045657" style="zoom:80%;" />

### splice 操作

从一个 list 转移元素给另一个。不复制或移动元素，仅重指向链表结点的内部指针。

**1、移动一个容器内的所有元素**

```c++
void test(){
    cout << endl << "测试splice函数" << endl;
    list<int> number = {1,2,3,5,6,8,9 };
    list<int> number3 = {111, 555, 777, 333, 666};
    display(number3);
    auto it = number.begin();
    ++it;
    ++it;
    cout << "*it = " << *it << endl;
    number.splice(it, number3); 
    display(number); 
    display(number3);
}
```

<img src="/1_store/1_asset/202312052131628.png" alt="image-20230413200952776" style="zoom:80%;" />

结果分析：splice 也如同 merge 一样，将 number3 的元素移动到 number 后，number3 自身的元素啥也没有了。和 merge 不一样的是，splice 可以在指定的位置下进行转移元素。

**2、移动一个容器内的一个元素**

```c++
void test(){
	cout << endl << "测试splice函数" << endl;
    list<int> number = {1,2,3,5,6,8,9 };
    list<int> number3 = {111, 555, 777, 333, 666};
    cout << "number链表: ";
    display(number);
    cout << "number3链表: ";
    display(number3);
    
    auto it = number.begin();
    ++it;
    ++it;
    cout << "*it = " << *it << endl;
    auto it2 = number3.begin();
    ++it2;
    cout << "*it2 = " << *it2 << endl;
    number.splice(it, number3, it2); //将number3中it2位置下的元素移动到number中的it位置下
    
    cout << "number链表: ";
    display(number);
    cout << "number3链表: ";
    display(number3);
}
```

<img src="https://gitee.com/demoCanon/pic-bed/raw/master/img/202304132024051.png" alt="image-20230413202121065" style="zoom:80%;" />



**3、移动一个容器内的范围元素**

```c++
void test()
{
	cout << endl << "测试splice函数" << endl;
    list<int> number = {1,2,3,5,6,8,9 };
    list<int> number3 = {111, 555, 777, 333, 666};
    cout << "number链表: ";
    display(number);
    cout << "number3链表: ";
    display(number3);
    auto it = number.begin();
    ++it;
    ++it;
    cout << "*it = " << *it << endl;

    auto it2 = number3.begin();
    ++it2;
    cout << "*it2 = " << *it2 << endl;
    auto it3 = number3.end();
    --it3;
    cout << "*it3 = " << *it3 << endl;
    //将number3中it2到it3范围内的元素移动到number中的it位置下
    number.splice(it, number3, it2, it3);//[it2, it3)左闭右开
    cout << "number链表: ";
    display(number);
    cout << "number3链表: ";
    display(number3);
}
```

<img src="/1_store/1_asset/202312052131968.png" alt="image-20230413202930821" style="zoom:80%;" />

**4、移动容器自身内的元素**  （避免使用，会出现问题）

```c++
void test(){
	auto it4 = number.begin();
    ++it4;
    ++it4;
    cout << "*it4 = " << *it4 << endl;
    auto it5 = number.end();
    --it5;
    --it5;
    cout << "*it5 = " << *it5 << endl;
    
    number.splice(it4, number, it5); //迭代器it4与it5都是number自身的元素
    display(number);
}
```



### vector 的 insert 插入扩容（了解）

```c++
void test()
{
    vector<int> number = {1, 3, 5, 9, 7, 4, 6, 2, 10, 3};
    display(number);
    cout << "number.size() = " << number.size() << endl
         << "number.capacity() = " << number.capacity() << endl;

    cout << endl << "在vector的尾部进行插入与删除" << endl;
    number.push_back(33);
    display(number);
    cout << "number.size() = " << number.size() << endl
         << "number.capacity() = " << number.capacity() << endl;

    cout << endl << "在vector的任意位置进行插入元素" << endl;
    vector<int>::iterator it = number.begin();
    ++it;
    ++it;
    cout << "*it = " << *it << endl;
    number.insert(it, 100);
    display(number);
    cout << "number.size() = " << number.size() << endl
         << "number.capacity() = " << number.capacity() << endl;
    cout << "*it = " << *it << endl;
}
```

<img src="/1_store/1_asset/202312052131822.png" alt="image-20230413205710784" style="zoom:80%;" />

结果分析：因为 push_back 每次只能插入一个元素，所以按照 2 * capacity 进行扩容是没有任何问题的，但是 insert 进行插入元素的时候，插入元素

的个数是不确定的，所以不能按照 2 * capacity 进行扩容

设置 `size() = m = 12`, `capacity() = n = 20`, 待插入的元素个数 `t` ：

1. 当插入元素的个数 `t < n - m`, 此时就不会扩容 
2. 当插入元素的个数 `n - m < t < m`, 此时就按照 `2 * m` 进行扩容
3. 当插入元素的个数 `n - m < t, m < t < n`, 此时就按照 `t + m` 进行扩容
4. 当插入元素的个数 `n - m < t, t > n`, 此时就按照 `t + m` 进行扩容

你可以试一下。



## 关联式容器

关联容器实现能快速查找（*O(log n)* 复杂度）的数据结构。

| [set](https://zh.cppreference.com/w/cpp/container/set)       | 唯一键的集合，按照键排序 (类模板)             |
| ------------------------------------------------------------ | --------------------------------------------- |
| [map](https://zh.cppreference.com/w/cpp/container/map)       | 键值对的集合，按照键排序，键是唯一的 (类模板) |
| [multiset](https://zh.cppreference.com/w/cpp/container/multiset) | 键的集合，按照键排序 (类模板)                 |
| [multimap](https://zh.cppreference.com/w/cpp/container/multimap) | 键值对的集合，按照键排序 (类模板)             |

### set 使用

set 的特征：

- key 值是唯一的，不能重复
- 默认情况下，会按照 key 值升序排列
- set 的底层使用的是红黑树数据结构



#### const 与 find 查找

```c++
void test(){
	cout << "------------set的初始化------------" << endl;
    set<int> number = {1, 3, 5, 5, 7, 5, 9, 2, 4};
    /* set<int, std::greater<int>> number = {1, 3, 5, 5, 7, 5, 9, 2, 4}; */
    display(number);
    cout << endl ;

    cout << "----------set中的count查找----------" << endl;
    size_t cnt1 = number.count(7);
    cout << "查找为7的元素是否在容器内? cnt1为1表示存在, 为0表示不存在" << endl;
    cout << "cnt1 = " << cnt1 << endl << endl;

    cout << "----------set中的find查找----------" << endl;
    /* set<int>::const_iterator it = number.find(10);  //OK的*/
    set<int>::iterator it = number.find(10);
    if(it == number.end()) //迭代器判空的独特方法，不能用nullptr
    {
        cout << "该为10的元素不存在set中, 查找失败" << endl;
    }
    else{   
        cout << "该为10的元素存在set中" << *it << endl;
    }
    cout << endl;
}
```

<img src="/1_store/1_asset/202312052132250.png" alt="image-20230413214649264" style="zoom:80%;" />

#### insert 插入

注意：set 容器内如果插入已经存在的元素，则不会重复插入，会被过滤掉。

```c++
void test(){
    set<int> number = {1, 3, 5, 5, 7, 5, 9, 2, 4};
    
	cout << "----------set中的insert插入----------" << endl;
    cout << "原number的元素有: ";
    display(number);
    
    //set的insert单独插入一个数返回的是pair迭代器类型的
    pair<set<int>::iterator, bool> ret = number.insert(8);
    if(ret.second)
    {
        cout << "该元素插入成功 " << *ret.first << endl;
    }
    else
    {
        cout << "该元素插入失败,该元素已经存在set中" << endl;
    }
    display(number);
    cout << endl ;
    
    //set的insert插入另一个容器范围的元素
    vector<int> vec = {1, 3, 7, 9, 11, 45, 7, 9, 5, 7};
    number.insert(vec.begin(), vec.end());
    display(number);
    cout << endl ;

    //set的insert插入大括号内的元素
    number.insert({13, 56, 78, 23, 11, 14});
    display(number);
}
```

<img src="/1_store/1_asset/202312052132232.png" alt="image-20230413215201616" style="zoom:80%;" />



#### 下标和修改操作

```c++
void test(){
    set<int> number = {1, 3, 5, 5, 7, 5, 9, 2, 4};
	cout << endl << "set的下标操作" << endl;
    //set是不支持下标的
    /* cout << "number[1] = " << number[1] << endl; */

    cout << endl << "set的修改操作" << endl;
    it = number.end();
    --it;
    --it;
    cout << "*it = " << *it << endl;
    /* *it = 123;//error, 不能修改 */
}
```



#### 实现 set 的类对象类型元素的排序

先了解一下

```c++
template<
    class Key,
    class Compare = std::less<Key>,
    class Allocator = std::allocator<Key>
> class set;

//看一下构造函数有关于大括号的构造
set( std::initializer_list<value_type> init,
     const Compare& comp = Compare(),
     const Allocator& alloc = Allocator() );

//less的比较运算符重载
constexpr bool operator()(const T &lhs, const T &rhs) const 
{
    return lhs < rhs; // 
}
```



``` c++
#include <math.h>
#include <iostream>
#include <set>
#include <utility>
#include <vector>

using std::cout;
using std::endl;
using std::set;
using std::pair;
using std::vector;

template <typename Container>
void display(const Container &con)
{
    for(auto &elem : con)
    {
        cout << elem << "  ";
    }
    cout << endl;
}


class Point
{
public:
    Point(int ix = 0, int iy = 0)
    : _ix(ix)
    , _iy(iy)
    {
        /* cout << "Point(int = 0, int = 0)" << endl; */
    }

    double getDistance() const
    {
        return hypot(_ix, _iy);
    }

    void print() const
    {
        cout << "(" << _ix 
             << ", " << _iy
             << ")";
    }

    ~Point()
    {
        /* cout << "~Point()" << endl; */
    }

    friend std::ostream &operator<<(std::ostream &os, const Point &rhs);
    friend bool operator<(const Point &lhs, const Point &rhs);

private:
    int _ix;
    int _iy;
};

std::ostream &operator<<(std::ostream &os, const Point &rhs)
{
    os << "(" << rhs._ix 
       << ", " << rhs._iy
       << ")";

    return os;
}

bool operator<(const Point &lhs, const Point &rhs) //会在less的函数对象中触发 < 
{
    cout << "bool operator<(const Point &, const Point &)" << endl;
    //按照距离的远近进行比较
    if(lhs.getDistance() < rhs.getDistance())
    {
        return true;
    }
    else if(lhs.getDistance() == rhs.getDistance())
    {
        if(lhs._ix < rhs._ix)
        {
            return true;
        }
        else if(lhs._ix == rhs._ix)
        {
            if(lhs._iy < rhs._iy)
            {
                return true;
            }
            else
            {
                return false;
            }
        }
        else
        {
            return false;
        }
    }
    else
    {
        return false;
    }
}
void test2()
{
    set<Point> number = {
        Point(-1, 2),
        Point(1, -2),
        Point(1, 2),
        Point(2, -2),
        Point(3, 5),
        Point(1, 2)
    };

    display(number);
}

int main(int argc, char **argv)
{
    test2();
    return 0;
}


```

<img src="/1_store/1_asset/202312052132235.png" alt="image-20230413224604737" style="zoom:67%;" />

**运行结果：之所以 set 中插入 Point 对象数据后，能够排序，是因为在 less 中比较两个对象时，触发了 < 比较运算符的我们自己写的重载。** 第二种方式：当然你也可以用函数对象去实现。不过要设计成友元的方式，不然无法访问内部私有成员。

<img src="/1_store/1_asset/202312052132715.png" alt="image-20230414084950887" style="zoom:67%;" />

第 3 种方式，对 `std::less` 的模板进行特化处理。

```c++
namespace std
{
//模板的特化
template<>
struct less <Point>
{
	bool operator()(const Point &lhs,const Point &rhs){
		..... //不过要用get方法去获取私有成员
	}
};
} //end of namespace std
```



针对于自定义类型的三种形式（重要）

##### 1、模板的特化

<img src="/1_store/1_asset/202312052132714.png" alt="image-20230414090634321" style="zoom:50%;" />

##### 2、函数对象的形式

<img src="/1_store/1_asset/202312052132316.png" alt="image-20230414090644223" style="zoom:50%;" />

##### 3、运算符重载形式

<img src="/1_store/1_asset/202312052132350.png" alt="image-20230414090655235" style="zoom:50%;" />



### multiset 的使用

multiset 的特征：

- **key 值是不唯一的，可以重复**
- 默认情况下，会按照 key 值升序排列
- multiset 的底层使用的是红黑树数据结构

除去 insert 的返回值是 pair 类型的双迭代器，其他的操作均与 set 效果相同。看一下 multiset 独有的 bound 函数

#### bound 函数

<img src="/1_store/1_asset/202312052132203.png" alt="image-20230414092838960" style="zoom:50%;" />

<img src="/1_store/1_asset/202312052132452.png" alt="image-20230414092851539" style="zoom:50%;" />

注意：set 与 multiset 针对于自定义类型的写法完全等同，都可以针对于自定义类型的三种形式完全一样。



### map 使用

map 的特征

- map 中存放的是 pair，也就是 key-value 类型，key 值是惟一的
- 不能重复，但是 value 是可以重复的, 也可以不重复
- 默认情况下，也会按照 key 值进行升序排列
- 底层实现也是使用的红黑树

#### 初始化

```c++
template <typename Container>
void display(const Container &con)
{
    for(auto &elem : con)
    {
        cout << elem.first << "  " << elem.second << endl;
    }
}
void test()
{
    /* map<int, string, std::greater<int>> number = { */
	map<int, string> number = {
    	pair<int, string>(8, "wuhan"),
    	pair<int, string>(6, "wuhan"),
    	pair<int, string>(9, "hubei"),
    	{1, "beijing"},
    	{1, "dongjing"},
    	{2, "nanjing"},
    	make_pair(3, "tianjin"),
    	make_pair(8, "wuhan"),
    	make_pair(10, "shenzhen")
    };
    display(number);
}
```



#### map 的查找

```c++
void test()
{
	cout << endl << "map的查找操作" << endl;
    //返回拥有关键 key 的元素数,但key是唯一的，只会返回 0或1
    size_t cnt = number.count(8);
    cout << "cnt = " << cnt << endl;

    cout << endl;
    //查找key为10的键值对
    map<int, string>::iterator it = number.find(10);
    //指向键等于 key 的元素的迭代器。若找不到这种元素，则返回尾后（见 end() ）迭代器。
    if(it == number.end())
    {
        cout << "该元素不存在map中" << endl;
    }
    else
    {
        cout << "该元素存在map中 " 
             << it->first << "  "
             << it->second << endl;
    }
}
```



#### map 的插入

`std::pair<iterator, bool> insert( const value_type& value );` 

map 的插入是一个 pair 类型，不是一个 key，所以需要构建 pair 类型

```c++


void test()
{
	cout << endl << "map的insert操作" << endl;
    pair<map<int, string>::iterator, bool>
        // 3种插入方式：1、pair类型数据 2、打括号插入 3、make_pair插入
        /* ret = number.insert(pair<int, string>(4, "chongqing")); */
        /* ret = number.insert({4, "chongqing"}); */
        ret = number.insert(make_pair(4, "chongqing"));
    
    if(ret.second) //pair第二个元素类型为bool，若成功插入则为true
    {
        cout << "插入成功 " 
             << ret.first->first << "   "
             << ret.first->second << endl;
    }
    else
    {
        cout << "插入失败, 证明该元素已经存在map中" << endl;
    }
    display(number);

    cout << endl << endl;
    map<int, string> number2 = {
        {7, "sichuan"},
        {5, "sichuan"},
    };
    //insert的范围插入
    number.insert(number2.begin(), number2.end());
    display(number);

    cout << endl << endl;
    //insert的大括号多个插入大括号嵌套大括号(一个个都是pair类型)
    number.insert({ {11, "hello"}, {2, "nanjing"}, {13, "hangzhou"} });
    display(number);

}
```



#### 下标和修改操作

支持下标操作和修改操作。

```c++
void test()
{
    cout << endl << "map的下标操作" << endl;
    
    //map的下标中传的是key类型，然后返回的是value类型
    cout << "number[1] = " << number[1] << endl;//查找
    
    //如果下标操作访问的key不存在，就会变为插入操作，value为空
    cout << "number[12] = " << number[12] << endl;//插入
    display(number);

    cout << endl << endl;
    //T &operator[](key)
    number.operator[](12) = "kiki";
    /* number[12] = "kiki";//修改 等价于上面*/ 
    
    number[5] = "kiki";
    display(number);
    
    /* const map<int, string> number3 = {make_pair(1, "hehhe")}; */
    /* number3[1]; error 因为是const是不允许通过下标访问修改value*/

}
```



#### 针对于自定义类型

如果自定义类型是 string，是不需要重载小于符号的，因为其可以进行直接比较。

```C++
void test2()
{
    /* map<Point, string> number = {//error */
    //之所以报错是因为，map会自动对key进行排序，但Point没有实现对比较运算符的重载，所以会报错
    map<string, Point> number = {
        {"hello", Point(1, 2)},
        {"world", Point(2, 2)},
        make_pair("wuhan", Point(3, 2)),
        make_pair("wuhan", Point(3, 2)),
        {"nanjing", Point(1, 2)},
        {"dongjing", Point(1, 2)},
    };
    display(number);
}
```

<img src="/1_store/1_asset/202312052132790.png" alt="image-20230414192505673" style="zoom:50%;" />

### multimap 的使用

#### multimap 特征

<img src="/1_store/1_asset/202312052132965.png" alt="image-20230414193944903" style="zoom:67%;" />

#### 不支持下标（重点）

<img src="/1_store/1_asset/202312052132020.png" alt="image-20230414194031626" style="zoom:67%;" />



## 无序关联式容器

无序关联容器提供能快速查找（均摊 *O(1)* ，最坏情况 *O(n)* 的复杂度）的无序（哈希）数据结构。

### 基本概念

底层使用的是哈希。

**哈希函数：** 通过 key 值找到 key 值存放的位置。size_t index = H(key)

**哈希函数的构建：** 目的就是为了设计哈希函数，尽量能减少冲突。

**哈希冲突：** `H(key1) = H(key2)`，key 值不一样，但是 key 值对应的位置值 index 是一样。

**解决哈希冲突：** 开放地址法、在散列法、建立公共溢出区、链地址法（STL 中使用就是这种方法）

<img src="/1_store/1_asset/202312052133562.png" alt="image-20230414194900802" style="zoom: 50%;" />

**装载因子**

装填因子。实际的数据个数/表的长度 = m。m 越大的话，那么冲突的概率就比较高，但是空间的利用率也比较高。m 越小的话，冲突的概率也会比较小，空间的利用率也会比较小。尽量将装载因子的大小设计在 [0.5, 0.75].   可以将数组看成是完美的哈希。



### unordered_set 的使用

<img src="/1_store/1_asset/202312052133155.png" alt="image-20230414203049541" style="zoom:80%;" />

总结：其他的查找方法与 set 完全相同，insert 插入的方法、以及 erase 方法与 set 完全相同。也没有下标访问运算符。

**针对自定义类型**

```C++
template<
    class Key,
    class Hash = std::hash<Key>,
    class KeyEqual = std::equal_to<Key>,
    class Allocator = std::allocator<Key>
> class unordered_set;
```

自定义实现：

https://gitee.com/boldness2012/cpp47_code/blob/master/20230413/unordered_set.cc#

<img src="/1_store/1_asset/202312052133986.png" alt="image-20230414214822719" style="zoom:67%;" />

**针对于 std::hash 的写法：**

<img src="/1_store/1_asset/202312052133359.png" alt="image-20230414214926837" style="zoom:67%;" />

----

<img src="/1_store/1_asset/202312052133884.png" alt="image-20230414214947041" style="zoom:55%;" />

**针对于 std::equal_to 的写法：**

<img src="/1_store/1_asset/202312052133729.png" alt="image-20230414215042580" style="zoom:50%;" />

---

<img src="/1_store/1_asset/202312052133621.png" alt="image-20230414215115833" style="zoom:50%;" />

### unordered_multiset 的使用

<img src="/1_store/1_asset/202312052133266.png" alt="image-20230414224828513" style="zoom:80%;" />

注意：unordered_multiset 的查找、插入、删除，可以参照 set 与 multiset 之间的区别。针对于自定义类型，完全与 unordered_set 等价。



### unordered_map 的使用

#### 特征：

<img src="/1_store/1_asset/202312052133811.png" alt="image-20230414221155382" style="zoom: 55%;" />

#### 下标访问（重点）：

<img src="/1_store/1_asset/202312052133125.png" alt="image-20230414221232041" style="zoom:55%;" />



### unordered_multimap 的使用

#### 基本特征

<img src="/1_store/1_asset/202312052133835.png" alt="image-20230414221616365" style="zoom: 67%;" />

#### 不支持下标：

<img src="/1_store/1_asset/202312052133627.png" alt="image-20230414221727501" style="zoom:67%;" />



## 优先级队列

```C++
template<
    class T,
    class Container = std::vector<T>,
    class Compare = std::less<typename Container::value_type> //类型萃取
> class priority_queue;
```

<img src="/1_store/1_asset/202312052133963.png" alt="image-20230414221841817" style="zoom:67%;" />

#### 针对于自定义类型

针对于自定义类型的时候，需要根据第二个模板参数中容器的 value_type 进行 std::less 的改写。std::less 有常规的三种形式，**模板的特化、函数对象的形式、运算符重载的形式**（在前面讲 set 的时候重点讲解过）。



## 容器的选择

### 如果说容器可以取下标？

vector、deque、map、unordered_map

总结：所有只要是带 set 的都没有下标，所有带 multi 的都与下标没有关系，list 没有下标。

### 元素是否有序？

关联式容器是有顺序，无序关联式容器没有，对于序列式容器而言，默认情况下与顺序没有关系，但是可以借助于对应排序函数进行排序。

### 时间复杂度？

无序关联式容器：底层使用的是哈希，如果没有冲突的话，时间复杂度 O(1).

关联式容器：底层使用的是红黑树，时间复杂度 O(logN).

序列式容器：时间复杂度 O(N)

### 迭代器的类型？

随机访问迭代器：vector、deque

双向迭代器：list、关联式容器。

前向迭代器：无序关联式容器。

没有迭代器：容器适配器 stack、queue、priority_queue



# 迭代器

输入输出流之所以可以看成是容器，因为会对应的有缓冲区，而缓冲区是可以存数据的，就可以当容器看待。

迭代器的头文件在  #include <iterator>

<img src="/1_store/1_asset/202312052133658.png" alt="image-20230415110426657" style="zoom:80%;" />

<img src="/1_store/1_asset/202312052134563.png" alt="image-20230415112537534" style="zoom:67%;" />

## 输出流迭代器

<img src="/1_store/1_asset/202312052134942.png" alt="image-20230415110523741" style="zoom:80%;" />

理解的要点是将输入/输出流作为容器看待。因此，任何接受迭代器参数的算法都可以和流一起工作。使用流迭代器必须要包含头文件 <iterator>

先介绍一个算法库的 copy 函数

```C++
Defined in header <algorithm>

template< class InputIt, class OutputIt >
OutputIt copy( InputIt first, InputIt last, OutputIt d_first );
```

### 摘取的 ostream_iterator 源码部分实现细节

```C++
class ostream_iterator
{
public:
    //ostream_iterator<int>  osi(cout, "\n");
    //ostream_type& __s = cout;  _M_stream = &cout;
    //const _CharT* __c = "\n";_M_string = "\n"
    ostream_iterator(ostream_type& __s, const _CharT* __c) 
    : _M_stream(&__s) //取得是输出流地址
    , _M_string(__c)  
    {
        
    }
    
    //osi = 1  osi对象内实现对赋值运算符的重载
    //__Tp(int) = __value(1)
    ostream_iterator<_Tp>& operator=(const _Tp& __value) 
    { 
        *_M_stream << __value;//cout << 1
        if (_M_string) //_M_string如果不为空就输入到cout内
            *_M_stream << _M_string;//cout << "\n"
        return *this;
    }
    ostream_iterator<_Tp>& operator*() 
    { 
        return *this; 
    }
    ostream_iterator<_Tp>& operator++() 
    { 
        return *this; 
    } 
    ostream_iterator<_Tp>& operator++(int) 
    { 
        return *this; 
    } 
private:
    ostream_type* _M_stream;
    const _CharT* _M_string;  
};
```

### 摘取的 copy 源码部分实现细节

```C++
//copy()函数内实现细节：
template <class _InputIter, class _OutputIter>
inline _OutputIter copy(_InputIter __first, _InputIter __last, _OutputIter __result) 
{	
	return __copy_aux(__first, __last, __result, __VALUE_TYPE(__first));//走到__copy_aux()函数内实现
}

//__copy_aux()函数内实现细节：
template <class _InputIter, class _OutputIter, class _Tp>
inline _OutputIter __copy_aux(_InputIter __first, _InputIter __last,
                              _OutputIter __result, _Tp*) 
{
	typedef typename __type_traits<_Tp>::has_trivial_assignment_operator  _Trivial;
	return __copy_aux2(__first, __last, __result, _Trivial()); //走到__copy_aux2()函数内实现
}
//__copy_aux2()函数内实现细节：
template <class _InputIter, class _OutputIter>
inline _OutputIter __copy_aux2(_InputIter __first, _InputIter __last,
                               _OutputIter __result, __false_type) 
{
    return __copy(__first, __last, __result,//走到__copy()函数内实现
                __ITERATOR_CATEGORY(__first),
                __DISTANCE_TYPE(__first));
}

//__copy()函数内实现细节：
template <class _InputIter, class _OutputIter, class _Distance>
inline _OutputIter __copy(_InputIter __first, _InputIter __last,
                          _OutputIter __result,
                          input_iterator_tag, _Distance*)
{
    for ( ; __first != __last; ++__result, ++__first)
    *__result = *__first;
    return __result;
}

//__copy函数实现剖析：
//vector<int> vec = {1, 3, 5, 9, 7};
//copy(vec.begin(), vec.end(), osi)
__first = vec.begin();
__last = vec.end();
__result = osi
_OutputIter __copy(_InputIter __first, _InputIter __last,
                          _OutputIter __result,
                          input_iterator_tag, _Distance*)
{
    for ( ; __first != __last; ++__result, ++__first)
        *__result = *__first; //osi = 3 请看上面osi对赋值运算符的重载
    return __result;
}
              last
1, 3, 5, 9, 7
              f
                  
    //osi = 1
    //__value = 1;3
    //__Tp = int
    ostream_iterator<_Tp>& operator=(const _Tp& __value) 
    { 
        *_M_stream << __value;//cout << 3
        if (_M_string) 
            *_M_stream << _M_string;//cout << "\n"
        return *this;
    }   
```





## 输入流迭代器

<img src="/1_store/1_asset/202312052134952.png" alt="image-20230415165502571" style="zoom:80%;" />

gdb 的使用

list，简写为 l，查看代码；breakpoint，简写为 b，设置断点的；run，简写为 r，让程序跑起来；step，简写为 s，单步调试；next，简写为 n，下一步；info b, 查看断点；bt 打印堆栈信息；print，简写为 p，可以打印变量的值；ptype，可以打印变量类型。

### 摘取的 ostream_iterator 源码部分实现细节

```C++
class istream_iterator
{
public:
    istream_iterator() 
    : _M_stream(0)
    , _M_ok(false) {}
    //istream_iterator<int> isi(cin)
    //istream_type& __s = cin;
    //_M_stream = &cin;
    istream_iterator(istream_type& __s) 
    : _M_stream(&__s) 
    { 
        _M_read(); 
    }
    
     void _M_read() 
     {	 // &cin && cin 
         _M_ok = (_M_stream && *_M_stream) ? true : false;
         if (_M_ok) 
         {	 
             *_M_stream >> _M_value;//cin >> _M_value  2
             _M_ok = *_M_stream ? true : false;
         }
     }
     reference operator*() const 
     { 
         return _M_value; 
     }
    
     istream_iterator& operator++() 
     { 
         _M_read(); 
         return *this;
     }
    istream_iterator operator++(int)  {
    istream_iterator __tmp = *this;
    _M_read();
    return __tmp;
  }
    //isi
    //__x = istream_iterator<int>()
   bool _M_equal(const istream_iterator& __x) const
   { 
       //_M_ok = 0;
       return (_M_ok == __x._M_ok) && (!_M_ok || _M_stream == __x._M_stream); 
   }
private:
    istream_type* _M_stream;
    _Tp _M_value;
    bool _M_ok;
};

//__first = isi = __x;
//__last = istream_iterator<int>() = __y;
template <class _Tp, class _CharT, class _Traits, class _Dist>
inline bool 
operator!=(const istream_iterator<_Tp, _CharT, _Traits, _Dist>& __x,
           const istream_iterator<_Tp, _CharT, _Traits, _Dist>& __y) {
  return !__x._M_equal(__y);
}

```

### 摘取的 copy 源码部分实现细节

```C++
//__first = isi;
//__last = istream_iterator<int>();
//__result = back_insertter(vec);
_OutputIter __copy(_InputIter __first, _InputIter __last,
                          _OutputIter __result,
                          input_iterator_tag, _Distance*)
{
  for ( ; __first != __last; ++__result, ++__first)
    *__result = *__first;
  return __result;
}

*__result   = 1;
*__result   = 2;
    
template <class _Container>
inline back_insert_iterator<_Container> back_inserter(_Container& __x) {
  return back_insert_iterator<_Container>(__x);
}
Point back_inserter(_Container& __x)
{
    return Point(x);
}
```

### 摘取的 back_insert 源码部分实现细节

注意：

```c++
void test()
{
    istream_iterator<int> isi(cin);
    vector<int> vec;
    /* copy(isi, istream_iterator<int>(), vec.begin()); */
    //对于vector而言，如果没有预留空间的话，最好使用push_back插入元素
    copy(isi, istream_iterator<int>(), std::back_inserter(vec));
 
    //可以看成是遍历vector
    copy(vec.begin(), vec.end(), ostream_iterator<int>(cout, "  "));
    cout << endl; //看打印是否成功
    
}
```



```C++
class back_insert_iterator 
{
public:
    explicit back_insert_iterator(_Container& __x) 
    : container(&__x) 
    {}
    
    back_insert_iterator<_Container>& 
    operator=(const typename _Container::value_type& __value) 
    { 
        container->push_back(__value); //插入元素
        return *this;
    }
    
    back_insert_iterator<_Container>& operator*() { return *this; }
    back_insert_iterator<_Container>& operator++() { return *this; }
    back_insert_iterator<_Container>& operator++(int) { return *this; }
protected:
  _Container* container;
}
```



## 迭代适配器

### 插入迭代器

```c++
void test()
{
    vector<int> vecNumber= {1, 3, 7, 9};
    list<int> listNumber = {11, 55, 88, 66};
    //back_inserter与back_insert_iterator底层执行了push_back函数
    /* 1、第一种方式，使用函数模板back_inserter
    copy(listNumber.begin(), listNumber.end(), back_inserter(vecNumber)); */
    
    // 2、第二种方式，使用类模板back_insert_iterator
    copy(listNumber.begin(), listNumber.end(), back_insert_iterator<vector<int>>(vecNumber));
    
    copy(vecNumber.begin(), vecNumber.end(), ostream_iterator<int>(cout, "  "));
    cout << endl;

    cout << endl << endl;
    //front_inserter与front_insert_iterator底层执行了push_front函数
    copy(vecNumber.begin(), vecNumber.end(), front_insert_iterator<list<int>>(listNumber));
    copy(listNumber.begin(), listNumber.end(), ostream_iterator<int>(cout, "  "));
    cout << endl;

    cout << endl << endl;
    //将vector中的数据插入到set中
    set<int> setNumber = {55, 33, 2, 8, 12, 19};
    auto it = setNumber.begin();
    //inserter与insert_iterator底层执行了insert函数
    copy(vecNumber.begin(), vecNumber.end(), insert_iterator<set<int>>(setNumber, it));
    copy(setNumber.begin(), setNumber.end(), ostream_iterator<int>(cout, "  "));
    cout << endl;
}

```



### 反向迭代器

<img src="/1_store/1_asset/202312052134368.png" alt="image-20230416142645138" style="zoom:67%;" />

```C++
void test()
{
    vector<int> number = {1, 3, 5, 7, 9, 8, 4, 2};
    copy(number.begin(), number.end(), ostream_iterator<int>(cout, "  "));
    cout << endl;

    //反向迭代器
    vector<int>::reverse_iterator rit;
    for(rit = number.rbegin(); rit != number.rend(); ++rit) //反向打印
    {
        cout << *rit << "  ";
    }
    cout << endl;
}
```



# 算法

## 算法分类

1、非修改式的算法，例如：count、find、**for_each**

2、修改式的算法，例如：**copy**、**remove_if**、replace、fill、swap

3、排序的算法，例如：**sort**、**lower_bound**、upper_bound

4、集合算法，例如：**set_intersection**、set_difference、set_union

5、堆的算法，例如：make_heap

6、取最值算法，例如：max、min

7、数值算法，例如：iota、accumulate

8、内存未初始化的算法，例如：**uninitialized_copy**

9、C 中的算法，例如：qsort



## for_each 的使用

```c++
template< class InputIt, class UnaryFunction >
UnaryFunction for_each( InputIt first, InputIt last, UnaryFunction f );
//UnaryFunction一元函数
```

一元函数：函数的参数是一个。二元函数：函数的参数是两个。

一元断言（谓词）：函数的参数是一个，并且函数的返回类型是 bool。
二元断言（谓词）：函数的参数是两个，并且函数的返回类型是 bool。

```c++
void print(int &value)
{
    ++value;
    cout << value << "  ";
}

void test()
{
    vector<int> vec = {1, 4, 8, 6, 7, 4};
    for_each(vec.begin(), vec.end(), print);//遍历容器的第6种方法
    //依次将vec容器内的元素赋予print函数，print自带的参数接收后，进行操作，并不会影响改变vec本身的内容
    
    //遍历容器的第5种方法
    copy(vec.begin(), vec.end(), ostream_iterator<int>(cout, "  "));
}

```

其实上面的 `for_each` 函数等级于下面的形式

```c++
for_each(vec.begin(), vec.end(), [](int &value) {
		++value;
		cout << value << "  ";
		}); //注意这里是大括号后跟小括号
//-------------------------------------------------------
[](int &value) {...} //称为匿名函数，又称lambda表达式；
//[],捕获列表，上面的式子与下面的式子等价
[](int &value) -> void {...} //函数的返回类型写在参数列表右小括号与函数体的左大括号之间，函数的返回类型是可以省略的

int a = 0;
//如果在外面定义了一个变量a,匿名函数想要使用，可以这样：
[a](int &value) -> void {...} //只读
[&a](int &value) -> void {...} //可以修改
```

<img src="/1_store/1_asset/202312052134237.png" alt="image-20230416152400347" style="zoom:67%;" />



## remove_if 算法



```c++
#include <iostream>
#include <algorithm>
#include <vector>
#include <iterator>
#include <functional>

using std::cout;
using std::endl;
using std::remove_if;
using std::copy;
using std::for_each;
using std::vector;
using std::ostream_iterator;
using std::bind1st;
using std::bind2nd;

bool func(int value)
{
    return value > 5;
}

void print(int value)
{
    cout << value << "  ";
}

void test()
{
    vector<int> vec = {1, 3, 5, 7, 5, 8, 9, 2};
    copy(vec.begin(), vec.end(), ostream_iterator<int>(cout, "  "));
    cout << endl;

    cout << endl << "测试remove_if函数" << endl;
    //remove_if不会将真正的元素删除，但是会返回待删除的元素的迭代器，然后配合erase函数进行删除
    auto it = remove_if(vec.begin(), vec.end(), func); 
    vec.erase(it, vec.end()); 
    for_each(vec.begin(), vec.end(), print);
    cout << endl;
    
    //lambda表达式
    auto it = remove_if(vec.begin(), vec.end(), [](int value)->bool{ 
                        return value > 5; 
                        }); 
    vec.erase(it, vec.end()); 
    for_each(vec.begin(), vec.end(), print);
    cout << endl; 
}


void test3()
{
    vector<int> vec;
    vec.push_back(1);

    bool flag = true;
    for(auto it = vec.begin(); it != vec.end(); ++it)
    {
        cout << *it << "  ";//读
        if(flag)
        {
            //底层已经发生了扩容,迭代器失效了
            vec.push_back(2);//写
            flag = false;
            it = vec.begin();//重置迭代器的位置
        }
    }
    cout << endl;
}

int main(int argc, char **argv)
{
    test();
    return 0;
}


```

部分源码实现：

UnaryPredicate ：一元断言

```c++
template<class InputIt, class UnaryPredicate>
constexpr InputIt find_if(InputIt first, InputIt last, UnaryPredicate p)
{
    for (; first != last; ++first) {
        if (p(*first)) {
            return first;
        }
    }
    return last;
}

template<class ForwardIt, class UnaryPredicate>
ForwardIt remove_if(ForwardIt first, ForwardIt last, UnaryPredicate p)
{
    first = std::find_if(first, last, p);
    if (first != last)
        for(ForwardIt i = first; ++i != last; )
            if (!p(*i))
                *first++ = std::move(*i);
    return first;
}

bool func(int value)
{
    return value > 5;
}
                       last
1, 3, 5, 7, 5, 8, 9, 2
         f
InputIt find_if(InputIt first, InputIt last, UnaryPredicate p)
{
    for (; first != last; ++first) 
    {
        if (p(*first)) 
        {
            return first;
        }
    }
    return last;
}

//vector<int> vec = {1, 3, 5, 7, 5, 8, 9, 2};
//remove_if(vec.begin(), vec.end(), func);

bool func(int value)
{
    return value > 5;
}
//first = vec.begin();
//last = vec.end();
//p = func
ForwardIt remove_if(ForwardIt first, ForwardIt last, UnaryPredicate p)
{
    first = std::find_if(first, last, p);
    if (first != last)
    {
        for(ForwardIt i = first; ++i != last; )
        {
             if (!p(*i))
             {
                 *first++ = std::move(*i);                
             }                    
        }        
    }
        
    return first;
}
               f        last
1, 3, 5, 5, 2, 8, 9, 2
                        i      
```



## bind 使用

注意事项：

```c++
//如果将func的第二个参数进行固定，那么func就应该具备大于符号的含义，那func就可以是greater
bool func(int lhs, int rhs = 5)
{
    return lhs > 5;
}

//如果将func的第一个参数进行固定，那么func就应该具备小于符号的含义，那func就可以是less
bool func(int lhs = 5, int rhs)
{
    return 5 < rhs;
}

struct greater
{
    bool operator()(const int &lhs, const int &rhs = 5) const
    {
        return lhs > rhs = 5;
    }
}

```

 如果 **用 bind1nd** 说明第一个参数固定：

- 用 greater 是大于，里面的实现则是 `bindnum > num` ，搭配 remove_if 使用，则是删除小于 bindnum
- 用 less 是小于，里面的实现则是 `bindnum < num ` ，搭配 remove_if 使用，则是删除大于 bindnum 的

 如果用 **bind2nd** 说明第二个参数固定：

- 用 greater 是大于，里面的实现则是 `num > bindnum` ，搭配 remove_if 使用，则是删除大于 bindnum 的
- 用 less 是小于，里面的实现则是 `num < bindnum ` ，搭配 remove_if 使用，则是删除小于 bindnum 的

<img src="/1_store/1_asset/202312052134257.png" alt="image-20230416195758658" style="zoom:67%;" />

```c++
void test2(){
    //删除的是小于5的,
    auto it = remove_if(vec.begin(), vec.end(), bind1st(std::greater<int>(), 5)); 

    //删除的是大于5的
    auto it = remove_if(vec.begin(), vec.end(), bind1st(std::less<int>(), 5));

    //删除的是大于5的
    /* auto it = remove_if(vec.begin(), vec.end(), bind2nd(std::greater<int>(), 5)); */
    vec.erase(it, vec.end());
    for_each(vec.begin(), vec.end(), print);
    cout << endl;
    //方便好记，只用bind1nd说明第一个参数固定，greater是大于，less是小于，里面的实现则是bindnumnhs
}

    //lambda表达式
    auto it = remove_if(vec.begin(), vec.end(), [](int value)->bool{ 
                        return value > 5; 
                        }); 
    vec.erase(it, vec.end()); 
    for_each(vec.begin(), vec.end(), print);
    cout << endl; 
}
```



bind1st 可以绑定二元函数对象 f 的第一个参数（也就是固定第一个参数），bind2nd 可以绑定二元函数对象的第二个参数（也就是固定第二个参数）。

那如果是 n 个参数固定呢？

<img src="/1_store/1_asset/202312052134173.png" alt="image-20230416195910592" style="zoom:80%;" />

在 C 语言中，函数名是函数的入口地址，在取地址与不取地址是一样
在 C++中，普通函数还是具备该特点，但是 C++中 **成员函数的地址**，必须要严格的用&写出来
 变量有类型 对象也有类型，都是 ok。函数也是有类型：包括函数的参数列表与函数的返回类型，函数的类型也可以称为函数的标签
全局 add 函数的类型：int(int, int) ，对于 f 而言，已经通过 bind 将 add 函数的参数都设置了值，f 的类型就是 int()
 所以 bind 是可以改变函数形态的(重要): 将二元转变为一元。

```C++
#include <iostream>
#include <functional>

using std::cout;
using std::endl;
using std::bind;

int add(int x, int y)
{
    cout << "int add(int, int)" << endl;
    return x + y;
}

int func(int x, int y, int z)
{
    cout << "int func(int, int, int)" << endl;
    return x + y + z;
}

class Example
{
public:
    int add(int x, int y)
    {
        cout << "int Example::add(int, int)" << endl;
        return x + y;
    }

    static int print(int x, int y)
    {
        cout << "int Example::print(int, int)" << endl;
        return x + y;
    }
};

void test()
{
    
    auto f = bind(add, 1, 2);
    cout <<"f() = " << f() << endl;

    cout << endl;
    //func函数的类型（标签）：int(int, int, int)
    auto f2 = bind(&func, 1, 2, 3);
    cout <<"f2() = " << f2() << endl;

    cout << endl;
    Example ex;
    //成员函数add的类型：int(Example *, int, int)
    auto f3 = bind(&Example::add, &ex, 10, 30); //非静态成员函数内含第一个默认参数对象指针，需要添加
    cout << "f3() = " << f3() << endl;

    cout << endl;
    //静态成员函数print的类型：int(int, int)
    auto f4 = bind(&Example::print, 10, 30); //静态成员函数没有this指针
    cout << "f4() = " << f4() << endl;
}

int func2()
{
    return 10;
}

int func3()
{
    return 50;
}

void test2()
{
    func2();
    //....Reactor,反应器模式
    //C语言中的函数都是普通函数，函数名就是函数的入口地址
    typedef int (*pFunc)();
    //延迟调用的思想
    pFunc f = &func2;//注册回调函数
    //...
    //...
    //...
    //...
    cout << "f() = " << f() << endl;//执行回调函数

    f = func3;
    cout << "f() = " << f() << endl;
}

int main(int argc, char **argv)
{
    test2();
    return 0;
}


```



## 回调函数的使用

```c++
auto f5 = bind(add,10,std::placeholders::_1);
//绑定第一个参数10
//占位符std::placeholders::_1

void fun5(int x1,int x2, int x3,const i)
```

function 的使用

```
function<int()> f = bind(add,10,std::placeholders::_1);
//f的函数形态int()
//返回类型由function，用来包装函数的返回类型，称为函数的容器
```

bind 绑定数据成员

- C++11 中，在声明数据成员的时候可以直接初始化

  ```c++ 
  Example ex2;
  f = bind(&Example::data, &ex2);
  cout << "f()=== " << f() << endl;
  //bind不仅可以绑定到普通函数，成员函数，还可以绑定到数据成员上面来
  ```

  



# bind 函数

std::bind 函数定义在头文件 functional 中，是一个函数模板，是函数适配器

<img src="/1_store/1_asset/202312052134012.png" alt="image-20230417194329368" style="zoom: 80%;" />



```
f	-	Callable object (function object, pointer to function, reference to function, pointer to member function, or pointer to data member) that will be bound to some arguments(参数).
```

`Callable object` 调用对象，可以是 **函数对象**（function object）、**函数指针**（pointer to function）、**函数引用**（reference to function）、**成员函数指针**（pointer to member function）、**数据成员指针**（pointer to data member）.  

```
args	-	list of arguments to bind, with the unbound arguments replaced by the placeholders _1, _2, _3... of namespace std::placeholders
```

未绑定的参数将会被占位符 _1 、 _2、 _3 所代替。

```
Return value
A function object of unspecified type T, for which std::is_bind_expression<T>::value == true. It has the following members:
```

一个未指定类型 T 的函数对象，bind 函数返回类型不确定所以用 `auto`

**函数对象就是函数名**

代码测试：

```c++
#include <iostream>
#include <functional>

using std::cout;
using std::endl;
using std::bind;
using std::function;

int add(int x, int y){
    cout << "int add(int,int)" << " = ";
    return x + y;
}

int func(int x,int y, int z){
    cout << "int func(int,int,int)" << " = ";
    return x + y + z;
}

class Example
{
public:
    int add(int x, int y)
    {
        cout << "int Example::add(int, int)" <<  " = ";
        return x + y;
    }
    static int print(int x,int y)
    {
        cout << "int Example::print(int, int)" <<  " = ";
        return x + y;
    }
    int data = 1;   //C++11,在声明数据成员的时候可以直接初始化
};

//1、bind函数可绑定多个参数
void test(){
    
    auto f = bind(add, 1, 2); //调用add函数，绑定了两个参数
    cout << "f() = " << f() << endl << endl;

    //func函数的类型（标签）：int(int,int,int)
    auto f2 = bind(&func,1,2,3); //调用func函数，绑定了三个参数,
    cout << "f2() = " << f2() << endl << endl;
}

//2、bind函数可以传入不同的调用对象
void test2(){
    //(1)传入函数对象
    auto f = bind(add, 1, 2); //调用add函数，绑定了两个参数
    cout << "f() = " << f() << endl << endl;

    //(2)传入函数指针，类型（标签）：int(int,int,int)
    auto f2 = bind(&func,1,2,3); 
    cout << "f2() = " << f2() << endl << endl;

    //(3)传入成员函数指针
    Example ex; //成员函数add的类型：int(Example*, int ,int)
    auto f3 = bind(&Example::add, &ex, 10, 30); // 成员函数隐藏this指针
    cout << "f3() = " << f3() << endl << endl;

    //静态成员函数print的类型：int(int, int)
    auto f4 = bind(&Example::print, 10, 30); //静态成员函数没有this指针
    cout << "f4() = " << f4() << endl << endl;

    //(4)传入数据成员指针
    Example ex2;	//将数据成员假看作为成员函数,是无参
    auto f5 = bind(&Example::data,&ex2); //data与ex2绑定
    cout << "f5() = Example::data = " << f5() << endl << endl;

}

//3、bind函数的占位符placeholders使用
void func_ph(int x1, int x2, int x3, const int &x4, int x5)
{
    cout << "x1 = " << x1 << endl
         << "x2 = " << x2 << endl
         << "x3 = " << x3 << endl
         << "x4 = " << x4 << endl
         << "x5 = " << x5 << endl;
}
void test3(){
    int number = 10;
    using namespace std::placeholders; //占位符
    auto f = bind(func_ph, 1, _4, _2, std::cref(number),number);
    number = 777;
    f(100, 200, 300, 400, 500, 600, 700); //多余的参数直接丢掉
    //std::ref = reference 引用
    //std::cref = const reference 常量引用
    //(1) 占位符本身代表的是形参的位置、占位符中的数字代表的是实参的位置
    //(2) 实际传参的个数一定要大于等于占位符中传递最大数字
    //(3) bind绑定的函数默认采用的是值传递,如果想使用引用传递，可以使用引用的包装器
}

//bind绑定的函数,进行传参时默认采用的是值传递
void test4(){
    Example ex; 
    auto f = bind(&Example::add, ex, 10, 30); //此处可以直接使用值传递传递ex对象
    cout << "f() = " << f() << endl;
}

int main(int argc, char **argv)
{
    test3();
    return 0;
}

```

<img src="/1_store/1_asset/202312052135936.png" alt="image-20231205213510845" style="zoom:80%;" />



从左到右分别是 test( )、test2( )、test3( ) 的执行结果。

注意事项：

> 1、在 C 语言中，函数名是函数的入口地址，在取地址与不取地址是一样；在 C++中，普通函数还是具备该特点，但 C++中成员函数的地址，必须严格写出来
> 2、变量有类型 对象也有类型，函数也是有类型：包括函数的参数列表与函数的返回类型
> 3、函数的类型也可以称为函数的标签，全局 add 函数的类型：int(int, int)
> 4、对于 `f` 而言，已经通过 bind 将 add 函数的参数都设置了值，`f` 的类型就是 `int()`，所以 bind 是可以改变函数形态的(重要)









# function 类

<img src="/1_store/1_asset/202312052135080.png" alt="image-20230417203442852" style="zoom:80%;" />

类模板 std::function 是一个通用的 **多态函数包装器**。**std::function 的实例可以存储、复制和调用任何可调用目标--函数、lambda 表达式、绑定表达式或其他函数对象，以及成员函数的指针和数据成员的指针。**

存储的可调用对象被称为 std::function 的目标。如果一个 std::函数不包含目标，它就被称为空函数。调用一个空的 std::function 的目标会导致抛出 std::bad_function_call 异常。

std::function 满足 CopyConstructible 和 CopyAssignable 的要求。

查看一下 function 类的构造函数

<img src="/1_store/1_asset/202312052135624.png" alt="image-20230417205350430" style="zoom:67%;" />

<img src="https://gitee.com/demoCanon/pic-bed/raw/master/img/202305171009580.png" alt="image-20230417205604953" style="zoom:67%;" />

**既然这样，std::function 的实例也可以接收 bind 函数返回的函数对象，并对其可以进行操作。**

```c++
void test4()
{
    //func函数的类型（标签）：int(int, int, int)
    //f的函数类型是int()
    function<int()> f = bind(&func, 1, 2, 3); 
    cout <<"f() = " << f() << endl << endl;

    //成员函数add的类型：int(Example *, int, int)
    Example ex;	
    //f2的函数类型是int()
    function<int()> f2 = bind(&Example::add, &ex, 10, 30);
    cout << "f2() = " << f2() << endl << endl;

    //add函数类型:int(int, int)
    //f3的函数类型是int(int)，因为有一个占位符存在
    function<int(int)> f3 = bind(add, 10, std::placeholders::_1);
    cout << "f3(90) = " << f3(90) << endl << endl; //占位符传入90

   	//将数据成员假看作为成员函数,是无参
    Example ex2;
    function<int()> f4 = bind(&Example::data, &ex2);
    cout << "f4() === " << f4() << endl;
}
```

<img src="/1_store/1_asset/202312052135134.png" alt="image-20230417213226681" style="zoom:80%;" />

注意事项：

> 1、对于 f 而言，已经通过 bind 将 add 函数的参数都设置了值，`f` 的类型就是 `int()`
>
> 2、而使用占位符后，就有了一个参数，`f` 的类型就是 `int(int)`
>
> 3、`f` 的函数形态 `int()` ，function 称为函数的容器



# 回调函数

说到回调函数自然离不开函数指针的使用，先讲解一下函数指针的使用。



## 函数指针

声明一个函数指针： `void (*foo)();`

- `()` 说明 `foo` 是一个函数，无参数
- `(*foo)` 说明 `foo` 是个指针，即函数指针变量的名字
- `void` 说明 `foo` 的返回类型是 `void`

例如：

```c++
int (*f1)(double);	//传入double，返回int
void (*f2)(char*);	//传入char指针，没有返回值
double* (*f3)(int,int);	//传递两个整数，返回double指针
```

函数指针的特征：用小括号将 ”指针“ 的星号与函数名绑定在一起。



## 使用函数指针

```c++
int square(int num){ // 定义一个int(int)的函数
    return num * num;
}
void test(){
    int (*fptr1)(int); //声明一个int(int)的函数指针

    int n = 5;
    fptr1 = square;	//将square函数地址赋予函数指针fptr1

    printf("%d squared is %d\n",n, fptr1(n)); //向函数对象内传参，调用执行函数
    //如果是无参，则是fptr1()
}
```

这就看出来函数指针的声明的过程就像是声明一个变量，最后为这个变量赋值。

## 类型定义

类比为变量的数据类型起别名，**为函数指针声明一个类型定义** 会比较方便。

通常，类型定义的名字是声明的最后一个元素。

```c++
int square(int num){ // 定义一个int(int)的函数
    return num * num;
}
void test(){
    //声明一个函数指针的类型定义
    typedef int (*funcptr)(int); //所以函数指针的类型定义为funcptr，funcptr就是那个元素

    int n = 5;
    funcptr fptr2 = square;	//funcptr初始化一个函数对象，并初始化为square函数地址

    printf("%d squared is %d\n",n, fptr2(n)); //向函数对象内传参，调用执行函数
    //如果是无参，则是fptr2()
}
```



## 传递函数指针

## 返回函数指针



## 回调函数

注意：C 语言中的函数都是普通函数，函数名就是函数的入口地址。而在 C++中，像成员函数必须用&传递函数地址

```c++
void demo()
{
    func2();
   
    typedef int (*pFunc)(); //声明一个函数指针的类型定义
    //延迟调用的思想
    pFunc f = &func2;//初始化一个函数指针对象，这一过程就叫做注册回调函数
    cout << "f() = " << f() << endl;//执行回调函数
}
```



# 多态的实现方式

## 面向对象

通过 **面向对象** 的思想实现多态

```c++
//抽象类，作为接口使用
class Figure
{
public:
    //纯虚函数
    virtual void display() const = 0;
    virtual double area() const = 0;
};
//省略Rectangle、Circle、Triangle类的实现过程，都是继承的Figure类
void func(const Figure &fig)
{
    fig.display();
    cout << "的面积 : " << fig.area() << endl;
}

int main(int argc, char **argv)
{
    Rectangle rectangle(10, 20);
    Circle circle(10);
    Triangle triangle(3, 4, 5);

    func(rectangle);
    func(circle);
    func(triangle);

    return 0;
}
```

<img src="/1_store/1_asset/202312052135082.png" alt="image-20230417222121232" style="zoom:80%;" />

## 面向过程

通过 **面向过程** 的思想实现多态

关于 `using` 的小知识点：

```c++
typedef function<void()> DisplayCallback;	//C++98 

using  DisplayCallback = function<void()>;	//C++11, 等价于c++98的类型定义别名
using  AreaCallback = function<double()>;	//C++11 
```



```c++
//抽象类，作为接口使用
class Figure
{
public:
    //virtual void display() const = 0;
    //virtual double area() const = 0;

    using DisplayCallback = function<void()>;
    using AreaCallback = function<double()>;

    //定义数据成员
    DisplayCallback _displayCallback; //定义的是一个函数对象
    AreaCallback _areaCallback; //定义的是一个函数对象

    //注册回调函数
    void setDisplayCallback(DisplayCallback &&cb){
        _displayCallback = std::move(cb);
    }
    void setAreaCallback(AreaCallback &&cb){
        _areaCallback = std::move(cb);
    }

    //执行回调函数
    void handleDisplayCallback() const {
        if(_displayCallback){ //如果函数对象不为空
            _displayCallback(); //调用执行函数
        }
    }
    double handleAreaCallback() const {
        if(_areaCallback){  //如果函数对象不为空
            return _areaCallback();  //调用执行函数,并返回函数执行结果
        }
        else{
            return 0.0;
        }
    }
};
//省略Rectangle、Circle、Triangle类的实现过程，都没有继承Figure类

void func(const Figure &fig) //const对象只能调用const成员函数
{
    fig.handleDisplayCallback();
    cout << "的面积 : " << fig.handleAreaCallback() << endl;
}

int main(int argc, char **argv)
{
    Rectangle rectangle(10, 20);
    Circle circle(10);
    Triangle triangle(3, 4, 5);

    //1、初始化一个对象
    Figure fig;

    //2、回调函数的注册
    //DisplayCallback cb = bind(&Rectangle::display,&rectangle);
    fig.setDisplayCallback(bind(&Rectangle::display,&rectangle)); //调用打印函数
    fig.setAreaCallback(bind(&Rectangle::area,&rectangle)); //执行面积计算函数
    func(fig);

    fig.setDisplayCallback(bind(&Circle::display,&circle));
    fig.setAreaCallback(bind(&Circle::area,&circle));
    func(fig);

    fig.setDisplayCallback(bind(&Triangle::display,&triangle));
    fig.setAreaCallback(bind(&Triangle::area,&triangle));
    func(fig);
    

    return 0;
}


```

<img src="/1_store/1_asset/202312052135811.png" alt="image-20230417230633526" style="zoom:80%;" />



# 成员函数适配器 mem_fn

成员函数适配器 **mem_fn**

对于 C++中的算法库中函数，如果用到成员函数，需要使用 **成员函数适配器** 进行包装一下

**member object** 成员对象: 一个类的成员变量是另一个类的对象。

<img src="/1_store/1_asset/202312052135564.png" alt="image-20230418185921454" style="zoom: 80%;" />

函数模板 `std::mem_fn` 生成指向成员指针的包装对象，它可以存储、复制及调用指向成员指针。

对象的引用和指针（含智能指针）可在调用 `std::mem_fn` 时使用。 pointers to members 可以指的是数据成员（包括成员对象）、成员函数

`pm` - 指向被包装成员的指针	返回值： 返回未指定类型的函数对象（其实就是包装成一个有参的普通函数）

<img src="/1_store/1_asset/202312052135758.png" alt="image-20230418200042878" style="zoom:67%;" />

Example：Use `mem_fn` to store and execute a member function and a member object

```c++
#include <functional>
#include <iostream>
 
struct Foo {
    void display_greeting() {
        std::cout << "Hello, world.\n";
    }
    void display_number(int i) {
        std::cout << "number: " << i << '\n';
    }
    int data = 7;
};
 
int main() {
    Foo f; //结构对象

    //无参成员函数被包装成函数对象greet
    auto greet = std::mem_fn(&Foo::display_greeting); 
    greet(f); //调用函数对象时，传入结构对象
    
    //有参成员函数被包装成函数对象print_num
    auto print_num = std::mem_fn(&Foo::display_number);
    print_num(f, 42);//调用函数对象时，传入结构对象
    
    //数据成员被包装成函数对象access_data
    auto access_data = std::mem_fn(&Foo::data);
    std::cout << "data: " << access_data(f) << '\n';
}
```

那看到这既然这样，mem_fn 与 bind 的作用差不多，但是 bind 的主要作用是在固定参数上，mem_fn 的主要作用是在包装成员上。



# 函数对象（仿函数）

**可以与小括号进行结合体现函数的含义的，统称为函数对象** 

1、重载了 **函数调用运算符** 的类创建的对象。

2、函数名

3、函数指针

# 空间配置器（重难点）

## **基本概念**

空间配置器 **std::allocator** 将空间的申请与对象的构建分开了，将空间的释放与对象的销毁分开了。

在 C++中所有 STL 容器的空间分配其实都是使用的 **std::allocator**, 它是可以感知类型的空间分配器，并将空间的申请与对象的构建分离开来。

单个对象使用 new 的时候，空间的申请与对象的构造好像是没有分开的，连在一起了，但是对于容器 vector 而言，其有个函数叫做 reserve 函数，先申请空间，然后在在该空间上构建对象。

为什么要将空间的申请与对象的构建分开呢？因为对于批量元素而言，肯定不会是将申请的空间全部的占满，肯定不是申请一个空间，然后在该空间构建对象，然后再申请一个空间，接着再构建对象。

对于 STL 中，容器可能一次需要存多个元素，需要将空间的申请与对象的构建分开，不然就变成了申请一块空间，创建一个对象，频繁进行操作。



## 基本使用

```c++
//空间的申请,申请的是原始的，未初始化的空间
pointer allocate( size_type n, const void * hint = 0 );
T* allocate( std::size_t n, const void * hint);
T* allocate( std::size_t n );

//空间的释放
void deallocate( T* p, std::size_t n );

//对象的构建，在指定的未初始化的空间上构建对象，使用的是定位new表达式
void construct( pointer p, const_reference val );

//对象的销毁
void destroy( pointer p );
```

一自定义的 MyVector 为例，看看代码实现。

https://gitee.com/boldness2012/cpp47_code/blob/master/20230417/MyVector.cc

## 原理

16 个自由链表 + 内存池。

<img src="/1_store/1_asset/202312052135573.png" alt="image-20230517100608369" style="zoom:67%;" />

源码

```c++
//第一级空间配置器
# ifdef __USE_MALLOC
typedef malloc_alloc alloc;
typedef __malloc_alloc_template<0> malloc_alloc;
class __malloc_alloc_template 
{
public:
	static void* allocate(size_t __n)
	{
      void* __result = malloc(__n);
      if (nullptr == __result) 
        __result = _S_oom_malloc(__n);//oom = out of memory

      return __result;
    }

    static void deallocate(void* __p, size_t /* __n */)
    {
		free(__p);
    }
};
#else
//第二级空间配置器（默认的空间配置器）
typedef __default_alloc_template<__NODE_ALLOCATOR_THREADS, 0> alloc;
class __default_alloc_template 
{
public:
    static void* allocate(size_t __n)
    {
		if(__n > 128)
		{
			malloc(__n);
		}
		else
		{
			//16个自由链表 + 内存池
			//1、减少内存碎片
			//2、内核态与用户态之间频繁的进行切换，效率比较低
		}
	}
	
	//当空间使用完毕后，重新链接会自由链表
	static void deallocate(void* __p, size_t __n)
    {
		if (__n > 128)
			free(__p);
		
		else {
			_Obj**  __my_free_list = _S_free_list + _S_freelist_index(__n);//_S_free_list[3]
			_Obj* __q = (_Obj*)__p;
			
			__q -> _M_free_list_link = *__my_free_list;
			*__my_free_list = __q;
			}
    }
};

#endif

class allocator 
{
	typedef alloc _Alloc;
public:
	_Tp* allocate(size_type __n, const void* = nullptr) 
	{
		return __n != 0 ? static_cast<_Tp*>(_Alloc::allocate(__n * sizeof(_Tp))) : 0;
	}

    // __p is not permitted to be a null pointer.
    void deallocate(pointer __p, size_type __n)
    { 
		_Alloc::deallocate(__p, __n * sizeof(_Tp)); 
	}
	
	//构建对象
	void construct(pointer __p, const _Tp& __val) 
	{ 
	    //new Point(1, 2)
		new(__p) _Tp(__val); //定位new表达式
	}
	//对象的销毁
    void destroy(pointer __p) 
	{ 
		__p->~_Tp(); 
	}
};







enum {_ALIGN = 8};
enum {_MAX_BYTES = 128};
enum {_NFREELISTS = 16}; 

union _Obj 
{
	union _Obj* _M_free_list_link;
    char _M_client_data[1];    /* The client sees this.        */
};


//__bytes = 32
static  size_t _S_freelist_index(size_t __bytes) 
{
	return (((__bytes) + (size_t)_ALIGN-1)/(size_t)_ALIGN - 1);
	(32 + 8 - 1)/8 - 1 = 39/8 - 1 = 4.x - 1 = 3.x = 3
}

//__bytes = 32
//[25, 32]  ==== 32
25/8 = 3.x  32/8 = 4   32
//向上取整，得到8的整数倍
static size_t _S_round_up(size_t __bytes) 
{ 
    return (((__bytes) + (size_t) _ALIGN-1) & ~((size_t) _ALIGN - 1));
        (32 + 8 - 1) & ~ (8 - 1);
		39 & ~7
		7 = 0000 0111
		~7 = 1111 1000
		
		39 = 32 + 4 + 2 + 1
		39 = 0010 0111
		
		0010 0111
	&	1111 1000
	    0010 0000  = 32
		
		0010 0110
	&	1111 1000
	    0010 0000  = 32
		
		0010 0000
	&	1111 1000
	    0010 0000  = 32
		
		0001 1111
	&	1111 1000
	    0001 1000  = 24
		
		0010 1000
	&	1111 1000
	    0010 1000  = 40
}

template <bool __threads, int __inst>
typename __default_alloc_template<__threads, __inst>::_Obj* __STL_VOLATILE
__default_alloc_template<__threads, __inst> ::_S_free_list[
# if defined(__SUNPRO_CC) || defined(__GNUC__) || defined(__HP_aCC)
    _NFREELISTS
# else
    __default_alloc_template<__threads, __inst>::_NFREELISTS
# endif
] = {0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, };


template <bool __threads, int __inst>
char* __default_alloc_template<__threads, __inst>::_S_start_free = nullptr;

template <bool __threads, int __inst>
char* __default_alloc_template<__threads, __inst>::_S_end_free = nullptr;

template <bool __threads, int __inst>
size_t __default_alloc_template<__threads, __inst>::_S_heap_size = 0;





//1、申请32字节时候，保证空间是足够的
//__size = 32, __nobjs = 20
char* __default_alloc_template::_S_chunk_alloc(size_t __size, int& __nobjs)
{
    char* __result;
    size_t __total_bytes = __size * __nobjs = 32 * 20 = 640;
    size_t __bytes_left = _S_end_free - _S_start_free = 0;
	
	else
	{
		 size_t __bytes_to_get = 2 * __total_bytes + _S_round_up(_S_heap_size >> 4)
		                            = 2 * 640  = 1280;
		_S_start_free = (char*)malloc(__bytes_to_get) = malloc(1280);
						  
		_S_heap_size += __bytes_to_get = 1280;
        _S_end_free = _S_start_free + __bytes_to_get;
        return(_S_chunk_alloc(__size, __nobjs));
	}
	//递归调用_S_chunk_alloc
	char* __result;
    size_t __total_bytes = __size * __nobjs = 32 * 20 = 640;
    size_t __bytes_left = _S_end_free - _S_start_free = 1280;
	
	 if (__bytes_left >= __total_bytes) 
	 {
        __result = _S_start_free;
        _S_start_free += __total_bytes;
        return(__result);
    }
}

//__n = 32
void* __default_alloc_template::_S_refill(size_t __n)
{
	int __nobjs = 20;
    char* __chunk = _S_chunk_alloc(__n, __nobjs);
	_Obj** __my_free_list;
    _Obj* __result;
    _Obj* __current_obj;
    _Obj* __next_obj;
    int __i;
	
	__my_free_list = _S_free_list + _S_freelist_index(__n);//_S_free_list[3]
	
	__result = (_Obj*)__chunk;
    *__my_free_list = __next_obj = (_Obj*)(__chunk + __n);
      for (__i = 1; ; __i++) 
	  {
        __current_obj = __next_obj;
        __next_obj = (_Obj*)((char*)__next_obj + __n);
        if (__nobjs - 1 == __i) {
            __current_obj -> _M_free_list_link = 0;
            break;
        } else {
            __current_obj -> _M_free_list_link = __next_obj;
        }
}

//__n = 32
static void* allocate(size_t __n)
{
	else
	{
		_Obj** __my_free_list = _S_free_list + _S_freelist_index(__n);//_S_free_list[3]
		
		_Obj*  __result = *__my_free_list;
        if (__result == nullptr)
			__ret = _S_refill(_S_round_up(__n));
        else 
		{
			*__my_free_list = __result -> _M_free_list_link;
			__ret = __result;
        }
	}
}



//2、申请64字节时候，保证空间是足够的
//__size = 64,__nobjs = 20
char*__default_alloc_template::_S_chunk_alloc(size_t __size, int& __nobjs)
{
    char* __result;
    size_t __total_bytes = __size * __nobjs = 64 * 20 = 1280;
    size_t __bytes_left = _S_end_free - _S_start_free = 640;
	
	else if (__bytes_left >= __size) 
	{
        __nobjs = (int)(__bytes_left/__size) = 640/64 = 10;
        __total_bytes = __size * __nobjs = 64 * 10 = 640;
        __result = _S_start_free;
        _S_start_free += __total_bytes;
        return(__result);
    }
}

//__n = 64
void* __default_alloc_template::_S_refill(size_t __n)
{
    int __nobjs = 20;
    char* __chunk = _S_chunk_alloc(__n, __nobjs);
    _Obj* __STL_VOLATILE* __my_free_list;
    _Obj* __result;
    _Obj* __current_obj;
    _Obj* __next_obj;
    int __i;
	
	 __my_free_list = _S_free_list + _S_freelist_index(__n);//_S_free_list[7]

 
      __result = (_Obj*)__chunk;
      *__my_free_list = __next_obj = (_Obj*)(__chunk + __n);
      for (__i = 1; ; __i++) 
	  {
        __current_obj = __next_obj;
        __next_obj = (_Obj*)((char*)__next_obj + __n);
        if (__nobjs - 1 == __i) 
		{
            __current_obj -> _M_free_list_link = 0;
            break;
        } else {
            __current_obj -> _M_free_list_link = __next_obj;
        }
      }

}

//__n = 64
static void* allocate(size_t __n)
{
    void* __ret = 0;
	else 
	{
      _Obj* * __my_free_list = _S_free_list + _S_freelist_index(__n);//_S_free_list[7]
	  _Obj*  __result = *__my_free_list;
      if (__result == 0)
        __ret = _S_refill(_S_round_up(__n));
      else {
        *__my_free_list = __result -> _M_free_list_link;
        __ret = __result;
      }
	}
}


//3、申请96字节时候，保证空间是足够的
//__size = 96,__nobjs = 20
char*__default_alloc_template::_S_chunk_alloc(size_t __size, int& __nobjs)
{
    char* __result;
    size_t __total_bytes = __size * __nobjs = 96 * 20 = 1920;
    size_t __bytes_left = _S_end_free - _S_start_free = 0;
	
	else {
        size_t __bytes_to_get = 2 * __total_bytes + _S_round_up(_S_heap_size >> 4)
		                           = 2 * 1920 + _S_round_up(1280 >> 4)
								   = 3840 + 80 = 3920;
		 _S_start_free = (char*)malloc(__bytes_to_get) = malloc(3920);
		 
		 _S_heap_size += __bytes_to_get = 1280 + 3920 = 5200;
        _S_end_free = _S_start_free + __bytes_to_get;
        return(_S_chunk_alloc(__size, __nobjs));
	}
	//递归调用_S_chunk_alloc
	 char* __result;
    size_t __total_bytes = __size * __nobjs = 96 * 20 = 1920;;
    size_t __bytes_left = _S_end_free - _S_start_free = 3920;
	
	if (__bytes_left >= __total_bytes) 
	{
        __result = _S_start_free;
        _S_start_free += __total_bytes;
        return(__result);
    }
	
}
//__n = 96
void* __default_alloc_template::_S_refill(size_t __n)
{
    int __nobjs = 20;
    char* __chunk = _S_chunk_alloc(__n, __nobjs);
    _Obj* __STL_VOLATILE* __my_free_list;
    _Obj* __result;
    _Obj* __current_obj;
    _Obj* __next_obj;
    int __i;
	
	 __my_free_list = _S_free_list + _S_freelist_index(__n);//_S_free_list[11]

    /* Build free list in chunk */
      __result = (_Obj*)__chunk;
      *__my_free_list = __next_obj = (_Obj*)(__chunk + __n);
      for (__i = 1; ; __i++) {
        __current_obj = __next_obj;
        __next_obj = (_Obj*)((char*)__next_obj + __n);
        if (__nobjs - 1 == __i) {
            __current_obj -> _M_free_list_link = 0;
            break;
        } else {
            __current_obj -> _M_free_list_link = __next_obj;
        }
      }
}
//__n = 96
static void* allocate(size_t __n)
{
    void* __ret = 0;
	else 
	{
      _Obj* * __my_free_list = _S_free_list + _S_freelist_index(__n);//_S_free_list[11]
	  _Obj*  __result = *__my_free_list;
      if (__result == nullptr)
        __ret = _S_refill(_S_round_up(__n));
      else {
        *__my_free_list = __result -> _M_free_list_link;
        __ret = __result;
      }
	}
}


//4、申请72字节时候，内存池与堆空间都没有连续的72字节
//此时72字节是从96字节借过来。
//__size = 72,__nobjs = 20
char*__default_alloc_template::_S_chunk_alloc(size_t __size, int& __nobjs)
{
    char* __result;
    size_t __total_bytes = __size * __nobjs = 72 * 20 = 1440;
    size_t __bytes_left = _S_end_free - _S_start_free = 0;
	
	else {
        size_t __bytes_to_get = 2 * __total_bytes + _S_round_up(_S_heap_size >> 4)
		                           = 2 * 1440 + _S_round_up(5200 >> 4) > 2880;
		_S_start_free = (char*)malloc(__bytes_to_get);
		 if (0 == _S_start_free) 
		 {
            size_t __i;
            _Obj* * __my_free_list;
	        _Obj* __p;
			
			//__size = 72,_MAX_BYTES = 128,_ALIGN = 8
			//72 80  88  96
			////_S_free_list[8]   _S_free_list[9]  _S_free_list[10]  _S_free_list[11]
			for (__i = __size; __i <= (size_t) _MAX_BYTES; __i += (size_t) _ALIGN) 
			{
				__my_free_list = _S_free_list + _S_freelist_index(__i);//_S_free_list[11]
                __p = *__my_free_list;
				
				if (0 != __p) {
                    *__my_free_list = __p -> _M_free_list_link;
                    _S_start_free = (char*)__p;
                    _S_end_free = _S_start_free + __i;
                    return(_S_chunk_alloc(__size, __nobjs));
                    // Any leftover piece will eventually make it to the
                    // right free list.
                }
			}
		 }
	}
	//递归调用_S_chunk_alloc
	char* __result;
    size_t __total_bytes = __size * __nobjs = 72 * 20 = 1440;
    size_t __bytes_left = _S_end_free - _S_start_free = 96;
	
	else if (__bytes_left >= __size) 
	{
        __nobjs = (int)(__bytes_left/__size) = 1;
        __total_bytes = __size * __nobjs = 72 ;
        __result = _S_start_free;
        _S_start_free += __total_bytes;
        return(__result);
    }

}


//__n = 72
void* __default_alloc_template::_S_refill(size_t __n)
{
    int __nobjs = 20;
    char* __chunk = _S_chunk_alloc(__n, __nobjs);
    _Obj* __STL_VOLATILE* __my_free_list;
    _Obj* __result;
    _Obj* __current_obj;
    _Obj* __next_obj;
    int __i;
	
	if (1 == __nobjs) return(__chunk);
}


//__n = 72
static void* allocate(size_t __n)
{
    void* __ret = 0;
	else 
	{
      _Obj* * __my_free_list = _S_free_list + _S_freelist_index(__n);//_S_free_list[8]
	  _Obj*  __result = *__my_free_list;
      if (__result == nullptr)
        __ret = _S_refill(_S_round_up(__n));
      else {
        *__my_free_list = __result -> _M_free_list_link;
        __ret = __result;
      }
	}
}
总结：
1、空间配置器申请的空间在哪里？都是在堆空间
2、空间申请对应的函数。
allocate：该函数是进行申请空间的，但是该函数会调用_S_refill。
_S_refill：本身不申请空间，只是将申请回来的空间进行切割，然后进行挂接在16个自由链表下面。该函数会调用_S_chunk_alloc进行真正的空间申请。
_S_chunk_alloc：本函数才是真正进行申请空间的函数，本函数会调用malloc或者直接在内存池中拿取内存，一般会申请比较大的一块空间，其中一部分返回给_S_refill，另外一部分会留在内存池中，被_S_start_free，_S_end_free进行控制。_S_chunk_alloc函数有可能会被递归调用。
_S_freelist_index：获取自由链表对应的下标
_S_round_up：向上取整，得到8的整数倍。
```

