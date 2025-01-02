# 1. shell 脚本基础

在创建 shell 脚本文件时，必须在文件的第一行指定要使用的 shell，格式如下：

```shell
#!/bin/bash
```

`#!`：会告诉 shell 用哪个 shell 来运行脚本。

shell 运行脚本的条件：

1. 脚本具备执行权限
2. shell 要找得到脚本文件位置，可以通过环境变量或路径指明

## 1.1 打印消息

使用 echo 命令打印字符串消息。

把字符串和命令输出显示在同一行中，可以使用 echo 命令的 -n 选项。（取消当前行的换行）

```shell
echo -n "The time and date are: "
date
```

效果如下：

```shell
$ ./test1 
The time and date are: Mon Jun 01 15:38:19 EST 2020
```

## 1.2 使用变量

### 1.2.1 环境变量

在脚本中，可以在环境变量名之前加上 $ 来引用这些环境变量。注意，echo 命令中的环境变量会在脚本运行时被替换成当前值。

### 1.2.2 用户自定义变量

除了环境变量，shell 脚本还允许用户在脚本中定义和使用自己的变量。定义变量允许在脚本中临时存储并使用数据，从而使 shell 脚本看起来更像一个真正的计算机程序。

用户自定义变量的名称可以是由字母、数字或下划线组成的字符串，长度不能超过 20 个字符且区分字母大小写。

在使用等号为变量赋值时，要注意在变量、等号和值之间是不能出现空格的。

shell 脚本会以字符串形式存储所有的变量值，脚本中的各个命令可以自行决定变量值的数据类型。shell 脚本中定义的变量在脚本的整个生命周期里会一直保持着它们的值，在脚本结束时会被删除。并且每次引用变量时，都会输出该变量的当前值。

> [!NOTE]
>
> 引用变量值时要加 `$` ，对变量赋值时则不用加 `$` 。

### 1.2.3 命令替换

shell 脚本中最有用的特性之一是可以从命令输出中提取信息并将其赋给变量。有两种方法可以将命令输出赋给变量：

```shell
# 1. 使用反引号
testing=`date`

# 2. 使用 $() 格式,推荐
testing=$(date)
```

以上两种，shell 会执行命令替换符内的命令，将其输出赋给变量 testing。过 ${variable}形式引用的变量。花括号通常用于帮助界定$ 后的变量名

例子 1：

```shell
today=$(date +%y%m%d)
```

today 变量保存着格式化后的 date 命令的输出。+%y%m%d 格式会告诉 date 命令将日期显示为两位数的年、月、日的组合。

## 1.3 重定向

### 1.3.1 输出重定向

bash shell 使用大于号（>）来实现将命令的输出重定向发送至文件的操作：`command > outputfile` 。随后出现在显示器上的命令输出就会被保存到指定的输出文件中。

- 如果输出文件已存在，则重定向运算符会用新数据覆盖已有的文件。
- 如果想要在原文件里追加而不是覆盖，可以用双大于号（>>）来追加数据。

### 1.3.2 输入重定向

输入重定向会将文件的内容重定向至命令：`command < intputfile` 。

wc 命令可以统计数据中的文本。在默认情况下，它会输出 3 个值。

- 文本的行数
- 文本的单词数
- 文本的字节数

```shell
$ wc < test6 
 2 11 60
```

内联输入重定向：无须使用文件进行重定向，只需在命令行中指定用于输入重定向的数据即可。注意哈，内联输入重定向运算符是双小于号（<<）。除了这个符号，必须指定一个文本标记来划分输入数据的起止。任何字符串都可以作为文本标记，但在数据开始和结尾的文本标记必须一致：

```shell
command << marker 
data 
marker
```

在命令行中使用内联输入重定向时，shell 会用 PS2 环境变量中定义的次提示符来提示输入数据，其用法如下所示：

```shell
$ wc << EOF 
> test string 1 
> test string 2 
> test string 3 
> EOF 
 3 9 42
```

次提示符会持续显示，以获取更多的输入数据，直到输入了作为文本标记的那个字符串。

## 1.4 管道

管道被置于命令之间，将一个命令的输出作为另一个命令的输入：`command1 | command2` 

