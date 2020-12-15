---
title: Facebook第三方登录(web前后端分离)
tags:
  - 第三方登录
id: '364'
categories:
  - - 敲代码
date: 2019-01-31 11:03:40
---

Facebook实现第三方登录，跟google一样，也是比较简单，当然官方文档跟google比就一般般了。 这里整理以**前后端分离**的方式实现Facebook的第三方登录

## 申请账号

可以直接在[app控制面板](https://developers.facebook.com/apps)创建应用，跟google差不多。 在控制面板页面>产品 添加一个`facebook登录`，选择设置进行相关设置即可。 ![设置](https://blog-1252719385.cos.ap-guangzhou.myqcloud.com/images/Selection_025.png "设置")

## 流程

1.前端引导用户授权并获取`access_token` 2.将获取到的`access_token`提交到后端验证并获取用户信息 3.后端成功验证后，获取用户信息完成系统登录或注册流程

## 前端代码

前端直接使用官方的[sdk](https://developers.facebook.com/docs/javascript)就行了，下边的代码也来自官方示例

```javascript
<script>
    // This is called with the results from from FB.getLoginStatus().
    function statusChangeCallback(response) {
        console.log('statusChangeCallback');
        console.log(response);
        // The response object is returned with a status field that lets the
        // app know the current login status of the person.
        // Full docs on the response object can be found in the documentation
        // for FB.getLoginStatus().
        if (response.status === 'connected') {
            // Logged into your app and Facebook.
            testAPI();
        } else {
            // The person is not logged into your app or we are unable to tell.
            document.getElementById('status').innerHTML = 'Please log ' +
                'into this app.';
        }
    }

    // This function is called when someone finishes with the Login
    // Button.  See the onlogin handler attached to it in the sample
    // code below.
    function checkLoginState() {
        FB.getLoginStatus(function(response) {
            console.log(response);
            statusChangeCallback(response);
        });
    }

    window.fbAsyncInit = function() {
        FB.init({
            appId      : 'your appid',
            cookie     : true,  // enable cookies to allow the server to access
                                // the session
            xfbml      : true,  // parse social plugins on this page
            version    : 'v2.8' // use graph api version 2.8
        });


        FB.getLoginStatus(function(response) {
            statusChangeCallback(response);
        });

    };

    // Load the SDK asynchronously
    (function(d, s, id) {
        var js, fjs = d.getElementsByTagName(s)[0];
        if (d.getElementById(id)) return;
        js = d.createElement(s); js.id = id;
        js.src = "https://connect.facebook.net/en_US/sdk.js";
        fjs.parentNode.insertBefore(js, fjs);
    }(document, 'script', 'facebook-jssdk'));

    // Here we run a very simple test of the Graph API after login is
    // successful.  See statusChangeCallback() for when this call is made.
    function testAPI() {
        console.log('Welcome!  Fetching your information.... ');
        FB.api('/me', function(response) {
            console.log('Successful login for: ' + response.name);
            document.getElementById('status').innerHTML =
                'Thanks for logging in, ' + response.name + '!';
        });
    }
</script>


<fb:login-button scope="public_profile,email" onlogin="checkLoginState();">
</fb:login-button>

<div id="status">
</div>
```

在`statusChangeCallback`方法中，用户授权后后，在控制台打印出来的`response`中就有我们需要的`access_token`了，通过将`access_token`提交后端验证就可以了。

## 后端代码（PHP）

后端可以通过官方包来完成整个验证流程，先验证`access_token`，在通过`access_token`获取用户信息。 安装Facebook官方包

```bash
composer require facebook/graph-sdk
```

以下是laravel框架实现代码

```php
$token = request()->get('token');
try {
    $client = new Facebook([
        'app_id'     => 'YOUR APP_ID',
        'app_secret' => 'YOUR APP_SECRET',
    ]);
    $oAuth2Client = $client->getOAuth2Client();

    $tokenMetadata = $oAuth2Client->debugToken($token);
    $tokenMetadata->validateAppId('YOUR APP_ID');
    $tokenMetadata->validateExpiration();

    $res = $client->sendRequest('GET', '/me',
                                ['fields' => 'id,name,birthday,gender,hometown,email,picture,devices'], $token);
    dd($res->getGraphUser());
} catch (\Exception $e) {
    dd($e->getMessage());
}
```

`$res->getGraphUser()` 就会获取到所需的用户信息，这里的信息可以在接口参数中指定，具体可以查看相关[接口文档](https://developers.facebook.com/docs/graph-api/reference/v2.6/user) 到这里，整个流程基本上就处理完了。