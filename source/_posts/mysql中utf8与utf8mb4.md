---
title: MySQL中utf8与utf8mb4
tags:
  - MySQL
id: '210'
categories:
  - - MySQL
  - - 数据库
  - - 敲代码
date: 2018-07-18 11:29:41
---

开发过微信相关的应该都知道，微信昵称中的emoji表情要想保存到MySQL数据库中，使用utf8编码是不行的，需要修改库表字符编码为utf8mb4。以前碰到这个问题的时候也没有深究，只是知道utf8mb4支持的范围比utf8更广。 直到今天看到infoQ的一篇文章[记住，永远不要在MySQL中使用“utf8”](http://www.infoq.com/cn/articles/in-mysql-never-use-utf8-use-utf8) ，才认真的看了下MySQL utf8与utf8mb4的问题。 标准utf8编码使用1~4个字节表示字符，如1个字节表示数字和英文字母，使用3~4个字节表示一个汉字。

### MySQL utf8与utf8mb4

关于MySQL utf8与utf8mb4的区别，[官方文档](https://dev.mysql.com/doc/refman/5.6/en/charset-unicode-utf8mb4.html)是这样说的：

> The `utfmb4` character set has these characteristics:
> 
> *   Supports BMP and supplementary characters.
> *   Requires a maximum of four bytes per multibyte character.
> 
> `utf8mb4` contrasts with the `utf8mb3` character set, which supports only BMP characters and uses a maximum of three bytes per character:

> *   For a BMP character, `utf8mb4` and `utf8mb3` have identical storage characteristics: same code values, same encoding, same length.
> *   For a supplementary character, `utf8mb4` requires four bytes to store it, whereas `utf8mb3` cannot store the character at all. When converting `utf8mb3` columns to`utf8mb4`, you need not worry about converting supplementary characters because there will be none.

mysql支持的 utf8 编码最大字符长度为 3 字节，如果遇到 4 字节的宽字符就会有问题了。MySQL在5.5.3之后增加了这个utf8mb4的编码，mb4就是most bytes 4的意思，专门用来兼容四字节的unicode。其实utf8是utf8mb3的别名，utf8mb4就是utf8的超集并完全兼容utf8，也是真正意义上符合标准utf8编码的。 为了更好的兼容性，应使用 utf8mb4，同时要注意连接字符集也要设置为utf8mb4，否则在 严格模式 下会出现 `Incorrect string value: /xF0/xA1/x8B/xBE/xE5/xA2… for column 'name'`这样的错误，非严格模式下此后的数据会被截断。 还有一点需要注意的是，对于 CHAR 类型数据，utf8mb4 会多消耗一些空间，根据 Mysql 官方建议，使用 VARCHAR  替代 CHAR。

### utf8升级utf8mb4具体步骤

SQL 语句如下：

```sql
# 修改数据库:
ALTER DATABASE database_name CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci;  
# 修改表:
ALTER TABLE table_name CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
# 修改表字段:
ALTER TABLE table_name CHANGE column_name column_name VARCHAR(191) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

修改MySQL配置文件 新增如下参数：

```vim
default-character-set = utf8mb4
default-character-set = utf8mb4
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
init_connect='SET NAMES utf8mb4
```

检查环境变量 和测试 SQL 如下：

```sql
SHOW VARIABLES WHERE Variable_name LIKE 'character\_set\_%' OR Variable_name LIKE 'collation%';
```

### key 768 long 错误

字符集从utf8转到utf8mb4之后，最容易引起的就是索引键超长的问题。 对于表行格式是 `COMPACT`或 `REDUNDANT`，InnoDB有单个索引最大字节数 768 的限制，而字段定义的是能存储的字符数，比如 `VARCHAR(200)` 代表能够存200个汉字，索引定义是字符集类型最大长度算的，即 utf8 maxbytes=3, utf8mb4 maxbytes=4，算下来utf8和utf8mb4两种情况的索引长度分别为600 bytes和800bytes，后者超过了768，导致出错：`Error 1071: Specified key was too long; max key length is 767 bytes`。 `COMPRESSED`和`DYNAMIC`格式不受限制，但也依然不建议索引太长，太浪费空间和cpu搜索资源。 如果已有定义超过这个长度的，可加上前缀索引，如果暂不能加上前缀索引（像唯一索引），可把该字段的字符集改回utf8或latin1。 但是，（ **敲黑板啦，很重要** ），要防止出现 `Illegal mix of collations (utf8_general_ci,IMPLICIT) and (utf8mb4_general_ci,COERCIBLE) for operation '='` 错误：连接字符集使用utf8mb4，但 SELECT/UPDATE where条件有utf8类型的列，且条件右边存在不属于utf8字符，就会触发该异常。