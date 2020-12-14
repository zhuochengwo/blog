---
title: shell基础（四）之处理用户输入
tags:
  - Shell
  - shell基础系列
id: '340'
categories:
  - - Shell
  - - 敲代码
date: 2018-12-07 23:26:22
---

## 概述

**这篇文章是[Shell基础系列](https://chengfeng.site/tag/shell%E5%9F%BA%E7%A1%80/)文章的第四篇** 这一章主要讲bash shell的对于用户输入的处理，涉及输入参数、选项、交互式输入等内容

## 命令行参数

在第一章，学习变量的时候，我们有接触几个特殊变量，这些就是用户输出参数相关的变量。

变量名

描述

$0

是脚本名

$n

第n个参数，如$1 是第一个参数

$#

参数个数

${!#}

最后一个参数

$\*

获取当前shell的所有参数 “$1 $2 $3 …注意与$#的区别

$?

获取执行的上一个指令的返回值(0 为成功， 非零为失败)

$!

执行上一个指令的PID

$$

获取当前shell的进程号（PID）

$@

这个程序的所有参数 “$1″ “$2″ “$3″ “…”

示例

```bash
#!/bin/bash

echo $0
echo $#
echo ${!#}
echo $*
echo $@
for (( a=0;a<=$#;a++ ))
do
    echo "Argument $a is" `eval echo '$'"$a"` # 这里eval是为了以变量$a的值作为变量名来获取新的变量名的值，相当于echo $0 echo $1 ....
done
```

执行 `bash test.sh 0 1 2 3` 输出：

```vim
test.sh
4
3
0 1 2 3
0 1 2 3
Argument 0 is test.sh
Argument 1 is 0
Argument 2 is 1
Argument 3 is 2
Argument 4 is 3
```

通过上边的例子，看不出`$*`与`$@`的区别。`$*`会将所有参数当成单个参数，`$@`会单独处理每个参数，在for循环遍历的时候就能看出区别

```bash
count=1
for param in "$*"
do
    echo "\$* Parameter #$count = $param"
    count=$[ $count + 1 ]
done
echo "----------------------------------------"
count=1
for param in "$@"
do
    echo "\$@ Parameter #$count = $param"
    count=$[ $count + 1 ]
done
```

执行 `bash test.sh hello world 123` 输出

```vim
$* Parameter #1 = hello world 123
----------------------------------------
$@ Parameter #1 = hello
$@ Parameter #2 = world
$@ Parameter #3 = 123
```

在使用参数之前，要注意检查参数是否有赋值，不然可能会导致程序出错

```bash
if [ -n $1 ]
then
    echo Hello $1
else
    echo nothing
fi
```

## `shift`命令移动参数

`shift`命令能用来操作移动命令行参数，在不知道参数个数的时候，可以通过此命令移动命令行参数完成命令行参数的遍历

```bash
count=1
while [ -n "$1" ]
do
    echo "Parameter #$count = $1"
    count=$[ $count + 1 ]
    shift
done
```

执行 `./test.sh 1 2 3 4`输出

```vim
Parameter #1 = 1
Parameter #2 = 2
Parameter #3 = 3
Parameter #4 = 4
```

## 处理选项

在shell中，一般用`case`结合`shift`命令来处理选项，但这种方式太麻烦不够实用，这里不做过多介绍，只给一个简单示例

```bash
while [ -n $1 ]
do
    case "$1" in
        -a) echo "Found the -a option";;
        -b) param=$2 # -b 后边紧跟选项参数的情况
            echo "Found the -b option,with parameter value $param"
            shift;;
        -c) echo "Found the -c option";;
        --) shift   # -- 用来标示选项列表结束
            break;;
        *) echo "$1 is not an option";;
    esac
    shift
done
count=1
for param in $@
do
    echo "Paramter #$count: $param"
    count=$[ $count + 1 ]
done
```

执行 `./test.sh -a -b optionB -c -- test1 test2` 输出

```vim
Found the -a option
Found the -b option,with parameter value optionB
Found the -c option
Paramter #1:test1
Paramter #2:test2
```

### `getopts`命令

如果要完美的解析参数其实是很麻烦的，还好有完善的工具来帮我们，简单而又实用。 语法： `getopts optstring variable` 其中`optstring`定义命令行有效的选项字母,用`:`来区分带参数的选项。如`ab:cd`表示有`-a -b -c -d`四个选项，其中`-b`选项需要一个参数值 `:`还有另外一个作用就是屏蔽错误信息，在`optstring`前边加一个`:`就能屏蔽掉输入错误的选项带来的错误提示 `getopts`命令会将当前参数保存在命令行定义的`variable`方便遍历处理 `getopts`命令会用到两个环境变量`OPTARG`和`OPTIND`

*   `OPTARG` 保存选项的参数值
*   `OPTIND` 保存参数列表中`getopts`正在处理的参数位置

示例：

```bash
while getopts :ab:c: opt    # 第一个:用户屏蔽错误信息
do
    case $opt in
        a) echo "Found the -a option,OPTIND:$OPTIND";;
        b) echo "Found the -b option,with value $OPTARG,OPTIND:$OPTIND";;
        c) echo "Found the -c option,with value $OPTARG,OPTIND:$OPTIND";;
        *) echo "Unknown option:$opt,OPTIND:$OPTIND";;
    esac
done

shift $[ $OPTIND - 1 ]  # getopts处理完选项会停止，利用OPTIND和shift方便处理后边的参数
count=1
for param in $@
do
    echo "Paramter #$count: $param"
    count=$[ $count + 1 ]
done
```

执行`./test.sh -ab "test1 test2" -ctest3 -d arg1 arg2`输出

```vim
Found the -a option,OPTIND:1
Found the -b option,with value test1 test2,OPTIND:3
Found the -c option,with value test3,OPTIND:4
Unknown option:?,OPTIND:5
Paramter #1: arg1
Paramter #2: arg2
```

输出结果分析

*   注意`-b`选项的参数，是可以用`""`来包含空格的
*   注意`-c`选项的参数，是可以放在一起而不用空格的
*   注意`?`，未定义选项统一用`?`发给代码
*   利用`shift`和`$OPTIND`移动参数，这样`$@`中就是单纯的命令行参数，没有选项信息了

### 选项标准化

在`Linux`世界中有些字母选项已经有了某种标准含义，我们在写`shell`脚本中能遵循这个“标准”会更友好

选项

描述

\-a

显示所有对象

\-c

生成一个计数

\-d

指定一个目录

\-e

扩展一个对象

\-f

指定读入数据的文件

\-h

显示命令帮助信息

\-i

忽略大小写

\-l

产生输出的长格式版本

\-n

使用非交互模式

\-o

将所有输出重定向到指定的输出文件

\-q

以安静模式运行

\-r

递归地处理目录和文件

\-s

以安静模式运行

\-v

详细输出

\-x

排除某个对象

\-y

对所有问题回答yes

## 获得用户输入

命令行选项和参数是从用户处获得输出的一种重要方式，但如果想要在脚本运行时交互式的获得用户还需要其它方式来实现

### `read`命令

`read`命令从标准输入或另外一个文件描述符中接受输入，将接收的数据放入一个变量 示例

```bash
echo -n "Enter your name:"
read name
read -p "Enter your age and birthday:" age birthday
read -p "Enter more information about you:"
echo "Get your name:$name, age:$age, birthday:$birthday and more: $REPLY"
```

执行后输出

```vim
./test.sh
Enter your name:Chengfeng
Enter your age and birthday:18 2018-01-01
Enter more information about you:male China
Get your name:Chengfeng, age:18, birthday:2018-01-01 and more: male China
```

分析：

*   echo -n\` 能让输出不换行
*   `read -p`能直接输出，可以替代`echo`
*   `read`后可以指定多个变量用于接收参数，如上例中的`age birthday`，如果输入的参数超过接收的变量个数，则最后一个变量将保存剩下所有输入的数据
*   `read`如果不指定变量则`REPLY`环境变量会保存输入的所有数据

#### 输入超时问题

`read`命令会一直等待用户输入，如果用户一直不输入，我们又需要程序在等待一段时间继续往下执行，我们就需要用到`read -t`

```bash
if read -t 5 -p "Please enter your name: " name
then
    echo "Hello $name, welcome to my script"
else
    echo
    echo "Sorry, too slow! "
fi 
```

### 隐藏输入

在输入密码等私密类信息是，不希望在屏幕上展示输入内容，可以用`read -s`

```bash
read -s -p "Enter your password: " pass 
echo
echo "Is your password really $pass? " 
```