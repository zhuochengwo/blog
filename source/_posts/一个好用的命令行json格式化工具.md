---
title: 一个好用的命令行json格式化工具
tags:
  - json
id: '389'
categories:
  - - 分享发现
  - - 工具
date: 2019-03-21 12:29:16
---

日常编码中，我们或多或少的会跟`json`打交道，一般对于接口返回的`json`，我们有浏览器的插件来格式化，或者类似`postman`这类工具也都是支持`json`格式化的。 我们直接复制的一段`json`，可能更多的是用一些在线的json格式化的工具来查看，但这样总感觉不是很爽，最近发现了一个命令行的`json`可视化工具，非常棒。格式化输出结构，支持鼠标操作，查找，展开/折叠等等等等，浏览器插件有的功能都有。 我要介绍的就是 **[fx](https://github.com/antonmedv/fx)** ，项目在github开源。 ![使用效果](https://camo.githubusercontent.com/b5df8c57792e443a18a56cd9a292b1a101ba2391/68747470733a2f2f6d6564762e696f2f6173736574732f66782e676966 "使用效果")

### 安装

简单的通过 `npm` 来安装，就可以愉快的开始享受了

```bash
npm install -g fx
```

### 使用

1.  复制json内容调用工具 这是我用这个工具用的最多的一种方式，直接复制一段`json`，命令行执行

```bash
echo '{"code":0,"message":"success","data":{"user_id":1,"username":"hello world","email":"1111@gmail.com","gender":1,"detail":{"address":"China","people":[1,2,3,4,5,6]}}}'  fx
```

2.  直接打开json文件

```bash
fx data.json
```

3.  格式化接口返回内容

```bash
curl https://movie.douban.com/j/search_subjects?type=movie&tag=%E7%83%AD%E9%97%A8&page_limit=50&page_start=0  fx
```

更多使用方式可以查看[官方文档](https://github.com/antonmedv/fx)