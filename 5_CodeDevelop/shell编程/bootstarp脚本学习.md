```shell
# $0 为当前脚本的文件名，PGR可以理解为Program Name的缩写
PRG="$0"

# 循环判断所指向的文件是否为软连接文件，是则将PRG指向该软连接所指向的文件，否则退出循环
while [ -h "$PRG" ] ; do
    # 获取软链接所指向的文件的详细信息
    ls=$(ls -ld "$PRG")
    # 提取软链接所指向的文件的路径， .* 代表
    link=$(expr "$ls" : '.*-> \(.*\)$')
    # 判断软链接所指向的文件是否为以 / 开头的绝对路径
    if expr "$link" : '/.*' > /dev/null; then
        PRG="$link"
    else
        PRG=$(dirname "$PRG")"/$link"
    fi
done
```

循环判断脚本文件是否为软链接文件，是则并获取其路径，否则退出循环。

```shell
PRG="$0"

while [ -h "$PRG" ] ; do
    ls=$(ls -ld "$PRG")
    link=$(expr "$ls" : '.*-> \(.*\)$')
    if expr "$link" : '/.*' > /dev/null; then
        PRG="$link"
    else
        PRG=$(dirname "$PRG")"/$link"
    fi
done
```

`while [ -h "$PRG" ] ; do`：循环判断所指向的是否为软链接文件

`link=$(expr "$ls" : '.*-> \(.*\)$')`：圆括号捕获 .* 所链接的 .* 作为一个表达式 B，来模式匹配表达式 `$ls` 为 A 。如果 A 能在 B 中匹配到，则返回表达式 B 的内容, 否则为空。将整体用 `$()` 方式命令替换为 link。在表达式 B，` $ ` 用于匹配输入字符串的结尾。