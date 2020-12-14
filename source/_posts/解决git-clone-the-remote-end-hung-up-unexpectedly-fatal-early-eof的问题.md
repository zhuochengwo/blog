---
title: '解决Git clone时The remote end hung up unexpectedly fatal: early EOF的问题'
tags:
  - Git
  - 踩坑
id: '163'
categories:
  - - Git
  - - 敲代码
date: 2018-06-02 14:51:03
---

在git clone项目的时候碰到这样一个错误：

```bash
git clone git@gitee.com:chengfengGit/blog.git
Cloning into 'blog'...
The authenticity of host 'gitee.com (120.55.226.24)' can't be established.
ECDSA key fingerprint is SHA256:FQGC9Kn/eye1W8icdBgrQp+KkGYoFgbVr17bmjey0Wc.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'gitee.com,120.55.226.24' (ECDSA) to the list of known hosts.
remote: Counting objects: 4389, done.
remote: Compressing objects: 100% (3238/3238), done.
packet_write_wait: Connection to 120.55.226.24 port 22: Broken pipe
fatal: The remote end hung up unexpectedly
fatal: early EOF
fatal: index-pack failed
```

网上找了下，解决方法如下：

```bash
git config --global core.compression 0
```

退出重新登录就好了。 碰到了两次，记录下，免得每次都去找