实际上，Linux 系统会同时运行这两个命令，在系统内部将二者连接起来。当第一个命令产生输出时，它会被立即传给第二个命令。数据传输不会用到任何中间文件或缓冲区。
现在，你可以利用管道轻松地将 rpm 命令的输出传

## 1.5 数学运算

在 bash 中，要将数学运算结果赋给变量，可以使用 `$` 和方括号`（$[ operation ]）`

例如：不过 bash shell 的数学运算符只支持整数运算。

```shell
var1=100
var2=50 
var3=45 
var4=$[$var1 * ($var2 - $var3)] 
echo The final result is $var4
```


## 1.6 退出脚本

shell 中运行的每个命令都使用退出状态码来告诉 shell 自己已经运行完毕。退出状态码是一个 0～255 的整数值，在命令结束运行时由其传给 shell。你可以获取这个值并在脚本中使用。

### 1.6.1 查看退出状态码

Linux 提供了专门的变量 `$?` 来保存最后一个已执行命令的退出状态码。对于需要进行检查的命令，必须在其运行完毕后立刻查看或使用 `$?` 变量。这是因为该变量的值会随时变成由 shell 所执行的最后一个命令的退出状态码：

```shell
$ date 
Mon Jun 01 16:01:30 EDT 2020 
$ echo $? 
0
```

按照惯例，对于成功结束的命令，其退出状态码是 0。对于因错误而结束的命令，其退出状态码是一个正整数。

```shell
$ asdfg 
-bash: asdfg: command not found
$ echo $? 
127
```

![[20240904202547.png]]


### 1.6.2 exit 命令

在默认情况下，shell 脚本会以脚本中的最后一个命令的退出状态码退出：

改变这种默认行为，若要返回自己的退出状态码，exit 命令是可以允许在脚本结束时指定一个退出状态码的。

```shell
#!/bin/bash 
# testing the exit status 
var1=10 
var2=30 
var3=$[ $var1 + $var2 ] 
echo The answer is $var3 
exit 5
```

当你检查脚本的退出状态码时，就会看到传给 exit 命令的参数值：

```shell
$ ./test13 
The answer is 40 
$ echo $? 
5
```

退出状态码被缩减到了 0～255 的区间。shell 通过模运算得到这个结果。一个值的模就是被除后的余数。



# 2. 结构化命令

## 2.1 if-then 语句

最基本的结构化命令是 if-then 语句。if-then 语句的格式如下：

```shell
if command
then 
    commands
fi
```

bash shell 的 if 语句会运行 if 之后的命令。如果该命令的退出状态码为 0（命令成功运行），那么位于 then 部分的命令就会被执行。如果该命令的退出状态码是其他值，则 then 部分的命令不会被执行，bash shell 会接着处理脚本中的下一条命令。

能出现在 then 部分的命令可以是多条的，bash shell 会将这些命令视为一个代码块。如果 if 语句行命令的退出状态值为 0，那么代码块中所有的命令都会被执行；否则，会跳过整个代码块。


## 2.2 if-then-else 语句

if-then-else 语句的格式如下：

```shell
if command
then 
    commands
else 
    commands
fi
```

当 if 语句中的命令返回退出状态码 0 时，then 部分中的命令会被执行，这跟普通的 if-then语句一样。当 if 语句中的命令返回非 0 退出状态码时，bash shell 会执行 else 部分中的命令。

## 2.3 嵌套 if 语句

有时需要在脚本中检查多种条件。对此，可以使用嵌套的 if-then 语句。但在脚本中使用这种嵌套 if-then 语句的问题在于代码不易阅读，逻辑流程很难厘清。因此可以使用 else 部分的另一种名为 elif 的形式，这样就不用再写多个 if-then 语句了。

elif 使用另一个 if-then 语句延续 else 部分：

```shell

if command1
then 
    command set 1 
elif command2 
then 
    command set 2
elif command3 
then 
    command set 3 
elif command4 
then 
    command set 4 
fi
```

例如：效果等同于编程语言中的 if-elif-else

