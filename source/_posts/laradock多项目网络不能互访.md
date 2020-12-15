---
title: laradock多项目互相访问的问题
tags:
  - laradock
id: '347'
categories:
  - - Docker
  - - PHP
  - - 敲代码
date: 2019-01-13 15:35:00
---

## 问题

laravdock能够非常方便的创建开发环境，但在开发中，我们可能需要多个项目互相访问。 例如，我们有`aaa.dev`，`bbb.dev`两个项目，`aaa.dev`需要请求`bbb.dev`项目的某些接口 由于`docker`容器的隔离，我们是不能够直接访问的。此时如果在`aaa.dev`使用`GuzzleHttp`访问`bbb.dev`的接口，就会得到如下错误：

```vim
cURL error 7: Failed to connect to bbb.dev port 80: Connection refused
```

这篇文章主要探讨怎么怎么解决这个问题。

## 解决

一开始找到的方案都是通过添加`php-fpm`中`extra_hosts`的配置来实现的，这种方案比较麻烦，配置好后还需要需要`rebuild` `wordspace`和`php-fpm`两个容器，这里主要介绍一种更加简单的方法。 找到`docker-compose.yml`中`nginx`配置 原`nginx`中`networks`配置如下：

```dockerfile
networks:
        - frontend
        - backend
```

将其修改为:

```dockerfile
networks:
        frontend:
         aliases:
          - bbb.dev
        backend:
         aliases:
          - bbb.dev
```

此配置是运行时配置，不需要`rebulid`，执行`docker-compose up -d nginx`会`recreate`容器，配置就能生效 `nginx`模块完整配置如下

```dockerfile
### NGINX Server #########################################
    nginx:
      build:
        context: ./nginx
        args:
          - PHP_UPSTREAM_CONTAINER=${NGINX_PHP_UPSTREAM_CONTAINER}
          - PHP_UPSTREAM_PORT=${NGINX_PHP_UPSTREAM_PORT}
          - CHANGE_SOURCE=${CHANGE_SOURCE}
      volumes:
        - ${APP_CODE_PATH_HOST}:${APP_CODE_PATH_CONTAINER}
        - ${NGINX_HOST_LOG_PATH}:/var/log/nginx
        - ${NGINX_SITES_PATH}:/etc/nginx/sites-available
        - ${NGINX_SSL_PATH}:/etc/nginx/ssl
      ports:
        - "${NGINX_HOST_HTTP_PORT}:80"
        - "${NGINX_HOST_HTTPS_PORT}:443"
      depends_on:
        - php-fpm
      networks:
        frontend:
         aliases:
          - bbb.dev
        backend:
         aliases:
          - bbb.dev
```

到这里基本就解决这个问题了，同时`aliases`配置的是个数组意味我们可以配置很多个项目来实现我们在`laradock`中多项目的互相访问。 关于此问题更多的信息可以关注`laradock`的[issue#435](https://github.com/laradock/laradock/issues/435),本篇文章的解决方案也来自于此issue中[iceheat的回复](https://github.com/laradock/laradock/issues/435#issuecomment-374576781)，对于`extra_hosts`的配置也有讨论，可以自行摸索。