---
title: Shell基础（二）之条件比较命令
tags:
  - Shell
  - shell基础系列
id: '316'
categories:
  - - Shell
  - - 敲代码
date: 2018-11-11 19:42:15
---

## 概述

**这篇文章是[Shell基础系列](https://chengfeng.site/tag/shell%E5%9F%BA%E7%A1%80/)文章的第二篇** 本篇涉及shell的 `if` `test` `数值比较` `字符串比较` `文件比较` `双括号(( ))` `双方括号[[ ]]` `case`相关基础知识，可以用于使用速查。

#### if命令

```bash
if condition # 或者 if condition;then
then
    commands
elif condition
then
    commands
else
    commands
if
```

#### test & \[\]

`test`能用来在测试条件，`[]`在shell中跟`test`效果一样 `test`可以用来：

*   数值比较
*   字符串比较
*   文件比较

```bash
# 测试变量是否存在
if test $var # 或者 if [ condition ]
then
    command
fi
```

#### 复合条件测试

*   `[ condition1 ]&& [ condition2 ]`
*   `[ condition1 ] [ condition2 ]`

#### 数值比较

比较

描述

n1 -eq n2

n1 == n2

n1 -ge n2

n1 >= n2

n1 -gt n2

n1 > n2

n1 -le n2

n1 <= n2

n1 -lt n2

n1 < n2

n1 -ne n2

n1 != n2

#### 字符串比较

比较

描述

str1 = str2

str1 == str2

str1 != str2

str1 != str2

str1 > str2

str1 > str2 使用时要转义 `[ str1 \> str2 ]`

str1 < str2

str1 < str2 使用时要转义 `[ str1 \< str2 ]`

\-n str

检查str的长度是否非0

\-z str

检查str的长度是否为0

字符串比较大小的时候，有大小写的情况下`test`认为大写字母小于小写字母，与`sort`命令相反

#### 文件比较

比较

描述

\-d file

检查file是否存在且是一个目录

\-e file

检查file是否存在

\-f file

检查file是否存在且是一个文件

\-r file

检查file是否存在且可读

\-s file

检查file是否存在且非空

\-w file

检查file是否存在且可写

\-x file

检查file是否存在且可执行

\-O file

检查file是否存在且属当前用户所有

\-G file

检查file是否存在且默认组与当前用户相同

file1 -nt file2

检查file1是否比file2新

file1 -ot file2

检查file2是否比file2旧

#### 双括号`(( expression ))`数学表达式

**双括号命令符号**

符号

描述

var++

后增

var--

后减

++var

先增

\--var

先减

!

逻辑反

~

位求反

\*\*

幂运算

<<

左位移

\>>

右位移

&

位布尔和and

|

位布尔或or

&&

逻辑和and

||

逻辑或or

**示例**

```bash
val1=10
if (( $val1 ** 2 > 90 ))
then
    (( val2 = $val1 ** 2 ))
    echo "The square of $val1 is $val2"
fi
```

#### 双方括号`[[ expression ]]`字符串

这种方式提供了`test`未提供的一个特效 —— 模式匹配

```bash
if [[ $USER == c* ]] # 匹配c开头
then
    echo "Hello $USER"
else
    echo "Sorry,I do not know you"
fi
```

#### case 命令

**语法：**

```bash
case varidable in
pattern1  pattern2) conmmands1;;
pattern3) conmmands2;;
*) default commands;;
esac
```

**示例：**

```bash
case $USER in
chengfeng  fengcheng) # 当然，只有一个条件也是可以的
    echo "Welcome $USER";;
testing)
    echo "Special testing account";;
*)
    echo "Sorry,you are not allowed here";;
esac
```

**基于这些命令，就可以处理shell编程中复杂的条件比较场景了，抓紧时间练习才能更加深刻的掌握这些知识。** 上一篇：[shell基础（一）之构建基础脚本](https://chengfeng.site/2018/11/11/shell基础（一）之构建基础脚本/) 下一篇：[shell基础（三）之循环命令](https://chengfeng.site/2018/11/22/shell基础（三）之循环命令/)