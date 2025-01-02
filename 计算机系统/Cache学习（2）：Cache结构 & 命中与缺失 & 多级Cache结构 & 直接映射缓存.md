# 1. Cache 名词解释

- 命中（hit）： CPU 要访问的数据在 Cache 中有缓存
- 缺失（miss）： CPU 要访问的数据在 Cache 中没有缓存
- Cache Size：Cache 的大小，代表 Cache 可以缓存最大数据的大小
- Cache Line：Cache 会被平均分成很多相等的块，每一个块称之为 Cache Line

Cache 会被平均分成每一个块的大小称之为 Cache Line Size；Cache Line 是 Cache 和主存之间数据传输的最小单位。

当 CPU 试图加载 1 Byte 数据的时候，如果 Cache 缺失，那么 Cache 控制器会从主存中一次性的加载一个 Cache Line 大小的数据到 Cache 中。例如，Cache Line 大小是 8 Byte。即使 CPU 读取 1 Byte 数据，然而在 Cache 出现 miss 后，Cache 也还是会从主存中加载 8 Byte 来填充整个 Cache Line。

Cache Size 为 64 Bytes 的 Cache 举两个例子：

- 将 64 Bytes 平均分成 64 块，那么 Cache Line 就是 1 Byte，总共 64 行 Cache Line
- 将 64 Bytes 平均分成 8 块，那么 Cache Line 就是 8 Byte，总共 8 行 Cache Line

现在的硬件设计中，一般 Cache Line 的大小是 4-128 Bytes。会有如下两个问题：

- Cache 如何判断是否命中
- 这个值为什么不是更低，低至 1 Byte，这样就可以更加灵活的映射，从而没有刷新整个 cache line 的开销
- 这个值为什么不是更高或者更低
- 为什么一次要读取整个 Cache Line

后文中进行解释说明

# 2. 多级 Cache 之间的配合工作

当 CPU 试图从某地址加载数据时，下图为只有两级 Cache 的系统举例：

- 从 L1 Cache 中查询是否命中，如果命中则把数据返回给 CPU（蓝色实线）
- L1 Cache 缺失，则继续从 L2 Cache 中查找。当 L2 Cache 命中时，数据会返回给 L1 Cache 以及 CPU（绿色实线&绿色虚线）
- L2 Cache 也缺失，需要从主存中 load 数据，将数据返回给 L2 Cache、L1 Cache 及 CPU（红色实线&红色虚线）

![[20240914174854.png|600]]

这种多级 Cache 的工作方式称之为 Inclusive Cache（包含式缓存）。某一地址的数据可能存在多级缓存中。与 Inclusive Cache 对应的是 Exclusive Cache（独立式缓存）。这种 Cache 保证某一地址的数据缓存只会存在于多级 Cache 的其中一级。也就是说，任意地址的数据不可能同时在 L1 和 L2 Cache 中缓存。

# 3. 直接映射缓存

直接映射缓存（Direct Mapped Cache）

## 3.1 概念简介

以一个 Cache Size 为 128 Bytes 并且 Cache Line 是 16 Bytes 的 Cache 为例。首先把这个 Cache 想象成一个数组，数组总共 8 个元素，每个元素大小是 16 Bytes，如下图：

![[20240914180245.png|600]]

现在考虑一个问题，CPU 从 0x0654 地址读取一个字节，由于 Cache 相对于主存来说，容量是非常小的。所以 Cache 只能缓存主存中极小一部分数据。如何根据地址在有限大小的 Cache 中查找数据呢？现在硬件采取的做法是对地址进行散列（可以理解成地址取模操作）。

## 3.2 命中与缺失

经过如下计算：

- 假设地址总线是 16 位，目标地址为 0x0654，转换为二进制为 `0000,0110,0101,0100`
- Offset：由于每个 Cache Line 中有 16 Byte，所以地址可最低 4 位来寻址，即为每一个 Cache Line 中的偏移 Offset，标记在这个 Cache Line 中的具体位置是哪个字节，举例中为 0100，即为图中地址段的蓝色背景部分
- Index：由于一共有 8 个 Cache Line，所以地址除去最低 4 位的后 3 位，即为不同 Cache Line 的索引 Index，标记具体在整个 Cache 中的那一个 Cache Line，举例中为 101，即为图中地址段的绿色背景部分

![[20240914180643.png|600]]

如果两个不同的地址，其地址的 Index 部分完全一样，这两个地址经过硬件散列之后都会找到同一个 Cache Line。所以，根据地址确定到 Cache Line 之后，只代表所需要访问的目标地址中存储的对应数据可能存在这个 Cache Line 中，但是该 Cache Line 也有可能存储其他地址对应的数据。

所以，独立于 Data Array，又引入 Tag Array 区域，Tag Array 和 Data Array 中的每一个 Cache Line 都有着一一对应关系。每一个 Cache Line 都对应唯一一个 tag，tag 中保存的是整个地址位宽去除 index 和 offset 使用的 bit 剩余部分（如上图地址粉色背景部分）。tag、index 和 offset 三者组合就可以唯一确定一个地址了。

