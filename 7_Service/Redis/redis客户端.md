github地址： 
https://github.com/redis/redis 

官网地址：
https://redis.io/

redis客户端：
https://github.com/sewenew/redis-plus-plus

# hiredis安装

redis-plus-plus 基于 hiredis，因此应先安装 hiredis。hiredis 的最低版本要求是 v0.12.1。

1、包管理器安装 hiredis 服务

```shell
sudo apt install libhiredis-dev
```

2、源码编译

```shell
git clone https://github.com/redis/hiredis.git
cd hiredis
make
make install
```

默认情况下，hiredis 安装在 `/usr/local`。如果想将 hiredis 安装在非默认位置，请使用以下命令指定安装路径。

```shell
make PREFIX=/non/default/path

make PREFIX=/non/default/path install
```

# redis++ 客户端安装

```shell
git clone https://github.com/sewenew/redis-plus-plus.git

cd redis-plus-plus

mkdir build

cd build

cmake ..

make

make install

cd ..
```

如果 hiredis 安装在非默认位置，则应使用 CMAKE_PREFIX_PATH 指定 hiredis 的安装路径。默认情况下，redis-plus-plus 安装在 /usr/local。不过，您可以使用 CMAKE_INSTALL_PREFIX 将 redis-plus-plus 安装到非默认位置。

```shell
cmake -DCMAKE_PREFIX_PATH=/path/to/hiredis -DCMAKE_INSTALL_PREFIX=/path/to/install/redis-plus-plus ..
```

Note:

为了明确指定 c++ 标准，可以使用以下 cmake 标志：-DREDIS_PLUS_PLUS_CXX_STANDARD=11

在编译 redis-plus-plus 时，它还会编译测试程序，这可能需要一些时间。不过，你可以使用以下 cmake 选项关闭编译测试程序：
-DREDIS_PLUS_PLUS_BUILD_TEST=OFF。

默认情况下，redis-plus-plus 会同时构建静态库和共享库。如果只想构建其中一个，可以使用 -DREDIS_PLUS_PLUS_BUILD_STATIC=OFF 或 -DREDIS_PLUS_PLUS_BUILD_SHARED=OFF 禁用另一个。

redis-plus-plus 默认使用 -fPIC 选项（即位置独立代码）构建静态库。不过，你可以使用 -DREDIS_PLUS_PLUS_BUILD_STATIC_WITH_PIC=OFF 关闭它。

# 客户端编译链接

用静态库

```shell
g++ -std=c++17 -o app app.cpp /path/to/libredis++.a /path/to/libhiredis.a -pthread

# 指定路径的静态库
g++ -std=c++17 -I/non-default/install/include/path -o app app.cpp /path/to/libredis++.a /path/to/libhiredis.a -pthread
```

用动态库

```shell
g++ -std=c++17 -o app app.cpp -lredis++ -lhiredis -pthread

# 指定路径的动态库
g++ -std=c++17 -I/non-default/install/include/path -L/non-default/install/lib/path -o app app.cpp -lredis++ -lhiredis -pthread
```

# cmake编译客户端

如果使用 cmake 来构建应用程序，则需要在 CMakeLists.txt 中添加 hiredis 和 redis-plus-plus 依赖项：

```cmake
# <---------- set c++ standard ------------->
# NOTE: you must build redis-plus-plus and your application code with the same standard.
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# <------------ add hiredis dependency --------------->
find_path(HIREDIS_HEADER hiredis)
target_include_directories(target PUBLIC ${HIREDIS_HEADER})

find_library(HIREDIS_LIB hiredis)
target_link_libraries(target ${HIREDIS_LIB})

# <------------ add redis-plus-plus dependency -------------->
# NOTE: this should be *sw* NOT *redis++*
find_path(REDIS_PLUS_PLUS_HEADER sw)
target_include_directories(target PUBLIC ${REDIS_PLUS_PLUS_HEADER})

find_library(REDIS_PLUS_PLUS_LIB redis++)
target_link_libraries(target ${REDIS_PLUS_PLUS_LIB})
```

此外，如果在非默认位置安装了 hiredis 和 redis-plus-plus，则需要在运行 cmake 时使用 CMAKE_PREFIX_PATH 选项指定这两个库的安装路径。

```shell
cmake -DCMAKE_PREFIX_PATH=/installation/path/to/the/two/libs ..
```