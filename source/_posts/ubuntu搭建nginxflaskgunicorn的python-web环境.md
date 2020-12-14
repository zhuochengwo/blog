---
title: Ubuntu搭建nginx+flask+gunicorn的Python web环境
tags:
  - Python
id: '200'
categories:
  - - Python
  - - 敲代码
date: 2018-07-11 11:32:54
---

最近开始接触Python的web开发，选用了flask框架，这里记录下python web环境的搭建。 现在大部分web语言都有提供自带的web server功能用于开发调试，在开发阶段，我们可以通过flask run启用一个简单的web server，但这样的web server性能及稳定性欠佳，不建议用于生产环境。 我们先通过

```bash
virtualenv -p /usr/bin/python3 --no-site-packages venv
```

创建一个虚拟环境，如果没有安装virtualenv请先安装，这里不再介绍virtualenv的安装。通过git pull或者其它方式将开发好的项目代码拉取过来，进入virtual环境并安装项目依赖，如果之前有将依赖导出到requirements.txt则可以非常快捷简单的安装了。

```bash
source venv/bin/activate

pip install -r requirements.txt
#或者直接安装flask和gunicorn
pip install flask
pip install gunicorn
```

这里以一个最简单的hello world为例

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'
```

运行gunicorn

```bash
gunicorn --workers=2 hello:app -b 127.0.0.1:9000 -D
```

关于gunicorn这里不做过多展开，只介绍几个常用参数，具体详细介绍可google。

*   **\--workers**   worker进程数
*   **hello:app**   模块名：变量名
*   **\-b**  绑定ip及端口，如果需要外网访问ip请使用0.0.0.0，因为我们后边会用到nginx做反向代理，这里就绑定127.0.0.1
*   **\-D** 守护进程

启动gunicorn后就可以通过http://127.0.0.1:9000 看到输出的hello world了。 如果我们有修改代码，需要重启gunicorn才能生效，可以使用

```bash
kill -HUP 进程号
```

重启，此处的进程号为master进程的pid。 接下来配置nginx。如果没有安装nginx可以通过下边命令安装

```bash
sudo apt-get install nginx
```

在/etc/nginx/site-enable下创建hello.flask.com文件，写入如下内容

```nginx
server {
        listen 80;
        server_name hello.flask.com;

        location / {
                proxy_pass http://127.0.0.1:9000;
                proxy_redirect     off;
                proxy_set_header   Host                 $http_host;
                proxy_set_header   X-Real-IP            $remote_addr;
                proxy_set_header   X-Forwarded-For      $proxy_add_x_forwarded_for;
                proxy_set_header   X-Forwarded-Proto    $scheme;
        }

}
```

通过nginx -t测试配置是否正确，以免重启服务影响已有站点。测试通过重新载入配置。

```bash
sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
$ sudo service nginx reload
```

配置/etc/hosts，写入

```vim
127.0.0.1 hello.flask.com
```

访问http://hello.flask.com能看到正常输出hello world，至此flask+gunicorn部署成功。