---
title: 使用docker搭建lnmp环境
tags:
  - Docker
  - PHP
id: '111'
categories:
  - - Docker
  - - 敲代码
date: 2018-05-02 19:00:11
---

最近看了点 **docker** 相关的知识，就想着用 **docker** 来搭建一个 **lnmp** 的开发环境。 不多说，动手搞起。 因为我的宿主机是 **ubuntu** 的，之前已经安装好了 **docker**，就不再介绍 **docker** 的安装了，还没有安装 **docker** 的话可以参考官方的[安装教程](https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-using-the-repository)。这里主要介绍依次安装**MySQL**，**PHP**，**NGINX**的 **docker** 容器，顺序上要注意，因为容器之前互相是有依赖的。  

## **1、mysql**

### 1.1拉取 **mysql** 官方的镜像

```bash
docker pull registry.docker-cn.com/library/mysql:5.7.22
```

这里使用国内的镜像加速，不然慢死。直接指定镜像源即可。**mysql:least** 已经到 **8.0.11** 了，这里不想使用 **mysql 8**，所以指定 **mysql** 的版本是 **5.7.22** **执行结果：**

```bash
5.7.22: Pulling from library/mysql
f2aa67a397c4: Pull complete 
1accf44cb7e0: Pull complete 
2d830ea9fa68: Pull complete 
740584693b89: Pull complete 
4d620357ec48: Pull complete 
ac3b7158d73d: Pull complete 
a48d784ee503: Pull complete 
bf1194add2f3: Pull complete 
0e5c74178a02: Pull complete 
c1614226503e: Pull complete 
0038589109a0: Pull complete 
Digest: sha256:e3ce1b609c9275ed24afb3465a9dd73cce38385e64c94755edd5e596a5c1bc8c
Status: Downloaded newer image for registry.docker-cn.com/library/mysql:5.7.22
```

可以通过 **docker images** 查看 **pull** 下来的镜像。

### 1.2创建 **mysql** 容器：

```bash
docker run -it --name dmysql -v /data/mysql/etc:/etc/mysql/conf.d -v /data/mysql/log:/var/log/mysql -v /data/mysql/data:/var/lib/mysql -p 3307:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7.22
```

