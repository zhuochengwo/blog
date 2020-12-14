---
title: shell基础（三）之循环命令
tags:
  - Shell
  - shell基础系列
id: '328'
categories:
  - - Shell
  - - 敲代码
date: 2018-11-22 09:51:34
---

## 概述

**这篇文章是[Shell基础系列](https://chengfeng.site/tag/shell%E5%9F%BA%E7%A1%80/)文章的第三篇** 这一章主要讲bash shell的循环命令之`for` `while` `until`

## for命令

### 列表的遍历

语法

```bash
for var in list
do
    commands
done
```

示例

```bash
for val in C Java PHP Python 
do
    echo "Language:$val"
done
```

for循环是以空格作为分割符的，如果某个单独的数据值里边有空格，需要使用`""`包裹起来;如果里边有特殊符号如`'` `+`等，同样要用`""`包裹起来，当然也可以使用转义字符`\`转义

```bash
for val in C Java "PHP Python" "C++" C\+\+
do
    echo "Language:$val"
done
```

输出结果是：

```vim
Language:C
Language:Java
Language:PHP Python
Language:C++
Language:C++
```

也可以将list放入变量中，对变量进行遍历。需要注意的是，变量需要用`""`像定义字符串一样。shell对于类型定义没有特别严格，可以认为有空格的字符串都可以当成列表来遍历。

```bash
list="C Java PHP Python"
for val in $list
do
    echo "Language:$val"
done
```

#### 更改字段分隔符

默认情况下，`bash shell`以下列字符作为字段分隔符

*   空格
*   制表符
*   换行符

这些字符保存在环境遍历`IFS`中，叫做`内部字段分隔符` 可以通过临时更改环境变量`IFS`的值来改变shell的分隔符字符，如`IFS=$'\n'` 这样就只能使用换行符作为分隔符了。 如果脚本比较长，临时改完`IFS`后，希望后边的脚本继续使用`IFS`默认的值，可以通过下边的方式实现

```bash
IFS.OLD=$IFS
IFS=$'\n'
IFS=$IFS.OLD
```

### 数组的遍历

对于数组的遍历，基本跟列表一样

```bash
arr=(C Java "PHP" "Python")  # 双引号可要可不要，同样以空格拆分
for val in ${arr[@]}
do
    echo "Language:$val"
done
```

### 从命令读取值

可以从命令的输出作为列表的值来遍历

```bash
file="languages.txt"
for lang in $(cat $file)
do
    echo "Language:$lang"
done
```

`languages.txt`文件中的内容

```vim
cat languages.txt 
C
Java
PHP
Python
```

脚本输出结果

```vim
Language:C
Language:Java
Language:PHP
Language:Python
```

### 遍历目录

结合之前章节的内容，可以写一个简单的目录遍历的脚本

```bash
for file in ./*.sh ./*.txt # 可以将任意多的通配符放进列表
do
    if [ -d "$file" ]   # 这里使用双引号因为目录可能有空格
    then
        echo "$file is a directory"
    elif [ -f "$file" ]
    then
        echo "$file is a file"
    fi
done
```

当前目录下的文件是

```vim
-rw-rw-r-- 1 chengfeng chengfeng   72 4月  26  2018 clean.sh
drwxr-xr-x 2 chengfeng chengfeng 4.0K 11月 12 22:36 data
-rw-r--r-- 1 chengfeng chengfeng   18 11月 12 22:26 languages.txt
drwxr-xr-x 2 chengfeng chengfeng 4.0K 11月 12 22:35 log
-rwxr--r-- 1 chengfeng chengfeng  255 11月 12 22:56 test.sh
```

脚本输出结果

```vim
./clean.sh is a file
./test.sh is a file
./languages.txt is a file
```

## C语言风格的for命令

基本语法

```bash
for (( variable assignment ; condition ; iteration process ))
```

示例

```bash
for (( i=1 ; i<=5 ; i++ )) # 也可以使用多个变量 for (( i=1, j = 1; i<=5; i++ ))
do
    echo "The num is $i"
done
```

## while 命令

基本语法

```bash
while test command
do
    commands
done
```

其中`test`与`if`语句的一样，同时也可以使用`[]`的方式 示例

```bash
var1=10
while [ $val -gt 0 ]
do
    echo $var1
    var1=$[ $var1 - 1 ]
done
```

## until 命令

基本语法

```bash
until test command
do
    commands
done
```

示例

```bash
var1=10
until [ $var1 -eq 0 ]
do
    echo $var1
    var1=$[ $var1 - 1 ]
done
```

## 控制循环

有循环的地方就会有控制循环，`break`和`continue`命令就是用来控制循环的

### break命令

`break`语法非常简单，主要需要考虑的是在多层循环的特殊之处。 通过指定`break n`来控制`break`跳出循环层级 **示例**

```bash
for (( a = 1; a < 4; a++ ))
do
    echo "Outer loop:$a"
    for (( b = 1; b < 100; b++ ))
    do
        if [ $b -gt 4 ]
        then
            break 2     #这里会跳出外部循环，如果直接 break 或 break 1则只是跳出当前循环
        fi
        echo " Inner loop $b"
    done
done
```

### continue 命令

`continue`命令与`break`命令类似，同样主要关注多层循环的特殊之处 通过`continue n`来控制要继续循环的层级 **示例**

```bash
for (( a = 1; a <= 5; a++ ))
do
    echo "Outer loop:$a"
    for (( b = 1; b < 5; b++ ))
    do
        if [ $b -gt 2 ] && [ $b -lt 4 ]
        then
            continue 2      #这里会继续外部循环，如果直接 continue 或 continue 1则只是继续当前循环
        fi
        res=$[ $a * $b ]
        echo " The restult of $a * $b is $res"
    done
done
```

## 处理循环输出

在`shell`中，可以对循环的输出使用管道或进行重定向，通过对`done`命令之后添加一个处理命令来实现。 这里通过之前的遍历目录的例子来展示。

```bash
for file in ./*.sh ./*.txt
do
    if [ -d "$file" ]
    then
        echo "$file is a directory"
    elif [ -f "$file" ]
    then
        echo "$file is a file"
    fi
done > output.txt   # 将输出重定向到output.txt文件中
```

最后，`for`循环的结果都会重定向到output.txt文件，不会展示到屏幕上。 上一篇：[Shell基础（二）之条件比较命令](https://chengfeng.site/2018/11/11/shell基础（二）之条件比较命令) 下一篇：[shell基础（四）之处理用户输入](https://chengfeng.site/2018/12/07/shell基础（四）之处理用户输入/)