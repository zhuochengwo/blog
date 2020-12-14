---
title: 简单快速用上https
tags:
  - https
id: '151'
categories:
  - - Ubuntu
  - - 敲代码
date: 2018-06-10 15:41:09
---

相信不少人在浏览网页的时候都曾有过被运营商劫持加上广告的经历，更有甚者账号被盗，造成财产损失的情况。 普通人去购买https证书确实有点划不来，本篇文章主要介绍如何免费的使用https加固我们的站点。 我们要使用的就是大名鼎鼎的[Let's Encrypt](https://letsencrypt.org/) ，[Let's Encrypt](https://letsencrypt.org/) 是一个免费、开放，自动化的证书颁发机构，由 ISRG（Internet Security Research Group）运作。其官方推荐的自动化工具是[Certbot](https://certbot.eff.org/) 也就是我要介绍的工具。 certbot使用非常简单，进入其官网后，选择web-server程序，以及系统版本，就会安装提示，下边以ubuntu-nginx作为演示： ![certbot选择界面](https://blog-1252719385.cos.ap-guangzhou.myqcloud.com/images/20180610150801.png)  在下方可以看到安装提示 ![certbot安装提示](https://blog-1252719385.cos.ap-guangzhou.myqcloud.com/images/20180610151442.png) 根据安装提示很容易就可以安装并使用certbot了 **安装：**

```bash
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python-certbot-nginx
```

**使用：**

```bash
sudo certbot --nginx
```

后边就根据提示选择对应的域名操作即可，这里我已本站点为演示：

1.  在列出的nginx域名列表中选择站点序号。选择后certbot会验证域名，并自动保存生产的证书信息
2.  选择是否重定向http到https。certbot会询问是否需要重定向
3.  看到"Congratulations! Your certificate and chain have been saved at:"表示证书安装成功
4.  重启nginx即可

下边是我的操作记录，可供参考：

```bash
$ sudo certbot --nginx
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator nginx, Installer nginx

Which names would you like to activate HTTPS for?
-------------------------------------------------------------------------------
1: chengfeng.site
-------------------------------------------------------------------------------
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel): 1
Obtaining a new certificate
Performing the following challenges:
http-01 challenge for chengfeng.site
Waiting for verification...
Cleaning up challenges
Deploying Certificate to VirtualHost /etc/nginx/sites-enabled/chengfeng.site

Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
-------------------------------------------------------------------------------
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
-------------------------------------------------------------------------------
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 2
Redirecting all traffic on port 80 to ssl in /etc/nginx/sites-enabled/chengfeng.site

-------------------------------------------------------------------------------
Congratulations! You have successfully enabled https://chengfeng.site

You should test your configuration at:
https://www.ssllabs.com/ssltest/analyze.html?d=chengfeng.site
-------------------------------------------------------------------------------

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/chengfeng.site/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/chengfeng.site/privkey.pem
   Your cert will expire on 2018-09-08. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

```

配置成功后，证书有三个月的有效期，三个月后可以通过

```bash
sudo certbot --nginx certonly
```

刷新证书，也可以通过非交互的方式

```bash
sudo certbot renew
```

刷新所有证书。 我们可以设置定时任务刷新证书，也可以开启certbot的邮件提醒功能避免站点异常。 **注意：** 使用https配置完证书后要注意站点中资源是否都是通过https域名获取的，如果有些资源还是http的话记得也改成https，否则在chrome等浏览器中不会出现绿色的安全提示。