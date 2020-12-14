---
title: shell基础（一）之构建基础脚本
tags:
  - Shell
  - shell基础系列
id: '321'
categories:
  - - Shell
  - - 敲代码
date: 2018-11-11 19:06:08
---

# 概述

**这篇文章是Shell基础系列文章的开篇，基于《Linux命令行与shell脚本编程大全》的学习笔记，笔记跳过了Linux基础部分，只关心shell的基本语法，在原书的基础上进行了相关补充。** **阅读本系列文章需要已有基本Linux基础** 本篇主要讲述脚本的创建，退出，变量，数学运算的相关知识。

## 开始编写脚本

创建脚本的时候，必须在文件第一行指定要使用的shell，`#!/bin/bash`

```bash
#!/bin/bash
# This script displays the date and who's logged on
echo "hello world"
```

运行脚本可以使用 `./name.sh` 或者 `bash name.sh` 其中的`name.sh` 表示脚本文件名，使用`./`的形式需要先给脚本执行权限`chmod u+x name.sh` 脚本使用`#`作为注释标记，多行注释使用：

```bash
:<<EOF
注释内容...
注释内容...
EOF
```

打印输出可以通过`echo`命令，如上边脚本的 `echo "hello world"`

## 使用变量

定义变量时，变量名不加美元符号，变量名和等号之间不能有空格，变量名的命名须遵循如下规则：

*   命名只能使用英文字母，数字和下划线，首个字符不能以数字开头。
*   中间不能有空格，可以使用下划线（\_）。
*   不能使用标点符号。
*   不能使用bash里的关键字（可用help命令查看保留关键字）。

可以在脚本中使用`环境变量`和`用户变量（局部变量`)，使用变量的时候需要加`$`符

```bash
#!/bin/bash
value="Welcome"   #定义用户变量，注意=左右不能有空格
echo "hello," $USER  #使用环境变量
echo $value #使用用户变量，必须使用$
```

对于变量的使用，可以用`$var`的形式，也可以用`${var}`，上述脚本`echo $value`等同于`echo ${value}`，`{}`仅作为变量边境，当涉及字符串拼接的时候，如`echo $valuexxx`应当改成`${value}xxx`

### 只读变量

使用`readonly`命令可以将变量定义为只读变量，只读变量的值不能被改变。

```bash
#!/bin/bash

val="hello"
readonly val
val="world"
```

执行上述命令输出错误

```vim
test.sh: line 5: val: readonly variable
```

### 删除变量

使用unset删除变量

```bash
unset variable_name
```

### 变量类型

#### 字符串

字符串可以用单引号，也可以用双引号，也可以不用引号。单双引号的区别跟PHP类似。 单引号字符串的限制：

*   单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的；
*   单引号字串中不能出现单独一个的单引号（对单引号使用转义符后也不行），但可成对出现，作为字符串拼接使用。

双引号的优点：

*   双引号里可以有变量
    
*   双引号里可以出现转义字符
    
    ```bash
    your_name='runoob'
    str="Hello, I know you are \"$your_name\"! \n"
    echo $str
    Hello, I know you are "runoob"! 
    ```
    

##### 拼接字符串

```bash
your_name="runoob"
# 使用双引号拼接
greeting="hello, "$your_name" !"
greeting_1="hello, ${your_name} !"
echo $greeting  $greeting_1
# 使用单引号拼接
greeting_2='hello, '$your_name' !'
greeting_3='hello, ${your_name} !'
echo $greeting_2  $greeting_3
```

##### 获取字符串长度

```bash
string="abcd"
echo ${#string} #输出 4
```

##### 提取子字符串

以下实例从字符串第 **2** 个字符开始截取 **4** 个字符：

```bash
string="runoob is a great site"
echo ${string:1:4} # 输出 unoo
```

##### 查找子字符串

查找字符 **i** 或 **o** 的位置(哪个字母先出现就计算哪个)：

```bash
string="runoob is a great site"
echo `expr index "$string" io`  # 输出 4
```

#### 数组

##### 定义数组

在 Shell 中，用括号来表示数组，数组元素用"空格"符号分割开。定义数组的一般形式为：

```bash
数组名=(值1 值2 ... 值n)
```

例如：

```bash
array_name=(value0 value1 value2 value3)
```

还可以单独定义数组的各个分量：

```bash
array_name[0]=value0
array_name[1]=value1
array_name[n]=valuen
```

可以不使用连续的下标，而且下标的范围没有限制。

##### 读取数组

读取数组元素值的一般格式是：

```bash
${数组名[下标]}
```

例如：

```bash
valuen=${array_name[n]}
```

使用 @ 符号可以获取数组中的所有元素，例如：

```bash
echo ${array_name[@]}
```

##### 获取数组的长度

获取数组长度的方法与获取字符串长度的方法相同，例如：

```bash
# 取得数组元素的个数
length=${#array_name[@]}
# 或者
length=${#array_name[*]}
# 取得数组单个元素的长度
lengthn=${#array_name[n]}
```

### 特殊变量

shell中有几个特殊的变量是需要记住的，也是应用非常多的

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

## 命令替换

shell脚本最有用的特性之一就是可以从命令输出中提取信息，并将其赋值给变量。 有两种方法可以将命令输出赋值给变量

*   反引号字符(\`)
    
*   $()格式 示例：
    
    ```bash
    today1=`date +%y%m%d`
    today2=$(date + %y%m%d)
    echo $today1
    echo $today2
    ```
    
    上述两种方式都会输出同样的结果
    

## 数学运算

在shell中执行数学运算会比较麻烦，有两种方式可以实现

### expr命令

Bourne shell提供了expr命令处理数学表达式，如 `expr 1 + 2` expr命令操作符

运算符

说明

举例

+

加法

`expr $a + $b` 结果为 30。

\-

减法

`expr $a - $b` 结果为 -10。

\*

乘法

`expr $a \* $b` 结果为 200。

/

除法

`expr $b / $a` 结果为 2。

%

取余

`expr $b % $a` 结果为 0。

|

or

`expr $a $b` $a不为null也不为0返回$a,否则返回$b

&

and

`expr $a & $b` $a,$b均不为null或0返回$a,否则返回0

<

小于

`expr $a < $b` 返回1 或 0

<=

小于等于

`expr $a <= $b` 返回1 或 0

\=

等于

`expr $a = $b` 返回1 或 0

!=

不等于

`expr $a ！= $b` 返回1 或 0

\>=

大于等于

`expr $a >= $b` 返回1 或 0

\>

大于

`expr $a > $b` 返回1 或 0

##### 使用`[]`

在shell中使用`[]`要比`expr`简洁

```bash
var=$[1 + 5]
```

**注意bash shell数学运算只支持整数运算**

## 退出脚本

### 退出状态码

Linux提供了一个专门的变量`$?`来保存上个已执行命令的退出码，在`特殊变量`章节有提到 Linux退出状态码

状态码

描述

0

成功

1

一般性未知错误

2

不适合的shell命令

126

命令不可执行

127

没找到命令

128

无效的退出参数

128+x

与linux信号x相关的严重错误

130

通过Ctrl+C终止的命令

255

正常范围之外的退出状态码

### exit命令

默认情况下，shell脚本会以脚本中最后一个命令的退出状态码退出。 可以通过`exit`命令退出并指定退出码

```bash
exit 0
```

`exit`一般会结合`if-then`使用，在下一篇会介绍相关知识。 下一篇 [shell基础（二）之条件比较命令](https://chengfeng.site/2018/11/11/shell基础（二）之条件比较命令)