# 安装MSYS2 MINGW64工具链

```shell
pacman -S --needed base-devel mingw-w64-x86_64-toolchain
```

- `--needed` ：只安装需要的软件包。
- `base-devel`：软件包名，包含了很多基础开发工具的组件，比如gcc、make、git等。
- `mingw-w64-x86_64-toolchain`： 是一个MinGW-w64工具链，它可以用来编译Windows程序。

更新软件源列表：`sudo pacman -Syy` 来同步软件包数据库。

# 安装cmake 和make
一定要注意安装的先后顺序，有讲究的，但是忘记在哪里看到了。

安装**make组件**：

```shell
pacman -S mingw-w64-x86_64-make
```

安装**cmake组件**

```shell
pacman -S mingw-w64-x86_64-cmake
```

添加到环境变量中
```text
C:\msys64\mingw64\bin
C:\msys64\usr\bin
```

默认的编译选项（带星号） 和可选的编译选项

```
cmake -G -h

cmake -G "Unix Makefiles" ..
```

