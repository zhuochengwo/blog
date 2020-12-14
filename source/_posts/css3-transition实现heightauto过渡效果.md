---
title: 'CSS3 transition实现height:auto过渡效果'
tags:
  - CSS
  - 前端
id: '222'
categories:
  - - CSS
  - - 敲代码
date: 2018-08-13 12:15:47
---

在看w3schools的[CSS Dropdowns](https://www.w3schools.com/css/css_dropdowns.asp)章节的时候，想着给下拉列表加上个过渡动画效果会更好，实际试了下，发现直接height:auto好像还实现不了，找了下，可以通过max-height的方式来实现。 想要实现的效果如下，在鼠标移动到dropdown上时过渡出现下拉列表，这个时候容器的height是不固定的，也是未知的，因为里边的item未知。想当然的想到height:auto，但发现transition效果没有预期出现，这主要是transition实现机制的问题。这里用max-height来实现，将max-height在hover时设置成一个肯定能大于height的值即可。

### 效果：

.dropbtn{background-color:#4caf50;color:white;padding:16px;font-size:16px;border:0;cursor:pointer}.dropdown{position:relative;display:inline-block}.dropdown-content{display:block;position:absolute;background-color:#f9f9f9;min-width:160px;box-shadow:0 8px 16px 0 rgba(0,0,0,0.2);z-index:1;max-height:0;overflow:hidden;transition:max-height .6s}.dropdown-content a{color:black;padding:12px 16px;text-decoration:none;display:block}.dropdown-content a:hover{background-color:#f1f1f1}.dropdown:hover .dropdown-content{max-height:200px;transition:max-height .8s}.dropdown:hover .dropbtn{background-color:#3e8e41}

Move the mouse over the button to open the dropdown menu.

Dropdown

[Link 1](#)[Link 2](#)[Link 3](#)

### 完整代码如下:

```markup
<!DOCTYPE html>
<html>
<head>
<style>
.dropbtn {
    background-color: #4CAF50;
    color: white;
    padding: 16px;
    font-size: 16px;
    border: none;
    cursor: pointer;
}

.dropdown {
    position: relative;
    display: inline-block;
}

.dropdown-content {
    display: block;
    position: absolute;
    background-color: #f9f9f9;
    min-width: 160px;
    box-shadow: 0px 8px 16px 0px rgba(0,0,0,0.2);
    z-index: 1;
    max-height:0;
    overflow:hidden;
    transition:max-height 0.6s;
}

.dropdown-content a {
    color: black;
    padding: 12px 16px;
    text-decoration: none;
    display: block;
}

.dropdown-content a:hover {background-color: #f1f1f1}

.dropdown:hover .dropdown-content {
    max-height:200px;
    transition:max-height 0.8s;
}

.dropdown:hover .dropbtn {
    background-color: #3e8e41;
}
</style>
</head>
<body>

<h2>Dropdown Menu</h2>
<p>Move the mouse over the button to open the dropdown menu.</p>

<div class="dropdown">
  <button class="dropbtn">Dropdown</button>
  <div class="dropdown-content">
    <a href="#">Link 1</a>
    <a href="#">Link 2</a>
    <a href="#">Link 3</a>
  </div>
</div>

<p><strong>Note:</strong> We use href="#" for test links. In a real web site this would be URLs.</p>

</body>
</html>
```

当然，其实还有其它的方式实现，不过这种方式应该是最简单的，这里记录一下。