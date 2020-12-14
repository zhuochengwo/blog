---
title: 宿主机Nginx 配合 Docker php-fpm 遇到的问题
tags: []
id: '489'
categories:
  - - Docker
  - - 敲代码
date: 2019-05-14 23:45:33
---

今天把博客迁移从自己的 `GCP` 迁移到了朋友的服务器上，开始了寄居生活（这里要感谢朋友的收留）。 这次用 `docker` 来部署了，确实简单方便好用。 因为朋友的服务器上已经有安装 `Nginx` 在用，`80` 端口冲突，没办法使用容器中的 `Nginx`了。采用的是宿主机的 `Nginx` 配合容器中的 `php-fpm` 来使用，其它如 `MySQL` 、`Redis` 等也均用 `docker-compose` 来管理了。 `php-fpm` 映射 `9001:9000`，数据库连接 `host` 注意使用 `--link` 的名字，`Nginx` 配置 `fastcgi_pass 127.0.0.1:9001`，这些都没什么好说的。 配置好后以为一切顺利，没想到跑步起来，访问网站报了个404，查看`Nginx`日志发现了如下错误： `FastCGI sent in stderr: "Primary script unknown" while reading response header from upstream` 一头雾水，当然是求助万能的Google了。 原来是`Nginx`配置`fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;`有问题。因为`php-fpm`运行在容器中，其中挂载的项目路径是`/aaa/bbb/blog:/var/www/blog`，这里 `$document_root`是`Nginx`中配置的`root`参数`/aaa/bbb/blog`与容器中项目路径`/var/www/blog`不一致就导致这个问题了，将配置修改为 `fastcgi_param SCRIPT_FILENAME /var/www/blog/$fastcgi_script_name;`即可解决。 [参考资料](https://segmentfault.com/q/1010000007077122) 暂时没有发现其它运行的问题，不过有一部分图片没有使用`cos`保存在原来的服务器还得迁移过来才行，顺便统一上传到`cos`算了。