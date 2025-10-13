## grep命令

`grep` 的强大之处在于它的各种选项，下面是最常用的一些：

| 选项           | 全称                   | 作用                                       | 示例                                                     |                     |
| :------------- | :--------------------- | :----------------------------------------- | :------------------------------------------------------- | ------------------- |
| `-i`           | `--ignore-case`        | **忽略大小写**                             | `grep -i "avocado" fruits.txt` 会找到 "Avocado"          |                     |
| `-v`           | `--invert-match`       | **反向选择**，打印**不匹配**的行           | `grep -v "apple" fruits.txt` 会打印所有不含 "apple" 的行 |                     |
| `-n`           | `--line-number`        | **显示行号**                               | `grep -n "grape" fruits.txt` 会输出 `4:grape`            |                     |
| `-c`           | `--count`              | **只统计匹配的行数**，不显示内容           | `grep -c "a" fruits.txt` 统计包含字母 'a' 的行数         |                     |
| `-r`           | `--recursive`          | **递归搜索**目录下的所有文件               | `grep -r "hello" /path/to/dir/`                          |                     |
| `-l`           | `--files-with-matches` | **只显示包含匹配项的文件名**               | `grep -l "apple" *.txt`                                  |                     |
| `-w`           | `--word-regexp`        | **全字匹配**，只匹配完整的单词             | `grep -w "apple"` 不会匹配到 "PINEAPPLE"                 |                     |
| `-A <n>`       | `--after-context=<n>`  | 显示匹配行及其**后 n 行**                  | `grep -A 2 "banana" fruits.txt`                          |                     |
| `-B <n>`       | `--before-context=<n>` | 显示匹配行及其**前 n 行**                  | `grep -B 1 "grape" fruits.txt`                           |                     |
| `-C <n>`       | `--context=<n>`        | 显示匹配行及其**前后各 n 行**              | `grep -C 1 "banana" fruits.txt`                          |                     |
| `-E`           | `--extended-regexp`    | 使用**扩展正则表达式**（功能更强）         | `grep -E "apple                                          | banana" fruits.txt` |
| `-F`           | `--fixed-strings`      | 将模式视为**固定字符串**，而不是正则表达式 | `grep -F "a.b" file` 会搜索字面值 "a.b"                  |                     |
| `--color=auto` |                        | **高亮显示**匹配到的文本                   | `grep --color=auto "apple" fruits.txt`                   |                     |

**提示**：大多数现代 Linux 系统已经为 `grep` 设置了默认别名 `alias grep='grep --color=auto'`，所以匹配结果会自动高亮。





## find命令

- 按文件名查找
  - -name 按**文件名**匹配（**区分大小写**）
  - -iname 按文件名匹配（**不区分大小写**）
- 按文件类型查找
  - f 普通文件
  - d 目录
  - l 符合连接
  - b 块设备文件
- 按时间查找
  - `-mtime <n>`	文件数据在 n * 24 小时前被修改过
  - `-mmin <n>`	文件数据在 n 分钟前被修改过
  - `-atime <n>`	文件被访问过的时间

根据文件大小进行查找。

| 大小参数 | 含义       |
| :------- | :--------- |
| `+n`     | 大于 n     |
| `-n`     | 小于 n     |
| `n`      | 正好等于 n |

**大小单位**:

- `c`: 字节 ( bytes )
- `k`: KiB ( 1024 bytes )
- `M`: MiB ( 1024 * 1024 bytes )
- `G`: GiB ( 1024 * 1024 * 1024 bytes )

对找到的文件执行操作 (-exec 和 -delete)



查看监听端口并显示对应的进程名/PID

netstat -tulnp 