**参数说明** **\-d** 让容器在后台运行 **\-p** 添加主机到容器的端口映射，映射容器的3306到宿主机的3307端口 **\-v** 挂载目录（目录映射），这样宿主机和容器内的这些目录是同步的。这里挂载了     **mysql** 的配置、数据和日志到 **/data/mysql/** 下 **\--name** 容器的名字，随便取但必须唯一 **\-e** 环境变量， **MYSQL\_ROOT\_PASSWORD** 用于初始化 **mysql root** 用户密码，方便待会儿登录进去 **执行结果：**

```bash
Unable to find image 'mysql:5.7.22' locally
5.7.22: Pulling from library/mysql
Digest: sha256:e3ce1b609c9275ed24afb3465a9dd73cce38385e64c94755edd5e596a5c1bc8c
Status: Downloaded newer image for mysql:5.7.22
b8d88d9315bf78299efbdab1759668365b5d127f6d6c4619be51942c597d7c2d
```

**docker ps -a**查看容器信息:

```bash
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
b8d88d9315bf mysql:5.7.22 "docker-entrypoint..." About a minute ago Up About a minute 0.0.0.0:3307->3306/tcp dmysql
```

使用 **docker exec -it dmysql /bin/bash** 进入到 **mysql** 容器，这里的 **dmysql** 是创建容器时取的名字，也可以使用容器 **id (b8d88d9315bf)**

```bash
docker exec -it dmysql /bin/bash
```

**mysql -uroot -p** 登录 **mysql** 创建容器的时候将容器内的 **3306** 端口映射到了宿主机的 **3307**，我们也可以直接在宿主机连接容器内的mysql，**但要注意指定 host 127.0.0.1 ,默认使用 localhost 是会无法连接成功的**

```bash
mysql -uroot -h127.0.0.1 -P3307 -p
```

使用默认的 **localhost** 的话需要指定 **tcp** 协议连接

```bash
mysql -uroot -P3307 --protocol=tcp -p
```

 

## **2、php**

### 2.1拉取 php-fpm 镜像

```bash
docker pull registry.docker-cn.com/library/php:7.2-fpm
```

```bash
7.2-fpm: Pulling from library/php
f2aa67a397c4: Already exists 
c533bdb78a46: Pull complete 
65a7293804ac: Pull complete 
35a9c1f94aea: Pull complete 
e8295a1cf501: Pull complete 
ecfd4131d449: Pull complete 
de975577344b: Pull complete 
6343baa4101d: Pull complete 
ba60ddde6cbf: Pull complete 
ac788da5f2b6: Pull complete 
05e19938fedb: Pull complete 
Digest: sha256:9c07d6f27ca1076bade65669e289180e06ab04022b64b97e6c406c9810cde672
Status: Downloaded newer image for registry.docker-cn.com/library/php:7.2-fpm
```

### 2.2创建镜像

```bash
docker run -it --name dphp --link dmysql:mysql -v /data/php/project:/var/www/html -p 9001:9000 -d php:7.2-fpm
```

**参数说明:** **\--link** 与另外一个容器建立起联系，这样就可以在当前容器使用另一个容器的服务。这里使用 **dmysql** 容器并在 **dphp** 容器中命名为 **mysql** 这里挂载了项目目录，方便执行 **php** 代码。 **执行结果：**

```bash
Unable to find image 'php:7.2-fpm' locally
7.2-fpm: Pulling from library/php
Digest: sha256:9c07d6f27ca1076bade65669e289180e06ab04022b64b97e6c406c9810cde672
Status: Downloaded newer image for php:7.2-fpm
5b18f3ff8b070e42bd149888506a8e3f62a062d22af9921c58a2e81e94cfc364
```

这样 **php** 的容器就运行起来的，同样可以通过 **docker exec** 进入容器：

```bash
docker exec -it dphp /bin/bash
```

进入容器后通过 **php -m** 发现没有安装 **pdo\_msyql** 扩展，这里需要通过容器内置的工具安装 **php** 的扩展。**注意容器内置工具 [docker-php-ext-install ](https://hub.docker.com/_/php/) 的使用及扩展配置。**

```bash
docker-php-ext-install pdo_mysql
```

再通过 **php -m** 就可以看到扩展已经装上了 通过 **docker restart dphp** 重启容器来重启 **php-fpm**，否则新安装的扩展不会生效

## **3、nginx**

### 3.1拉取 **nginx** 镜像

```bash
docker pull registry.docker-cn.com/library/nginx
```

```bash
nginx docker pull registry.docker-cn.com/library/nginx 
Using default tag: latest
latest: Pulling from library/nginx
f2aa67a397c4: Already exists 
3c091c23e29d: Pull complete 
4a99993b8636: Pull complete 
Digest: sha256:0fb320e2a1b1620b4905facb3447e3d84ad36da0b2c8aa8fe3a5a81d1187b884
Status: Downloaded newer image for registry.docker-cn.com/library/nginx:latest
```

### 3.2创建 **nginx** 容器

```bash
docker run -itd --name dnginx -v /data/php/project:/var/www/html --link dphp:phpfpm -p 8090:80 nginx
```

进入 **nginx** 容器,修改 **/etc/nginx/conf.d/default.cnf** 。**这里有一点需要特别注意，fastcgi\_pass 填入的是 phpfpm:9000，不能直接使用 php 容器的 ip，这里的 phpfpm 是 dphp 容器在 dnginx 容器中的名称**

```nginx
location ~ \.php$ {
    root /var/www/html;
    fastcgi_pass phpfpm:9000;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
}
```

### **3.3重载配置：**

```bash
service nginx reload
```

因为已经在 **dnginx** 容器中挂载了宿主机的 **/data/php/project** 目录到容器 **/var/www/html** ，我们可以在宿主机的此目录下创建项目文件，就可以在 **dnginx** 中访问到了。在宿主机 **/data/php/project** 下新建 **index.php** 文件并简单打印数据库 **user** 的相关信息以便测试三个容器正常工作。

```php
<?php
try {
    $con = new PDO('mysql:host=mysql;dbname=mysql', 'root', '123456');
    $con->query('SET NAMES UTF8');
    $res =  $con->query('SELECT * FROM `user` WHERE User="root"');
    while ($row = $res->fetch(PDO::FETCH_ASSOC)) {
        echo "User:{$row['User']}  Host:{$row['Host']}\n";
    }
} catch (PDOException $e) {
     echo '错误原因：'  . $e->getMessage();
}
```

在宿主机通过 **curl http://localhost:8090/index.php**测试

```bash
curl http://localhost:8090/index.php
User:root Host:localhost
User:root Host:%
```

这里可以看到成功的从 **mysql** 数据库中查出了 **user** 信息，至此 **docker** 的 **lnmp** 已经搭建完成。 有几个地方需要注意，一般我们会把容器内配置，数据，日志单独挂载到物理机或者单独的数据卷，如本文例子中 **nginx** 的配置是在容器内的，其实是不建议这样做的，进入容器修改配置我们还需要单独安装 **vim**，还有就是 **nginx** 的日志，默认是会直接输出到 **docker log** 的，需要通过 **docker logs nginx** 容器名来查看日志。再一个就是容器的监控， **docker** 有自带的 [healthcheck](https://docs.docker.com/engine/reference/builder/#healthcheck)，也是需要了解的。