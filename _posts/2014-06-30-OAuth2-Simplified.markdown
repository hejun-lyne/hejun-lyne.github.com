---
layout: post
title: OAuth 2 简述
category: oauth
url: OAuth2-Simplified.html
summary: 介绍了OAuth2.0的相关知识点和流程
---
*来自[OAuth 2 Simplified](http://aaronparecki.com/articles/2012/07/29/1/oauth2-simplified)*
###角色
####第三方应用：“Client”
Client是指试图访问用户账号的应用，需要先从用户获得允许之后才能这么做。
####API：“资源服务器”
资源服务器就是用来访问用户信息的API服务器。
####User：“资源拥有者”
资源拥有者是那些可以允许访问他们账户的某些信息的人。

###创建一个App
在开始OAuth流程之前，你需要先通过服务注册一个新的App。通常需要提供下面这些信息：应用名称、网站及Logo。另外，你必须注册一个重定行URI用来将用户重定向到Web服务器（基于浏览器或者是移动应用）。
####重定向URI
服务只会重定向用户到已注册的URI，这个可以用于防止某些攻击。任何的HTTP重定向URI必须通过TLS安全保护，所以服务只会重定向到以“https”开头的URI。这可以防止token在认证过程中被拦截。
####Client ID和Secret
注册app之后，你会收到一个客户端ID和客户端Secret。客户端ID认为是公开信息，用于搭建登录URL，或者出现在Javascript源代码中。客户端Secret需要保密保存。如果一个已部署app不能保密保存Secret（如Javascript或Native apps），那么可以不使用Secret。
###认证（Authorization）
OAuth 2的第一个是从用户获取授权。对于基于浏览器和移动的App，通常是通过显示服务提供的界面来完成的。
OAth 2为不同用户场景提供了若干“grant types”，定义如下：
* Authorization Code，在web server中运行的App；
* Implicit，browser-based 或 mobile apps；
* Password，通过username和password登录；
* Client credentials，应用访问。

下面对每一种用户场景进行详细描述。
###Web Server Apps
这是最常遇见的应用类型。Web apps是使用服务端语言编写的，运行在服务器上面，源代码不公开的应用。
####认证
创建一个“Log In”链接引导用户至：
{% highlight js %}
"https://oauth2server.com/auth?response_type=code&
  client_id=CLIENT_ID&redirect_uri=REDIRECT_URI&scope=photos"
{% endhighlight %}
用户可以看到认证的确认框：<br />
![Authorization](http://aaronparecki.com/articles/2012/07/29/1/files/oauth-authorization-prompt.png)<br />
如果用户点击“Allow”，那么服务带着auth code将用户重定向到你的站点
{% highlight js %}
https://oauth2client.com/cb?code=AUTH_CODE_HERE
{% endhighlight %}
你的服务器可以将auth code交换为access token
{% highlight js %}
POST https://api.oauth2server.com/token
    grant_type=authorization_code&
    code=AUTH_CODE_HERE&
    redirect_uri=REDIRECT_URI&
    client_id=CLIENT_ID&
    client_secret=CLIENT_SECRET
{% endhighlight %}
服务器返回access token
{% highlight js %}
{
    "access_token":"RsT5OjbzRn430zqMLgV3Ia"
}
{% endhighlight %}
或者是出错信息
{% highlight js %}
{
    "error":"invalid_request"
}
{% endhighlight %}
安全性：注意服务要求App必须预先注册他们的重定向URI
###Browser-Based Apps
基于浏览器的App是完全在浏览器运行的。因为全部的源代码都是可被浏览器访问的，所以不能维持client secret的保密性，所以secret不被使用。
####认证
创建一个“Log In”链接引导用户至：
{% highlight js %}
"https://oauth2server.com/auth?response_type=token&client_id=CLIENT_ID&redirect_uri=REDIRECT_URI&scope=photos"
{% endhighlight %}
用户可以看到认证的确认框：<br />
![Authorization](http://aaronparecki.com/articles/2012/07/29/1/files/oauth-authorization-prompt.png)<br />
如果用户点击“Allow”，那么服务带着access token将用户重定向到你的站点
{% highlight js %}
https://oauth2client.com/cb#token=ACCESS_TOKEN
{% endhighlight %}
拿到access token之后就可以开始请求API服务了，如果出错那么返回
{% highlight js %}
https://oauth2client.com/cb#error=access_denied
{% endhighlight %}
###Mobile Apps
和browser-based app一样，mobile apps也不能维持client secret的保密性。所以mobile app必须使用不请求client secret的OAuth流程。
####认证
创建“Log in”按钮引导用户至服务的Native app或者网页。在iPhone上，apps可以注册一个自定义的URI协议如“facebook://”启动native的Facebook应用。在Android，apps可以注册符合格式的URL来启动native app。
#####iPhone
如果用户有Native的Fackbook app，引导到下面的URL：
{% highlight js %}
fbauth2://authorize?response_type=token&client_id=CLIENT_ID
  &redirect_uri=REDIRECT_URI&scope=email
{% endhighlight %}
在这种情况下你的重定向URI看起来像这样`fb00000000://authorize`，在这里"fb"后面跟着你app的client ID。这意味着你的app必须使用该protocol进行了注册。
#####Android或其他
可以通过标准的Web认证URL来进行授权。
{% highlight js %}
https://facebook.com/dialog/oauth?response_type=token
  &client_id=CLIENT_ID&redirect_uri=REDIRECT_URI&scope=email
{% endhighlight %}
用户会看到下面的页面：<br />
![Web Authorization](http://aaronparecki.com/articles/2012/07/29/1/files/everyday-city-auth.png)<br />
点击"Okay"之后，用户会通过下面的URL重定向到你自己的应用：
{% highlight js %}
fb00000000://authorize#token=ACCESS_TOKEN
{% endhighlight %}
####其他
#####Password
这种方式允许app收集用户名和密码。应该只用于服务自己创建的App。例如Native的Twitter app。需要使用password grant type的时候使用如下的POST请求：
{% highlight js %}
POST https://api.oauth2server.com/token
    grant_type=password&
    username=USERNAME&
    password=PASSWORD&
    client_id=CLIENT_ID
{% endhighlight %}
服务器返回与其他类型格式相同的access token。注意这里不包含secret，因为大部分情况下都是通过mobile或者desktop app来使用。
#####Application access
某些情况下，应用希望能够更新他们自己的信息（website URL或者application icon），又或者希望获取app用户的统计数据。所以application需要有一种方式可以获取他们自己账号的access token。OAuth提供了client_credentials grant type来满足这个需求。
{% highlight js %}
POST https://api.oauth2server.com/token
    grant_type=client_credentials&
    client_id=CLIENT_ID&
    client_secret=CLIENT_SECRET
{% endhighlight %}
返回结果是access token。
###生成已认证请求
获取到access token之后就可以请求API了。如
{% highlight js %}
curl -H "Authorization: Bearer RsT5OjbzRn430zqMLgV3Ia" \
https://api.oauth2server.com/1/me
{% endhighlight %}