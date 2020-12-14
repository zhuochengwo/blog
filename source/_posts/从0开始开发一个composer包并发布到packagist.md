---
title: 从0开始开发一个composer包并发布到packagist
tags:
  - PHP
id: '55'
categories:
  - - PHP
  - - 敲代码
date: 2018-02-08 13:49:06
---

composer包对于php来说是革命性的，平常我们开发都会用到各种各样的composer包，现在我们来着手从0开始开发一个php的composer包。 **首先安装composer** 已经安装了的话请跳过。 几行命令就搞定了，不解释，关于composer的更多内容可以参考[composer](https://getcomposer.org/)官方介绍

```bash
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('SHA384', 'composer-setup.php') === '544e09ee996cdf60ece3804abc52599c22b1f40f4323403c44d44fdfdd586475ca9813a858088ffbc1f233e9b180f061') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
sudo mv composer.phar /usr/bin/composer
```

由于众所周知的原因，我们把默认的包镜像源换成国内的

```bash
composer config -g repo.packagist composer https://packagist.phpcomposer.com
```

**初始化composer.json** 我们可以使用composer自带的composer init初始化项目，非常方便。它支持以下参数，也可以不带参数而通过交互模式输入，可以通过composer init --help查看详细介绍

*   `--name` - 包的名称
*   `--description` - 包的描述
*   `--author` - 包的作者
*   `--homepage` - 包的主页
*   `--require` - 需要依赖的其它包,必须要有一个版本约束,并且应该遵循 foo/bar:1.0.0 这样的格式
*   `--require-dev` - 开发版的依赖包,内容格式与 --require 相同
*   `--stability (-s)` - minimum-stability 字段的值

初始化完后的composer.json大概是这个样子的：

```json
{
    "name": "chengfeng/demo",
    "description": "This project is for studying php composer package development.",
    "type": "library",
    "require": {
        "nesbot/carbon": "^1.22"
    },
    "license": "MIT",
    "authors": [
        {
            "name": "chengfeng",
            "email": "****@gmail.com"
        }
    ]
}
```

其中关于minimun-stability的解释可以看这里 [https://docs.phpcomposer.com/04-schema.html#minimum-stability](https://docs.phpcomposer.com/04-schema.html#minimum-stability) 这里使用了默认的stable 这里我们尝试配置一个依赖carbon，也可以选择跳过，在初始化完后编辑composer.json添加依赖，或通过composer require安装 在项目根目录下新建src目录，用于存放项目源文件，在这里我们再建立一个Demo的目录并创建一个Demo.php的文件，稍后再来编写具体的逻辑。同时，再建立一个tests目录并创建DemoTest的测试用例文件，用于编写我们的test case。然后在composer.json配置好autoload，和autoload-dev。 autoload我们按照prs-4规范配置

```json
"autoload": {
    "psr-4": {
        "Demo\\": "src/Demo/"
    }
},
"autoload-dev": {
    "psr-4": {
        "Test\\": "tests/"
    }
}
```

`composer install`安装项目所有依赖，到这里，前期的工作已经做完了，可以开始项目的开发。 现在,整个项目目录结构是这样的：

```vim
demo
    
    -src
       -Demo
           -Demo.php
       - ...
       
    -tests
       -DemoTest.php
       -...
     
    -verdor
    -composer.json
    -composer.lock
    -
```

**配置phpunit** 在开发中，可能会引入一些第三方包，但在生产环境又是不需要的，比如phpunit，我们可以通过--dev参数来安装

```bash
composer require phpunit/phpunit --dev
```

使用phpunit时可以提前配置phpunit.xml 以下是一个简单示例，具体配置可以参考phpunit文档

```markup
<?xml version="1.0" encoding="UTF-8"?>
<phpunit backupGlobals="false"
         backupStaticAttributes="false"
         bootstrap="vendor/autoload.php"
         colors="true"
         convertErrorsToExceptions="true"
         convertNoticesToExceptions="true"
         convertWarningsToExceptions="true"
         processIsolation="false"
         stopOnFailure="false"
>

    <filter>
        <whitelist>
            <directory>src/Demo</directory>
        </whitelist>
    </filter>

    <testsuites>
        <testsuite name="Demo Test Suite">
            <directory>tests</directory>
        </testsuite>
    </testsuites>
</phpunit>
```

配置好了phpunit，我们就可以通过./vendor/bin/phpunit来执行测试用例了。 **正式开始写代码** 在Demo.php中写一个简单的获取今天昨天明天日期的方法

```php
<?php

namespace Demo;

use Carbon\Carbon;

class Demo
{
    public static function getToday()
    {
        return Carbon::today()->format('Y-m-d');
    }

    public static function getYesterday()
    {
        return Carbon::yesterday()->format('Y-m-d');
    }

    public static function getTomorrow()
    {
        return Carbon::tomorrow()->format('Y-m-d');
    }
}
```

DemoTest.php写一个简单的测试用例

```php
<?php


use Demo\Demo;

class DemoTest extends \PHPUnit\Framework\TestCase
{
    public function testToday()
    {
        $today = date('Y-m-d');
        $testToday = Demo::getToday();
        $this->assertEquals($today, $testToday);
    }

    public function testYesterday()
    {
        $yesterday = date('Y-m-d', time() - 86400);
        $testYesterday = Demo::getYesterday();
        $this->assertEquals($yesterday, $testYesterday);
    }

    public function testTomorrow()
    {
        $tomorrow = date('Y-m-d', time() + 86400);
        $testTomorrow = Demo::getTomorrow();
        $this->assertEquals($tomorrow, $testTomorrow);
    }
}
```

现在可以执行下测试用例看看效果

```vim
demo ./vendor/bin/phpunit
PHPUnit 6.5.6 by Sebastian Bergmann and contributors.

... 3 / 3 (100%)

Time: 22 ms, Memory: 4.00MB

OK (3 tests, 3 assertions)
```

完美，测试用例全部通过。 我们也可以通过composer scripts来来定义一些脚步，参考carbon写了同样的配置

```json
"scripts":{
    "test": "./vendor/bin/phpunit; ./vendor/bin/php-cs-fixer fix -v --diff --dry-run;",
    "phpunit": "./vendor/bin/phpunit;",
    "phpcs": "./vendor/bin/php-cs-fixer fix -v --diff --dry-run;"
}
```

这样就可以通过composer run-script phpunit来执行单元测试了。关于script更多强大的功能可以参考 [composer文档](https://docs.phpcomposer.com/04-schema.html#scripts) 到这里，这个包已经开发完了。最后composer.json是这个样子的

```json
{
    "name": "chengfeng/demo",
    "description": "This project is for studying php composer package development",
    "type": "library",
    "homepage": "https://chengfeng.site",
    "require": {
        "nesbot/carbon": "^1.22"
    },
    "license": "MIT",
    "autoload": {
        "psr-4": {
            "Demo\\": "src/Demo/"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "Test\\": "tests/"
        }
    },
    "authors": [
        {
            "name": "chengfeng",
            "email": "drondyw@gmail.com"
        }
    ],
    "require-dev": {
        "phpunit/phpunit": "^6.5"
    },
    "scripts":{
        "test": "./vendor/bin/phpunit; ./vendor/bin/php-cs-fixer fix -v --diff --dry-run;",
        "phpunit": "./vendor/bin/phpunit;",
        "phpcs": "./vendor/bin/php-cs-fixer fix -v --diff --dry-run;"
    }
}
```

**将代码提交到github** 我们将提交到github，然后发布到packagist就可以了

```bash
git init
git remote add origin git@github.com:chengfengGit/demo.git
vim gitignore #忽略verdor文件和idea文件如.idea
git add -A
git commit -m "+ demo"
git push origin master -u
```

**发布到packagist** 访问Packagist主页，没有账号的话可以先注册一个账号，登录进去，然后点击右上角的Submit，然后填入我们创建的仓库的地址，点击Check，确认没问题后，再点击Submit。 这样，我们的项目就提交成功了，在packagist搜所chengfeng也能看到chengfeng/demo这个项目了。Congratulations！！！ 至此，我们的包已经发布到线上了，我们可以通过

```bash
composer require chengfeng/demo dev-master
```

来安装我们的依赖。（注意，我们之前设置了国内的源，这里可能会有一定的延时） 好像一切到此就完了，但是我们会发现一些问题，在提交完，有一个提示，说不能自动更新。解决这个问题也简单，我们在github添加一个packagist的service就好了，packagist的官网也给出了[详细步骤](https://packagist.org/about#how-to-update-packages) 按照说明一步一步就可以非常轻松的搞定了。 还有一个问题就是，我们在packagist看到chengfeng/demo只有一个dev-master的版本，这不符合我们的要求啊，别人的包都有一个一个的版本的。其实就是因为我们只有一个分支master，对应的composer包就是dev-master版本，composer包的版本是来自于git的分支和tag，分支代表dev版本(除master外)，tag代表stable版本。我们在提交一个版本的分支（如v1.0.0）这样就有了一个新的版本

```bash
git checkout -b v1.0.0
git push origin v1.0.0
```

我们推送分之到远程后，刷新packagist网站上我们项目的地址就可以看到多了一个新的版本v1.0.0-dev。还是不够，我们并没有一个稳定版本，大部分线上的项目都是要求稳定版本的，我们的包则无法安装。我们通过git添加一个tag发布一个稳定版本

```bash
git tag 1.0.0
git push --tag
```

再刷新页面就看到了。 ![](https://blog-1252719385.cos.ap-guangzhou.myqcloud.com/images/20180208194939.png) 现在，我们的包就正式开发完成了。 可以通过`composer require chengfeng/demo`安装了。 关于源代码，可以在[https://github.com/chengfengGit/demo](https://github.com/chengfengGit/demo)下载