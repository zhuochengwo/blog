---
title: Nodejs获取命令行参数
tags:
  - JavaScript
  - Nodejs
id: '102'
categories:
  - - Nodejs
  - - 敲代码
date: 2018-03-22 19:03:53
---

使用Nodejs开发命令行程序就不得不解决命令行参数的问题，好在Nodejs有强大的第三方库支持，我们只需要站在巨人的肩膀上完成自己的业务逻辑 我们都知道，Nodejs命令行参数我们可以简单的通过process.argv获取所有的参数

```javascript
#!/usr/bin/env node
let argv = process.argv;
console.log(argv);
```

很快我们会发现，我们通过proces.argv，简单的参数解析还比较好弄，但像-n,--name这样的参数，而且参数个数又比较多的情况下，简直就是噩梦。这个时候就需要我们强大的解析库了[commder.js](https://github.com/tj/commander.js) 先安装commder.js

```bash
npm install commander --save
```

然后在项目中引入就可以了，以下是个简单的示例：

```javascript
const Options = require('commander');

Options
    .version('1.0.0', '-v,--version')
    .usage('node test [options]')
    .option('-n,--num <n>', 'num', parseInt)
    .option('-l,--list', 'list items', doList)
    .parse(process.argv);

function doList() {
    [1, 2, 3].forEach((item) => {
        console.log("List item:" + item);
    })
}

if (!Options.num && !Options.list) {
    Options.help();
} else {
    if (Options.num) console.log('num is:' + Options.num);
    if (Options.list) console.log('list items:' + Options.list);
}
```

以下是执行结果：

```bash
tools git:(master) ✗ node test   

  Usage: test node test [options]

  Options:

    -v,--version  output the version number
    -n,--num <n>  num
    -l,--list     list items
    -h, --help    output usage information
➜  tools git:(master) ✗ node test -l
List item:1
List item:2
List item:3
list items:true
➜  tools git:(master) ✗ node test -n 100
num is:100
➜  tools git:(master) ✗ node test --num=1000
num is:1000
➜  tools git:(master) ✗ node test --num=1000 --list
List item:1
List item:2
List item:3
num is:1000
list items:true
➜  tools git:(master) ✗ node test --num            

  error: option `-n,--num <n>' argument missing
➜  tools git:(master) ✗
```

当然这只是个简单示例，实际功能相当强大，支持非常多的高级功能，具体还需要根据自己需求参考官方文档实践。 对比之前用的python的getopt，commder.js实在是好用太多，直观明了。 Options.help()能自动输出文档，这是一个非常棒的功能，而且能自定义内容，option的第三个参数支持自定义函数。 需要注意的是，要获取值必须使用类似这样的方式option('-n,--num ', 'num')，其中表示必填参数，没有传入的情况下会报错，如上面结果输出的error: option \`-n,--num ' argument missing，如果改用\[n\]则代表选填参数。类似option('-l,--list', 'list items')，标示默认获取的是bool值