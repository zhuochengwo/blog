---
title: laradock workspace启用xdebug单元测试覆盖率分析
tags: []
id: '506'
categories:
  - - 敲代码
date: 2019-06-26 18:18:00
---

因为需要做单元测试覆盖率的分析，需要`xdebug`扩展，之前`laradock`创建`worksapce`时是没有安装`xdebug`的，所以需要重新安装。 修改`laradock`中`.env`文件

```vim
WORKSPACE_INSTALL_XDEBUG=true
```

修改完后重新`build`

```bash
docker-compose build workspace
```

然后我的却没有`build`成功，报了个错

```log
Err:1 http://ppa.launchpad.net/ondrej/php/ubuntu xenial/main amd64 php-xdebug amd64 2.6.0+2.5.5-1+ubuntu16.04.1+deb.sury.org+1
  404  Not Found
E: Failed to fetch http://ppa.launchpad.net/ondrej/php/ubuntu/pool/main/x/xdebug/php-xdebug_2.6.0+2.5.5-1+ubuntu16.04.1+deb.sury.org+1_amd64.deb  404  Not Found

E: Unable to fetch some archives, maybe run apt-get update or try with --fix-missing?
ERROR: Service 'workspace' failed to build: The command '/bin/sh -c if [ ${INSTALL_XDEBUG} = true ]; then     apt-get install -y php${LARADOCK_PHP_VERSION}-xdebug &&     sed -i 's/^;//g' /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/20-xdebug.ini &&     echo "alias phpunit='php -dzend_extension=xdebug.so /var/www/vendor/bin/phpunit'" >> ~/.bashrc ;fi' returned a non-zero code: 100
```

这个问题是因为源失效了。 修改`laradock/workspace/Dockerfile`文件中**261**行原`apt-get install -y php${LARADOCK_PHP_VERSION}-xdebug && \` 为 `apt-get update && apt-get install -y php${LARADOCK_PHP_VERSION}-xdebug && \` 以下是正确配置：

```vim
RUN if [ ${INSTALL_XDEBUG} = true ]; then \
    # Load the xdebug extension only with phpunit commands
    apt-get update && apt-get install -y php${LARADOCK_PHP_VERSION}-xdebug && \
    sed -i 's/^;//g' /etc/php/${LARADOCK_PHP_VERSION}/cli/conf.d/20-xdebug.ini && \
    echo "alias phpunit='php -dzend_extension=xdebug.so /var/www/vendor/bin/phpunit'" >> ~/.bashrc \
;fi
```

更多信息可以关注`laradock`的`issue`：[issue#1847](https://github.com/laradock/laradock/issues/1847) 改完以后就能成功创建了。 然后重启容器 `docker-compose up -d workspace`，进入`workspace`容器通过`php -m grep xdebug`能看到`xdenug`正常安装了。 通过`phpunit --coverage-html ./tests/codeCoverage`输出单元测试覆盖率的分析结构，可以通过浏览器打开查看，也可以自行修改`./tests/codeCoverage`结果输出目录。