```shell
#!/bin/bash
# testing nested ifs - using elif
#
testuser=snow
#
if grep $testuser /etc/passwd
then
    echo "The user $testuser account exists on this system."
    echo
elif ls -d /home/$testuser/
then
    echo "The user $testuser has a directory,"
    echo "even though $testuser doesn't have an account."
else
    echo "The user $testuser does not exist on this system,"
    echo "and no directory exists for the $testuser."
fi
echo "We are outside the nested if statements."
```

每个代码块会根据命令是否会返回退出状态码 0 来执行。记住，bash shell 会依次执行 if 语句，只有第一个返回退出状态码 0 的语句中的 then 部分会被执行。
尽管使用了 elif 语句的代码看起来更清晰，但是脚本的逻辑仍然会让人犯晕。


## 2.4 test 命令

由于 if-then 语句只能判断命令退出状态码的条件，因此需要一种命令能够测试命令退出状态码之外的条件。

test 命令可以在 if-then 语句中测试不同的条件。如果 test 命令中列出的条件成立，那么 test 命令就会退出并返回退出状态码 0。这样 if-then 语句的工作方式就和其他编程语言中的
if-then 语句差不多了。如果条件不成立，那么 test 命令就会退出并返回非 0 的退出状态码，这使得 if-then 语句不会再被执行。

当用在 if-then 语句中时，test 命令看起来如下所示：

```shell
if test condition
then 
    commands
fi
```

如果不写 test 命令的 condition 部分，则它会以非 0 的退出状态码退出并执行 else 代码块语句。

例子：由于变量 my_variable 中包含内容（Full），因此当 test 命令测试条件时，返回的退出状态码为 0。这使得 then 语句块中的语句得以执行。

```shell
#!/bin/bash
# testing if a variable has content
#
my_variable="Full"
#
if test $my_variable
then
    echo "The my_variable variable has content and returns a True."
    echo "The my_variable variable content is: $my_variable"
else
    echo "The my_variable variable doesn't have content,"
    echo "and returns a False."
fi
```

如果该变量中没有包含内容，就会出现相反的情况。

bash shell 提供了另一种条件测试方式，无须在 if-then 语句中写明 test 命令：

```shell
if [ condition ] 
then 
    commands
fi
```

方括号定义了测试条件。注意，第一个方括号之后和第二个方括号之前必须留有空格，否则就会报错。

test 命令和测试条件可以判断 3 类条件：

- 数值比较
- 字符串比较
- 文件比较

### 2.4.1 数值比较

### 2.4.2 字符串比较

### 2.4.3 文件比较

## 2.5 复合条件测试

if-then 语句允许使用布尔逻辑将测试条件组合起来。可以使用以下两种布尔运算符：

```shell
[ condition1 ] && [ condition2 ]
[ condition1 ] || [ condition2 ]
```

## 2.6 if-then 的高级特性

bash shell 还提供了 3 个可在 if-then 语句中使用的高级特性：

- 在子 shell 中执行命令的单括号
- 用于数学表达式的双括号
- 用于高级字符串处理功能的双方括号

### 2.6.1 使用单括号

单括号允许在 if 语句中使用子 shell。单括号形式的 test 命令格式如下：

```shell
(command)
```
 
在 bash shell 执行 command 之前，会先创建一个子 shell，然后在其中执行命令。如果命令成功结束，则退出状态码会被设为 0，then 部分的命令就会被执行。如果命令的退出状态码不为 0，则不执行 then 部分的命令。来看一个使用子 shell 进行测试的例子：

```shell
#!/bin/bash                                                                                                                                        # Testing a single parentheses condition
#
echo $BASH_SUBSHELL
#
if (echo $BASH_SUBSHELL)
then
    echo "The subshell command operated successfully."
    #
else
    echo "The subshell command was NOT successful."
    #
fi
```

当脚本第一次（在 if 语句之前）执行 `echo $BASH_SUBSHELL` 命令时，是在当前 shell 中完成的。该命令会输出 0，表明没有使用子 shell。在 if 语句内，脚本在子 shell 中执行 `echo $BASH_SUBSHELL` 命令，该命令会输出 1，表明使用了子 shell。子 shell 操作成功结束，接下来是执行 then 部分的命令。

### 2.6.2 使用双括号

