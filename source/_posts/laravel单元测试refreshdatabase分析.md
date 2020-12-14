---
title: laravel单元测试RefreshDatabase分析
tags: []
id: '516'
categories:
  - - 敲代码
date: 2019-08-14 22:12:33
---

在编写单元测试时，我们或多或少会使用到数据库，一方面我们需要一个干净的数据库，另一方面我们也需要能方便的回滚/清理掉测试产生的数据。

## 使用

在`laravel`单元测试中，我们可以创建`.env.testing`，配置测试需要的环境参数。如单独配置一个测试库，在执行`phpunit`时，`laravel`会自动切换到这个配置，如果没有则会使用`.env`的配置。 `laravel`有提供用于清理数据库中数据的`trait`，在单元测试的`class`中就能直接使用

```php
use RefreshDatabase;
```

*   [DatabaseMigration](https://laravel.com/api/5.8/Illuminate/Foundation/Testing/DatabaseMigrations.html)
*   [DatabaseTransactions](https://laravel.com/api/5.8/Illuminate/Foundation/Testing/DatabaseTransactions.html)
*   [RefreshDatabase](https://laravel.com/api/5.8/Illuminate/Foundation/Testing/RefreshDatabase.html)

## 分析

这几个`trait`各有优缺点，需要看自己的需要选择。 `DatabaseMigration`比较简单粗暴，在每个单元测试开始时`migrate:fresh`创建库表结构，结束时直接使用`migrate:rollback`回滚了整个测试库，优点是真的干净，缺点就是慢。 `DatabaseTransactions` 跟 `RefreshDatabase` 差别不大，都是采用事务回滚的方式，只是`RefreshDatabase`能提供内存型数据库的支持，本文也主要是分析`RefreshDatabase`的执行过程。

#### 为什么在`tes`t类`use`相关`Trait`就能发生作用呢？

追踪`TestCase`源码，发现在基类[TestCase](https://laravel.com/api/5.8/Illuminate/Foundation/Testing/TestCase.html)中`setUpTraits`方法会检查框架预定义好的一系列`Trait`,其中就有我们需要的`RefreshDatabase`

```php
protected function setUpTraits()
    {
        $uses = array_flip(class_uses_recursive(static::class));

        if (isset($uses[RefreshDatabase::class])) {
            $this->refreshDatabase();
        }

        if (isset($uses[DatabaseMigrations::class])) {
            $this->runDatabaseMigrations();
        }

        // ....
    }
```

此段代码在检测到`RefreshDatabase`后会调用其`refreshDatabase`方法，在此方法中会调用自己的相关方法，检查`migrate`是否执行，否则会执行`migrate:fresh`。然后会调用`beforeApplicationDestroyed`方法注册一系列回调方法，当中就有最重要的`rollback`。

#### `RefreshDatabase`具体执行了哪些`sql`呢？

通过`laravel`通过的事件我们是不能获取到如`START TRANSACTION`等相关语句是不会出现的。 我们开启`MySQL`的`sql`日志

```
SET GLOBAL general_log = 'ON';
```

执行`phpunit`，通过日志可以看到会先执行`migrate`相关内容，就然后就通过`START TRANSACTION`开启一个事务，在测试结束后`ROLLBACK`。 如果执行的需要测试的代码有使用事务，则会通过`SAVEPOINT` 给子事务标记，如果需要回滚则只会滚到标记点即可。对于其中事务的提交，则因`laravel`事务处理机制问题，并不会提交，只是标记。 与`aa`的区别就是，不会在每一个测试开始是都会执行`migrate`，原因是在第一次执行后，其通过`RefreshDatabaseState::$migrated`标记已执行`migrate`，后续的`test`虽然会执行其`refreshDatabase`方法，但由于能检测到已经执行就不会再次执行了。

```php
protected function refreshTestDatabase()
    {
        //先检查，没有执行migrate则会执行
        if (! RefreshDatabaseState::$migrated) {
            $this->artisan('migrate:fresh', [
                '--drop-views' => $this->shouldDropViews(),
                '--drop-types' => $this->shouldDropTypes(),
            ]);

            $this->app[Kernel::class]->setArtisan(null);

            RefreshDatabaseState::$migrated = true;
        }

        $this->beginDatabaseTransaction();
    }
```

而后则只会开启事务

```php
    /**
     * Begin a database transaction on the testing database.
     *
     * @return void
     */
    public function beginDatabaseTransaction()
    {
        $database = $this->app->make('db');

        foreach ($this->connectionsToTransact() as $name) {
            $connection = $database->connection($name);
            $dispatcher = $connection->getEventDispatcher();

            $connection->unsetEventDispatcher();
            $connection->beginTransaction();
            $connection->setEventDispatcher($dispatcher);
        }

        $this->beforeApplicationDestroyed(function () use ($database) {
            foreach ($this->connectionsToTransact() as $name) {
                $connection = $database->connection($name);
                $dispatcher = $connection->getEventDispatcher();

                $connection->unsetEventDispatcher();
                $connection->rollback();
                $connection->setEventDispatcher($dispatcher);
                $connection->disconnect();
            }
        });
    }
```

至此，`RefreshDatabase`的执行流程大致清晰了。

### 注意

需要注意的是，因为使用的是事务的方式，如果需要测试的代码中使用了事务，而忘了使用`COMMIT` ，则是不能在单元测试中发现，因为`laravel`并不会单独对子事务提交。