---
title: Twitter第三方登录(web前后端分离)
tags:
  - 第三方登录
id: '353'
categories:
  - - 敲代码
date: 2019-01-30 19:12:47
---

Twitter的第三方登录实现相对来说比较糟心，我自己也是翻遍了google前几页的搜索结果，在这里整理一个比较简单的方法实现，希望能对大家有所帮助。

## 申请开发者账户并创建app

Twitter的审核还是很麻烦的，申请api的时候需要写一篇小作文，300字左右，创建app还得再写100字左右的描述。 因为我自己的Twitter审核没有通过，这里就不做过多的描述了。 [app配置](https://developer.twitter.com/en/apps)主要注意回调地址和`Consumer API keys`这些内容

## 梳理

Twitter第三方登录[官方资料](https://developer.twitter.com/en/docs/twitter-for-websites/log-in-with-twitter/login-in-with-twitter)就在这里了，说实话感觉写的比较烂，不过也需要大致读一些，了解下整个流程。 总结来说就是以下几步： 1.通过[oauth/request\_token](https://developer.twitter.com/en/docs/basics/authentication/api-reference/request_token)接口获取`request_token`，请求参数中的`oauth_callback`需要与创建App时填写的`Callback URL`匹配，`request_token`中会返回`oauth_token`和`oauth_token_secret`，需要保存其中的`oauth_token_secret`备用。 2.通过得到的`request_token`中的`oauth_token`来组装授权链接`https://api.twitter.com/oauth/authenticate?oauth_token=上一步得到的oauth_token`，跳转并引导用户授权，[参考](https://developer.twitter.com/en/docs/basics/authentication/api-reference/authenticate) 3.用户授权后会回调到`oauth_callback`填写的地址，并带上两个重要参数`oauth_token`,`oauth_verifier` 4.通过[oauth/access\_token](https://developer.twitter.com/en/docs/basics/authentication/api-reference/access_token)接口换取`access_token`，这个接口需要第一步获取的`request_token`中的`oauth_token_secret`，以及第三步中的`oauth_verifier`，得到`oauth_token`和`oauth_token_secret` 5.将第4步获取的`oauth_token`和`oauth_token_secret`提交后端接口验证，后端接口通过[account/verify\_credentials](https://developer.twitter.com/en/docs/accounts-and-users/manage-account-settings/api-reference/get-account-verify_credentials.html)验证token并获取用户信息

## 前端实现

前端代码实现上有一个挺好用的库[codebird-js](https://github.com/jublo/codebird-js)，可以直接拿来用,上边涉及到的步骤都有实现 获取`request_token`并跳转到Twitter的授权页面

```javascript
<script type="text/javascript" src="./codebird.js"></script>

<script type="text/javascript">
    var cb = new Codebird();
    cb.setConsumerKey("consumer_key", "consumer_secret");
    cb.__call("oauth_requestToken", { oauth_callback: "回调地址" }, function(
        reply,
        rate,
        err
    ) {
        if (err) {
            console.log("error response or timeout exceeded" + err.error);
        }
        if (reply) {
            if (reply.errors && reply.errors["415"]) {
                // check your callback URL
                console.log(reply.errors["415"]);
                return;
            }
            // stores the token
            cb.setToken(reply.oauth_token, reply.oauth_token_secret);
            // gets the authorize screen URL
            cb.__call("oauth_authorize", {}, function(auth_url) {
                window.codebird_auth = window.open(auth_url);
            });
        }
    });

</script>
```

用户授权后会跳转到回调页面，get参数带上`oauth_token`,`oauth_verifier` 需要在这个页面获取参数完成后续步骤

```javascript
var cb = new Codebird();
    var current_url = location.toString();
    var query = current_url.match(/\?(.+)$/).split("&");
    var parameters = {};
    var parameter;

    cb.setConsumerKey("consumer_key", "consumer_secret");

    for (var i = 0; i < query.length; i++) {
        parameter = query[i].split("=");
        if (parameter.length === 1) {
            parameter[1] = "";
        }
        parameters[decodeURIComponent(parameter[0])] = decodeURIComponent(
            parameter[1]
        );
    }

    // check if oauth_verifier is set
    if (typeof parameters.oauth_verifier !== "undefined") {
        cb.setToken(
            stored_somewhere.oauth_token,
            stored_somewhere.oauth_token_secret
        );

        cb.__call(
            "oauth_accessToken",
            {
                oauth_verifier: parameters.oauth_verifier
            },
            function (reply, rate, err) {
                if (err) {
                    console.log("error response or timeout exceeded" + err.error);
                }
                if (reply) {
                    cb.setToken(reply.oauth_token, reply.oauth_token_secret);
                }
            }
        );
    }
```

后续只需要将`oauth_token`和`oauth_token_secret`提交到后端验证即可

## 后端代码（PHP）

须先安装第三方库

```php
composer require abraham/twitteroauth
```

对前端提交的`oauth_token`和`oauth_token_secret`验证 以下是`laravel`框架实现代码

```php
$oauth_token = request()->get('oauth_token');
$oauth_token_secret = request()->get('oauth_token_secret');

try {
    $client = new TwitterOAuth('consumer_key','consumer_secret');
    $client->setOauthToken($oauth_token, $oauth_token_secret);
    $response = $client->get("account/verify_credentials");
} catch (\Exception $e) {
    dd($e->getMessage());
}
dd($response);
//response获取到的就是用户信息，可以自行处理后续逻辑
```

至此Twitter第三方登录前后端分离的实现就基本完成了。 同样的`Android`和`iOS`都有同样的实现，后端可以提供统一的接口验证token就可以完成整个流程了

## 注意

需要注意的是，这里创建的app的`consumer_key`和`consumer_secret`都是暴露出去的，`codebird-js`这个库的作者是建议将app的设置成`read-only`

> Because the Consumer Key and Token Secret are available in the code, it is important that you configure your app as read-only at Twitter, unless you are sure to know what you are doing.

`Android`和`iOS`官方包的实现上也都是两个都需要的，也就是说Twitter官方肯定已经考虑到泄露的问题，从安全性上来说应该是没有问题的，毕竟回调地址是Twitter后台设定的。 相关文章： [Google第三方登录](https://chengfeng.site/2019/01/29/google%E7%AC%AC%E4%B8%89%E6%96%B9%E7%99%BB%E5%BD%95%E5%AE%9E%E7%8E%B0web%E5%89%8D%E5%90%8E%E7%AB%AF%E5%88%86%E7%A6%BB/) [Facebook第三方登录](https://chengfeng.site/2019/01/31/facebook%E7%AC%AC%E4%B8%89%E6%96%B9%E7%99%BB%E5%BD%95web%E5%89%8D%E5%90%8E%E7%AB%AF%E5%88%86%E7%A6%BB/)