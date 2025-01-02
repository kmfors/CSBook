CMAKE_INSTALL_PREFIX 环境变量为 CMAKE_INSTALL_PREFIX 变量指定一个自定义的默认值，以取代 CMake 本身指定的默认值。指定的值必须是一个目录的绝对路径。


CMake 提供了一个命令行签名，用于安装已生成的项目二进制树：

`cmake --install <dir> [<options>]`

```shell
# 在生成构建系统时指定的安装参数
cmake .. -DCMAKE_INSTALL_PREFIX=<dest_dir>
```

以下的命令是用于在完成构建之后的安装操作的

```shell
# 执行构建目录下的默认位置安装
cmake --install <build_dir>

# 执行构建目录下的指定位置安装
cmake --install <build_dir> --prefix <dest_dir>
```

---

https://cmake.org/cmake/help/git-stage/variable/CMAKE_INSTALL_PREFIX.html#variable:CMAKE_INSTALL_PREFIX

```shell
# 以下操作都基于在CMakeLists同级目录下进行使用#等价于cmake
alias cbd='cmake -B ./build -DCMAKE_BUILD_TYPE=Debug'
alias cbr='cmake -B ./build -DCMAKE_BUILD_TYPE=Release'

# 等价于make -j4
alias cuu='cmake --build ./build --parallel 4'

# 在构建完成之后的默认安装命令,等价于 make install
alias cib='cmake --install ./build'

# 在构建完成之后的指定目录安装命令, cibp <dest_path>
alias cibp='cmake --install ./build --prefix'

# 清空构建目录
alias crb='rm -rf ./build'

alias rma='rm -rf'

alias gg='cd ~/Repo/CodeFree/5_src'

```