因此，根据地址中 index 位找到 Cache Line 后，取出当前 Cache Line 对应的 tag，然后和目标地址的 tag 进行比较，如果相等，这说明 Cache 命中。如果不相等，说明当前 Cache Line 存储的是其他地址的数据，这就是 Cache 缺失。

在上述图中，我们看到 tag 的值是 `0,0000,1100`，和地址中的 tag 部分相等，因此在本次访问会命中。

我们可以从图中看到 tag 旁边还有一个 valid bit，这个 bit 用来表示 Cache Line 中数据是否有效（例如：1 代表有效；0 代表无效）。当系统刚启动时，Cache 中的数据都应该是无效的，因为还没有缓存任何数据。Cache 控制器可以根据 valid bit 确认当前 Cache Line 数据是否有效。所以，上述比较 tag 确认 Cache Line 是否命中之前还会检查 valid bit 是否有效。只有在有效的情况下，比较 tag 才有意义。如果无效，直接判定 Cache 缺失。

此时回答，前文提出的第二个问题：这个值为什么不是更低？低至 1 字节，这样就可以更加灵活的映射，从而避免了因为部分所需要的数据而刷新整个 Cache Line 的开销。（理解：Cache 平分的越细，CacheLine 容量就越小，数量就越多，避免因一次缺失而刷新整个 CacheLine 的开销，CacheLine 容量越大，开销越大嘛）

> [!NOTE]
>
> 由于tag的引入。这样会导致硬件成本的上升，将两种情况进行对比：
>
> - 原本Cache Line 设置为16 Byte：每16 Byte对应一个tag，需要8个tag
> - 假设Cache Line设置为1 Byte：需要128个Tag同时每一个Tag的长度也会更长，因为Offest缩短了因此可以发现这样做占用了很多内存。需要注意：tag也是Cache的一部分，但是谈到Cache size的时候并不考虑tag占用的内存部分。

上面的例子中，总结如下：Cache Size 是 128 Byte 并且 Cache Line size 是 16 Byte，共计 8 个 Cache Line。

- offset：4bit
- index：3bit
- tag：9bits（假设地址宽度是 16 bit）

## 3.3 优缺点

- 优点 1：直接映射缓存在硬件设计上会更加简单
- 优点 2：因为优点 1，所以成本上也会较低

根据直接映射缓存的工作方式，可以计算出不同主存地址段和对应的 Cache：


|              地址段              | Cache Line Index |
| :---------------------------: | :--------------: |
| 0x0000-0x000F，0x0080-0x008F，… |        0         |
| 0x0010-0x001F，0x0090-0x009F，… |        1         |
| 0x0020-0x002F，0x00A0-0x00AF，… |        2         |
| 0x0030-0x003F，0x00B0-0x00BF，… |        3         |
| 0x0040-0x004F，0x00C0-0x00CF，… |        4         |
| 0x0050-0x005F，0x00D0-0x00DF，… |        5         |
| 0x0060-0x006F，0x00E0-0x00EF，… |        6         |
| 0x0070-0x007F，0x00F0-0x00FF，… |        7         |
可以看到，地址 0x0000-0x007F 地址（0x0000-0x000F ~ 0x0070-0x007F）处对应的数据可以覆盖整个 Cache。0x0080-0x00FF 地址的数据也同样是覆盖整个 Cache。

现在思考一个问题，如果一个程序试图依次访问地址 0x0000、0x0080、0x0100，Cache 中的数据会发生什么呢？首先应该明白 0x0000、0x0080、0x0100 地址中 index 部分是一样的。因此，这 3 个地址对应的 Cache Line 是同一个。所以，当访问 0x0000 地址时，Cache 会缺失，然后数据会从主存中加载到 Cache 中第 0 行 Cache Line。

当我们访问 0x0080 地址时，依然索引到 Cache 中第 0 行 Cache Line，由于此时 Cache Line 中存储的是地址 0x0000 地址对应的数据，所以此时依然会 Cache 缺失。然后从主存中加载 0x0080 地址数据到第一行 Cache Line 中。

同理，继续访问 0x0100 地址，依然会 Cache 缺失。这就相当于每次访问数据都要从主存中读取，所以 Cache 的存在并没有对性能有什么提升。访问 0x0080 地址时，就会把 0x00 地址缓存的数据替换。这种现象叫做 Cache 颠簸（Cache thrashing）。针对这个问题，在后面的文章中引入多路组相连缓存优化规避这一问题。

![[20240914183304.png|600]]





# 参考资料

- [Cache 学习（2）：Cache 结构 & 命中与缺失 & 多级 Cache 结构 & 直接映射缓存](https://blog.csdn.net/qq_41554005/article/details/134589493)