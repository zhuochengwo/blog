---
title: Google第三方登录实现(web前后端分离)
tags:
  - 第三方登录
id: '357'
categories:
  - - 敲代码
date: 2019-01-29 16:31:48
---

关于google第三方登录的实现，google官方有非常完善的文档，这里主要是整理，以一种最简单的，**前后端分离**的方式实现。

## 创建项目

在[创建项目](https://console.developers.google.com/projectcreate)填入`Project Name`就可以非常简单的完成项目的创建

## 创建`OAuth Client ID`

在[控制面板](https://console.developers.google.com/apis/credentials)可以创建和查看App的相关信息 新创建的App需要在这里生成一个`OAuth Client ID`，如果App还没配置，创建的时候可能会提示配置`OAuth consent screen`，点击配置，在配置页面填写下`Application name`保存即可。 ![生成OAuth Client ID](https://blog-1252719385.cos.ap-guangzhou.myqcloud.com/images/Selection_022.png "生成OAuth Client ID") 创建的时候选择`Web application`,注意填写`Authorized JavaScript origins`,就是前端项目的域名，回调地址可以先不填。 ![生成OAuth Client ID](https://blog-1252719385.cos.ap-guangzhou.myqcloud.com/images/Selection_023.png "生成OAuth Client ID") 这样我们就有了需要的`Client ID`和`Client Secret`。

## 获取`id_token`

### 前端代码

修改`meta`中的`YOUR_CLIENT_ID`为申请的应用的`Client ID`，这样就可以获取到我们需要的`id_token`了，将`id_token`以请求参数传递给后端验证就可以完成整个登录流程了。如果需要的话，我们也可以直接在前端获取到`access_token`。下边代码会打印`id_token`和`access_token`，可以在控制台查看。

```markup
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <script src="https://apis.google.com/js/platform.js" async defer></script>
    <meta name="google-signin-client_id" content="YOUR_CLIENT_ID.apps.googleusercontent.com">
    <title>test google</title>
</head>
<body>
<div class="g-signin2" data-onsuccess="onSignIn"></div>
</body>
<script>
    function onSignIn(googleUser) {
        console.log(googleUser.getAuthResponse().id_token);
        console.log(googleUser.getAuthResponse(true).access_token);
    }
</script>
</html>
```

### 后端代码（PHP）

后端提供接口，使用`google`提供的api验证`id_token`就可以了 先安装官方包

```php
composer require google/apiclient
```

```php
$client = new Google_Client(['client_id' => $CLIENT_ID]);
$payload = $client->verifyIdToken($id_token);
if ($payload) {
//验证成功
  $userid = $payload['sub'];
} else {
  //验证失败
}
```

其它语言实现可参考[官方文档](https://developers.google.com/identity/sign-in/web/backend-auth) 这样就非常简单的完成了google第三方登录的流程。 如果还有其它不清楚的地方，可以查看[官方文档](https://developers.google.com/identity/sign-in/web/backend-auth)的说明，相对来说，google的文档还是比较简单清晰的。 相关文章： [Facebook第三方登录](https://chengfeng.site/2019/01/31/facebook%E7%AC%AC%E4%B8%89%E6%96%B9%E7%99%BB%E5%BD%95web%E5%89%8D%E5%90%8E%E7%AB%AF%E5%88%86%E7%A6%BB/) [Twitter第三方登录](https://chengfeng.site/2019/01/30/twitter%E7%AC%AC%E4%B8%89%E6%96%B9%E7%99%BB%E5%BD%95web%E5%89%8D%E5%90%8E%E7%AB%AF%E5%88%86%E7%A6%BB/)