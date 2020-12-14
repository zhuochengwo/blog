---
title: go相对路径引入模块导致cannot find package的问题
tags:
  - 踩坑
id: '325'
categories:
  - - Go
  - - 敲代码
date: 2018-11-16 17:56:31
---

**今天碰到一个因为使用相对路径引入项目模块导致项目vendor内的包加载不到的问题，这里记录下。**

### 背景

我是用的官方的dep处理依赖管理的，在`GOPATH`目录`git clone`下来项目后，使用`dep ensure install`安装好依赖，准备愉快的开始了。 因为之前的项目名跟gitlib上的项目名不一样，我clone的时候直接使用的gitlab的项目名，所以之前项目中import项目模块需要改一下，如之前项目名是`web`，现在项目名是`app`，那之前`import web/setting`应该改成`import app/setting`。改的时候，我“聪明”的想到了用相对路径来解决，直接改成`import ./setting`这种形式，IDE也正确的帮我识别了路径，一切看起来都很完美。 结果，启动却报错`cannot find package`。尴了个尬了。 go版本是：

```bash
go version go1.11.2 linux/amd64
```

### 解决

一开始也没想到是因为我改了相对路径导致的，因为报错的是项目vendor中的包引用不到，直到找到了这个 [issure](https://github.com/golang/go/issues/15478)提出issue的人跟我碰到的问题是一样的，最后自己自问自答的解决了这个问题，可以参考他自己的[回答](https://github.com/golang/go/issues/15478#issuecomment-215424925)。 问题就是相对路径导致的，需要将我之前改的`import ./setting`的形式改成`impoet app/setting`就可以解决了。 但这个解决方案其实是不完美的，就像issue作者说的，他觉得golang官方应该解决这个问题。 这个issue是2016年提的，到现在2018年了golang官方也没有管这个问题，后边又碰到同样的问题的，可以参考解决了。