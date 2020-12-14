---
title: ubuntu升级18.04进不了桌面
tags:
  - Ubuntu
id: '118'
categories:
  - - Ubuntu
  - - 敲代码
date: 2018-05-02 15:20:19
---

4月26日，`ubuntu`如期发布了其长期稳定版`18.04`，我的虚拟机内`ubuntu`是`16.04`，第一时间就升级了，除了遇到一个显卡驱动兼容问题外，一切都很顺利。解决这个问题也很简单，在`virtualbox`设置界面中`[显示]`选项去掉启用`3D加速`即可。 看起来升级挺容易，没什么问题，于是决定升级下公司工作用的那台`ubuntu`了。版本是`17.04`的，之前升级过一次，后边没有再次升级到最新的`17.10`。网上找了一圈，好像没办法直接从`17.04`升级到`18.04`，也就没管了，直接用官方的方式先升级到`17.10`。 电脑比较渣，升级了半天才搞完，重启后发现进不了桌面了，这就有点尴尬了。没别的办法`google`走起，方法也就是重装桌面什么的。 `“ctrl alt + f2”`进入命令行界面后，鼓捣了下似乎没啥作用，问题是`apt update`就执行不了。有点不想弄了，准备直接重装，反正u盘都刻好了。但看了`ubuntu`提示当前版本`17.10`有新版本`18.04`可以升级，抱着试一试的态度直接又执行升级了，万一升级后就好了呢！就像`windows`的重启大法一样。 等折腾了半天新版本升级安装完成后，发现问题还是存在，继续进不了桌面。这下没法子了，只能硬着头皮网上找方法。 找到的方法就是重装桌面

```bash
sudo apt-get update

sudo apt-get install --reinstall ubuntu-desktop

sudo apt-get install unity
```

第一条执行就报错了

```bash
sudo apt-get update 
Err:1 http://archive.ubuntu.com/ubuntu bionic InRelease
Could not resolve 'archive.ubuntu.com'
Err:2 http://archive.ubuntu.com/ubuntu bionic-updates InRelease
Could not resolve 'archive.ubuntu.com'
Err:3 http://archive.ubuntu.com/ubuntu bionic-backports InRelease
Could not resolve 'archive.ubuntu.com'
Err:4 http://archive.ubuntu.com/ubuntu bionic-security InRelease
Could not resolve 'archive.ubuntu.com'
Reading package lists... Done
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic/InRelease Could not resolve 'archive.ubuntu.com'
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic-updates/InRelease Could not resolve 'archive.ubuntu.com'
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic-backports/InRelease Could not resolve 'archive.ubuntu.com'
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic-security/InRelease Could not resolve 'archive.ubuntu.com'
W: Some index files failed to download. They have been ignored, or old ones used instead.
```

排查了下是dns的问题，于是

```bash
sudo vim /etc/resolv.conf

nameserver 8.8.8.8
```

然后还是不行

```bash
sudo apt-get update 
Hit:1 http://archive.ubuntu.com/ubuntu bionic InRelease
```

这个是之前编辑过源的问题，只能是重新搞回来了。正好虚拟机的sources.list可以参考

```bash
vim /etc/apt/sources.list
```

写入如下内容

```vim
deb http://archive.ubuntu.com/ubuntu bionic main restricted

deb http://archive.ubuntu.com/ubuntu bionic-updates main restricted


deb http://archive.ubuntu.com/ubuntu bionic universe


deb http://archive.ubuntu.com/ubuntu bionic-updates universe
deb http://archive.ubuntu.com/ubuntu bionic multiverse
deb http://archive.ubuntu.com/ubuntu bionic-updates multiverse
deb http://archive.ubuntu.com/ubuntu bionic-backports main restricted universe multiverse

deb http://archive.ubuntu.com/ubuntu bionic-security main restricted
deb http://archive.ubuntu.com/ubuntu bionic-security universe
deb http://archive.ubuntu.com/ubuntu bionic-security multiverse
```

然后执行安装，重启搞定 `18.04`桌面有一个回收站和挂载的盘，因为我个人喜欢干干净净的桌面，所以想关掉这些图标，而且窗口的控制最大化、最小化和关闭页挪到了右边有点不喜欢，这些都可以通过安装`gnome-tweak-tool`来解决

```bash
sudo apt-get install gnome-shell gnome-tweak-tool
```

打开`tewak`就可以设置这些内容了 去掉桌面图标 ![](https://blog-1252719385.cos.ap-guangzhou.myqcloud.com/images/20180502153708.png) 窗口控制按钮挪到左边 ![](https://blog-1252719385.cos.ap-guangzhou.myqcloud.com/images/20180502153749.png) 到这里基本就可以愉快的使用新版`ubuntu 18.04`了。