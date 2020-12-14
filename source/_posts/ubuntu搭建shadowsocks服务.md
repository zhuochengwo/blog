---
title: Ubuntu搭建shadowsocks服务
tags:
  - shadowsocks
  - Ubuntu
id: '146'
categories:
  - - Ubuntu
date: 2018-05-25 11:15:58
---

试用的 `aws` 到期了，这两天准备迁移到 `gcp` 了，不得不说，`gcp` 用起来还是比 `aws` 舒服多了，选了`ubuntu 18.04` 的台湾机房服务器，当然第一件事就是 `shadowsocks` 了。 搭建很简单，几条命名搞定：

```bash
sudo apt-get update

# 如果还没有安装python的话先安装python，Ubuntu默认已经安装了
sudo apt-get install python

sudo apt-get install python-pip

sudo pip install shadowsocks
```

编辑一个`ss`账号的配置文件：

```bash
vim /etc/shadowsocks.json
```

```json
{
    "server":"server_ip",
    "server_port":8388,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"mypassword",
    "timeout":300,
    "method":"aes-256-cfb"
}
```

上述配置中的`server_ip`可以填入`0.0.0.0` 或是服务器内网ip 如果需要配置多个账号的话可以这样：

```json
{
    "server":"server_ip",
    "port_password":{
        "9001":"pwd001",
        "9002":"pwd002",
        "9003":"pwd003"
     },
    "local_address":"127.0.0.1",
    "local_port":1080,
    "timeout":300,
    "method":"aes-256-cfb"
}
```

启动服务

```bash
sudo ssserver -c /etc/shadowsocks.json -d start
```

本来一切到这里就结束了，但在`18.04上`报了个错，原因是`openssl`版本的问题。

```bash
$ sudo ssserver -c /etc/shadowsocks.json -d start
INFO: loading config from /etc/shadowsocks.json
2018-05-25 02:32:18 INFO     loading libcrypto from libcrypto.so.1.1
Traceback (most recent call last):
  File "/usr/local/bin/ssserver", line 11, in <module>
    load_entry_point('shadowsocks==2.8.2', 'console_scripts', 'ssserver')()
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/server.py", line 34, in main
    config = shell.get_config(False)
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/shell.py", line 262, in get_config
    check_config(config, is_local)
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/shell.py", line 124, in check_config
    encrypt.try_cipher(config['password'], config['method'])
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/encrypt.py", line 44, in try_cipher
    Encryptor(key, method)
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/encrypt.py", line 83, in __init__
    random_string(self._method_info[1]))
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/encrypt.py", line 109, in get_cipher
    return m[2](method, key, iv, op)
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/openssl.py", line 76, in __init__
    load_openssl()
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/openssl.py", line 52, in load_openssl
    libcrypto.EVP_CIPHER_CTX_cleanup.argtypes = (c_void_p,)
  File "/usr/lib/python2.7/ctypes/__init__.py", line 379, in __getattr__
    func = self.__getitem__(name)
  File "/usr/lib/python2.7/ctypes/__init__.py", line 384, in __getitem__
    func = self._FuncPtr((name_or_ordinal, self))
AttributeError: /usr/lib/x86_64-linux-gnu/libcrypto.so.1.1: undefined symbol: EVP_CIPHER_CTX_cleanup
```

官方其实已经解决了这个问题，但是我们通过pip安装的shadowsocks并不是最新版，更新一下就好

```bash
sudo pip install -U git+https://github.com/shadowsocks/shadowsocks.git@master
```

之前装的是`2.8.2`的版本，更新后现在是`3.0.0`的。再启动就正常了 设置一下防火墙，下边是我的设置，我开的是`1194`端口，根据自己的端口修改即可 ![](https://blog-1252719385.cos.ap-guangzhou.myqcloud.com/images/20180525133441.png) `shadowsocks`的日志在

```bash
/var/log/shadowsocks.log
```

可以看到被代理的相关信息。 剩下的就是在自己的客户端配置上新的ss咯！