双括号命令允许在比较过程中使用高级数学表达式。test 命令在进行比较的时候只能使用简单的算术操作。双括号命令提供了更多的数学符号，这些符号对有过其他编程语言经验的程序员而言并不陌生。双括号命令的格式如下：

```shell
(( expression ))
```

expression 可以是任意的数学赋值或比较表达式。

![[20240905110936.png|625]]

例子：注意，双括号中表达式的大于号不用转义。这是双括号命令又一个优越性的体现。

```shell
#!/bin/bash 
# Testing a double parentheses command 
# 
val1=10 
# 
if (( $val1 ** 2 > 90 )) 
then 
 (( val2 = $val1 ** 2 )) 
 echo "The square of $val1 is $val2," 
 echo "which is greater than 90." 
# 
fi
```

### 2.6.3 使用双方括号

双方括号命令提供了针对字符串比较的高级特性。双方括号的格式如下：

```shell
[[ expression ]]
```

expression 可以使用 test 命令中的标准字符串比较。除此之外，它还提供了 test 命令所不具备的另一个特性 —— **模式匹配**。

在进行模式匹配时，可以定义通配符或正则表达式来匹配字符串：

```shell
#!/bin/bash
# Using double brackets for pattern matching
#
#
if [[ $BASH_VERSION == 5.* ]]
then
    echo "You are using the Bash Shell version 5 series."
fi
```

上述脚本中使用了双等号`（==）`。双等号会将右侧的字符串`（5.*）`视为一个模式并应用模式匹配规则。双方括号命令会对$BASH_VERSION 环境变量进行匹配，看是否以字符串 5.起始。如果是，则测试通过，shell 会执行 then 部分的命令。

## 2.7 case 命令

case 命令会采用列表格式来检查变量的多个值：

```shell
case variable in 
pattern1 | pattern2) commands1;; 
pattern3) commands2;; 
*) default commands;; 
esac
```

case 命令会将指定变量与不同模式进行比较。如果变量与模式匹配，那么 shell 就会执行为该模式指定的命令。你可以通过竖线运算符在一行中分隔出多个模式。星号会捕获所有与已知模
式不匹配的值。下面是一个将 if-then-else 程序转换成使用 case 命令的例子：

```shell
#!/bin/bash
# Using a short case statement
#
case $USER in
    rich | christine)
        echo "Welcome $USER"
        echo "Please enjoy your visit.";;
    barbara | tim)
        echo "Hi there, $USER"
        echo "We're glad you could join us.";;
    testing)
        echo "Please log out when done with test.";;
    *)
        echo "Sorry, you are not allowed here."
esac
```


补充：

```shell
if command -v dpkg &> /dev/null
```

command 是一个 shell 内置命令，用于在系统路径中查找指定的可执行文件。-v 选项表示显示找到的可执行文件的完整路径。在这里，它试图查找名为 "dpkg" 的命令。

`&>` ：这是一个重定向操作符，将命令的标准输出和标准错误输出都重定向到 `/dev/null`。

`/dev/null` 是一个特殊的设备文件，它会丢弃所有写入其中的数据。因此，这个操作的目的是屏蔽掉命令的输出，只关心命令是否成功执行。

综上所述，这段代码的作用是检查系统中是否存在 "dpkg" 命令。如果存在，那么 if 语句的条件为真，可以继续执行后续的操作；如果不存在，那么 if 语句的条件为假，不会执行后续的操作。

在命令行中，> 和 &> 都是用于重定向输出的操作符，但它们之间有一些区别：

`>` 是一个基本的重定向操作符，用于将标准输出（stdout）重定向到指定的文件或设备。如果使用 `>` 重定向，只有标准输出会被重定向，而标准错误输出（stderr）仍然会显示在终端上。

`&>` 是一个组合的重定向操作符，用于将标准输出和标准错误输出都重定向到指定的文件或设备。使用 &> 时，标准输出和标准错误输出都会被重定向，不会显示在终端上。


# 3. 结构化命令进阶

## 3.1 for 命令

bash shell 提供了 for 命令，以允许创建遍历一系列值的循环。每次迭代都使用其中一个值来执行已定义好的一组命令。for 命令的基本格式如下：

```shell
for var in list
do 
    commands
done
```

### 3.1.1 读取列表中的值

for 命令最基本的用法是遍历其自身所定义的一系列值：

