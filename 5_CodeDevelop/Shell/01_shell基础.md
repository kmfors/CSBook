# 基本命令

## `pwd` – 显示当前目录路径

```bash
pwd
```

## `cd` – 切换目录

```bash
cd /var/log      # 进入指定目录
cd ..            # 返回上一级
cd ~             # 回主目录
cd -             # 回上次目录
```

## `ls` – 列出目录内容

```bash
ls               # 列出文件名
ls -l            # 详细列表（权限、大小、时间）
ls -a            # 包含隐藏文件
ls -lh           # 人类可读的大小
```

## `mkdir` – 创建目录

```bash
mkdir myfolder          # 创建目录
mkdir -p a/b/c          # 创建多级目录（父目录不存在时自动创建）
```

## `rm` – 删除

```bash
rm file.txt             # 删除文件
rm -r myfolder/         # 删除目录（含内容）
rm -rf myfolder/        # 强制删除，不询问（⚠️ 危险！删前 pwd 确认）
```

## `cp` – 复制

```bash
cp source.txt dest.txt   # 复制文件
cp -r sourcedir/ destdir/ # 复制目录
```

## `mv` – 移动/重命名

```bash
mv old.txt new.txt       # 重命名
mv file.txt /tmp/        # 移动
```

## `cat` – 显示文件内容

```bash
cat file.txt             # 显示整个文件
cat -n file.txt          # 显示行号
```

## `less` – 分页查看（推荐）

```bash
less large.log           # 空格下翻，b上翻，/搜索，q退出
```

## `head` / `tail` – 查看文件头尾

```bash
head -n 20 file.txt      # 前20行（默认10）
tail -n 20 file.txt      # 后20行
tail -f app.log          # 实时追踪日志（常用）
```

---

# 常用命令

| 命令    | 作用                  | 示例                                |
| ------- | --------------------- | ----------------------------------- |
| `touch` | 创建空文件/更新时间戳 | `touch file.txt`                    |
| `echo`  | 输出字符串            | `echo "Hello"`                      |
| `which` | 查找命令位置          | `which python`                      |
| `find`  | 搜索文件              | `find /home -name "*.txt"`          |
| `grep`  | 搜索文本              | `grep "error" log.txt`              |
| `wc`    | 统计行数/单词数       | `wc -l file.txt`                    |
| `sort`  | 排序                  | `sort names.txt`                    |
| `uniq`  | 去重（先排序）        | `sort file \| uniq -c`              |
| `tar`   | 打包/压缩             | `tar -czf archive.tar.gz mydir/`    |
| `chmod` | 改权限                | `chmod +x script.sh`                |
| `ps`    | 查看进程              | `ps aux`                            |
| `kill`  | 终止进程              | `kill 1234`，`kill -9 1234`（强制） |

---



# 文件权限

## `chmod` – 修改权限

**符号模式**（好记）：
```bash
chmod u+x script.sh    # 所有者加执行权限
chmod g-w file.txt     # 移除组的写权限
chmod -R g+r folder/   # 递归给组加读权限
```

**数字模式**（快速）：
```bash
chmod 755 script.sh    # rwxr-xr-x（脚本常用）
chmod 600 secret.txt   # rw-------（敏感文件）
chmod 644 config.conf  # rw-r--r--（配置文件）
```

> 数字计算：r=4, w=2, x=1，求和

## `chown` – 修改所有者

```bash
sudo chown alice file           # 改所有者
sudo chown :developers file     # 改组
sudo chown -R alice:dev folder/ # 递归改所有者和组
```

## `umask` – 默认权限掩码

决定了新文件/目录的默认权限。  

计算公式：文件 = `666 - umask`，目录 = `777 - umask`

```bash
umask        # 查看当前值（通常是 0022）
umask 0002   # 新文件变成 664，新目录 775（团队协作）
umask 0077   # 新文件 600，新目录 700（仅自己）
```



# 文本处理

## `echo` – 输出文本

```bash
echo "Hello World"           # 输出字符串
echo $PATH                   # 输出变量值
echo -e "line1\nline2"       # 启用转义符（\n换行，\t制表符）
echo "text" > file.txt       # 写入文件（覆盖）
echo "more" >> file.txt      # 追加到文件
```



------

## `grep` – 搜索文本（最常用）

```bash
grep "error" log.txt         # 在文件中搜索
grep -i "error" log.txt      # 忽略大小写
grep -r "TODO" ./src/        # 递归搜索整个目录
grep -n "error" log.txt      # 显示行号
grep -v "debug" log.txt      # 反向匹配（显示不含 debug 的行）

# 常用组合
tail -f app.log | grep ERROR  # 实时日志过滤
ps aux | grep python          # 找 Python 进程
```



## `sed` – 流编辑器（替换/增删）

最核心：替换文本

```bash
# 输出替换，不改变原文件
sed 's/old/new/' file.txt      # 替换每行第一个 old
sed 's/old/new/g' file.txt     # 替换整行所有 old（g=全局）
sed 's/old/new/2' file.txt     # 替换每行第二个 old

# 直接修改文件（加 -i）
sed -i 's/old/new/g' file.txt  # 直接改原文件
sed -i.bak 's/old/new/g' file.txt  # 先备份成 file.txt.bak

# 指定行替换
sed '3,5s/old/new/g' file.txt  # 只替换第3-5行
```

其他实用操作：

```bash
sed '/pattern/d' file.txt      # 删除匹配的行
sed '3d' file.txt              # 删除第3行
sed -n '5,10p' file.txt        # 只打印第5-10行（-n 抑制默认输出）
```



## `awk` – 文本处理（按列操作）

awk 按行处理，每行自动按分隔符拆分成列，然后可以对列进行操作

```bash
# 基础语法：awk '{动作}' 文件

awk '{print $1}' file.txt      # 打印第1列
awk '{print $1, $3}' file.txt  # 打印第1和第3列
awk '{print $NF}' file.txt     # 打印最后一列（NF=列数）
awk '{print NR, $0}' file.txt  # 显示行号（NR=行号，$0=整行）
```

指定分隔符：

```bash
awk -F',' '{print $1}' data.csv   # 用逗号作为分隔符
awk -F':' '{print $1}' /etc/passwd # 处理 passwd 文件
```

条件过滤：

```
awk '/error/ {print $0}' log.txt     # 只处理包含 error 的行
awk '$3 > 100 {print $1, $3}' data.txt  # 第3列大于100才打印
awk 'NR>=10 && NR<=20' file.txt      # 打印第10-20行
```

常用内置变量：

| 变量      | 含义                           |
| :-------- | :----------------------------- |
| `$0`      | 整行                           |
| `$1`~`$n` | 第1~n列                        |
| `NF`      | 当前行的列数（`$NF`=最后一列） |
| `NR`      | 当前行号                       |
| `FS`      | 字段分隔符（可用 `-F` 设置）   |

组合使用（真正的生产力）：

```bash
# 提取错误日志的第1列（时间戳）并去重
grep "ERROR" app.log | awk '{print $1}' | sort -u

# 把配置文件中的 port 从 8080 改成 9090
sed -i 's/port=8080/port=9090/g' config.ini

# 找出内存超过 80% 的进程
ps aux | awk '$4 > 80 {print $2, $11}'  # $4=内存%，$11=进程名
```

