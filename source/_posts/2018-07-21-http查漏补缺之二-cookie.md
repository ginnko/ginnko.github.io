---
layout: post
title: http查漏补缺之二 《cookie》
date: 2018-07-21
tag:
- http
- cookie
---

[mdn地址](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies)

1. set-cookie响应头部

    服务器使用Set-Cookie响应头部向用户代理（一般是浏览器）发送Cookie信息。

    写法：

    ```
    Set-Cookie: <cookie名>=<cookie值>
    ```
<!-- more -->

2. cookie请求头部

    现在，对该服务器发起的每一次新请求，浏览器都会将之前保存的Cookie信息通过Cookie请求头部再发送给服务器。

3. 会话期cookie

    会话期Cookie是最简单的Cookie：浏览器关闭之后它会被自动删除，也就是说它仅在会话期内有效。会话期Cookie不需要指定过期时间（Expires）或者有效期（Max-Age）。

4. 持久性cookie

    和关闭浏览器便失效的会话期Cookie不同，持久性Cookie可以指定一个特定的过期时间（Expires）或有效期（Max-Age）。

    **当Cookie的过期时间被设定时，设定的日期和时间只与客户端相关，而不是服务端。**

5. secure标记

    cookie只能通过https协议发送。

6. httpOnly标记

    cookie无法被`document.cookie`获取，只能发送给服务端，可以避免跨域脚本攻击（XSS）。

7. Cookie的作用域

    domain和path这两个标识符定义了Cookie的作用域。

    Domain 标识指定了哪些主机可以接受Cookie。如果不指定，默认为当前文档的主机（不包含子域名）。如果指定了Domain，则一般包含子域名。

    > 例如，如果设置 Domain=mozilla.org，则Cookie也包含在子域名中（如developer.mozilla.org）。

    Path 标识指定了主机下的哪些路径可以接受Cookie（该URL路径必须存在于请求URL中）。以字符 %x2F ("/") 作为路径分隔符，子路径也会被匹配。

    > 例如，设置 Path=/docs，则以下地址都会匹配：  
    /docs  
    /docs/Web/  
    /docs/Web/HTTP

8. 第三方cookie

    如果Cookie的域和页面的域不同，则称之为第三方Cookie；

    如果Cookie的域和页面的域相同，那么我们称这个Cookie为第一方Cookie。

9. 涉及cookie的安全问题

    1. xss（跨站攻击）

        在Web应用中，Cookie常用来标记用户或授权会话。因此，如果Web应用的Cookie被窃取，可能导致授权用户的会话受到攻击。常用的窃取Cookie的方法有利用社会工程学攻击和利用应用程序漏洞进行XSS攻击。

    2. csrf（跨站请求伪造）

        比如在不安全聊天室或论坛上的一张图片，它实际上是一个给你银行服务器发送提现的请求。当你打开含有了这张图片的HTML页面时，如果你之前已经登录了你的银行帐号并且Cookie仍然有效（还没有其它验证步骤），你银行里的钱很可能会被自动转走。