```shell
for test in Alabama Alaska Arizona Arkansas California Colorado 
do 
    echo The next state is $test 
done
```


### 3.1.2 读取列表中的复杂值

记住，for 循环假定各个值之间是以空格分隔的，另外要注意的是，当你使用双引号引用某个值时，shell 并不会将双引号当成值的一部分。

### 3.1.3 从变量中读取值列表

```shell
#!/bin/bash
# using a variable to hold the list 
list="Alabama Alaska Arizona Arkansas Colorado" 
list=$list" Connecticut"
for state in $list 
do 
    echo "Have you ever visited $state?" 
done
```

注意，脚本中还使用了另一个赋值语句向 `$list` 变量包含的值列表中追加（或者说是拼接）了一项。这是向变量中已有的字符串尾部添加文本的一种常用方法。

### 3.1.4 从命令中读取值列表

```shell
#!/bin/bash 
# reading values from a file 
file="states.txt" 
for state in $(cat $file) 
do 
    echo "Visit beautiful $state" 
done
```

注意，states.txt 文件中每个值各占一行，而不是以空格分隔。for 命令仍然以每次一行的方式遍历 cat 命令的输出。但这并没有解决数据中含有空格的问题。如果你列出了一个名字中有空格的州，则 for 命令仍然
会用空格来分隔值。

### 3.1.5 更改字段分隔符

造成上面问题的原因是特殊的环境变量 IFS（internal field separator，内部字段分隔符）。IFS 环境变量定义了 bash shell 用作字段分隔符的一系列字符。在默认情况下，bash shell 会将下列字
符视为字段分隔符。

- 空格
- 制表符
- 换行符

如果 bash shell 在数据中看到了这些字符中的任意一个，那么它就会认为这是列表中的一个新字段的开始。在处理可能含有空格的数据（比如文件名）时，这就很烦人了，就像你在先前脚本示例中看到的那样。

解决这个问题的办法是在 shell 脚本中临时更改 IFS 环境变量的值来限制被 bash shell 视为字段分隔符的字符。如果想修改 IFS 的值，使其只能识别换行符，可以这么做：

```shell
IFS=$'\n'
```

将该语句加入脚本，告诉 bash shell 忽略数据中的空格和制表符。对前一个脚本使用这种方法，将获得如下输出：

```shell
#!/bin/bash 
# reading values from a file 
file="states.txt" 
IFS=$'\n' 
for state in $(cat $file) 
do 
    echo "Visit beautiful $state" 
done
```

```shell
# 现在 shell 脚本能够识别出列表中含有空格的值了。
Visit beautiful Colorado 
Visit beautiful Connecticut 
Visit beautiful Delaware 
Visit beautiful Florida 
Visit beautiful Georgia 
Visit beautiful New York 
Visit beautiful New Hampshire 
Visit beautiful North Carolina
```

还有其他一些 IFS 环境变量的绝妙用法。如果要遍历文件中以冒号分隔的值（比如/etc/passwd文件），则只需将 IFS 的值设为冒号即可：

```shell
IFS=:
```

如果要指定多个 IFS 字符，则只需在赋值语句中将这些字符写在一起即可：

```shell
IFS=$'\n:;"'
```

该语句会将换行符、冒号、分号和双引号作为字段分隔符。如何使用 IFS 字符解析数据没有任何限制。


### 3.1.6 使用通配符读取目录

最后，还可以用 for 命令来自动遍历目录中的文件。为此，必须在文件名或路径名中使用通配符，这会强制 shell 使用文件名通配符匹配（file globbing）。文件名通配符匹配是生成与指定通配符匹配的文件名或路径名的过程。

在处理目录中的文件时，如果不知道所有的文件名，上述特性是非常好用的：

```shell
#!/bin/bash
# iterate through all the files in a directory
for file in /home/snow/*
do
    if [ -d "$file" ]
    then
        echo "$file is a directory"
    elif [ -f "$file" ]
    then
        echo "$file is a file"
    fi
done
```

在 Linux 中，目录名和文件名中包含空格是完全合法的。要应对这种情况，应该将 $file 变量放入双引号内。否则，遇到含有空格的目录名或文件名时会产生错误：