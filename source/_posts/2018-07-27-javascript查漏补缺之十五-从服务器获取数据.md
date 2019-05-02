---
layout: post
title: javascript查漏补缺之十五 ——《从服务器获取数据》
date: 2018-07-27
tag: javascript
---

[链接地址](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Client-side_web_APIs/Fetching_data)

1. XMLHttpRequest

    使用步骤

    1. 使用`XMLHttpRequest()`构造函数创建一个新的请求对象

        ```javascript
        var request = new XMLHttpRequest();
        ```

    2. 使用`open()`方法指定用于从网络请求资源的**方法**以及**url**

        ```javascript
        request.open('GET', url);
        ```

    3. 设置响应类型（XHR默认返回文本）

        ```javascript
        request.responseType = 'text';
        ```

    4. XHR使用`onload`事件处理来处理响应

        ```javascript
        request.onload = function() {
            poemDisplay.textContent = request.response;
        }
        ```
    5. 发出请求

        ```javascript
        request.send()
        ```

2. fetch

    1. 速览
        使用fetch替换上面的XHR的代码：

        ```javascript
        fetch(url).then(function(response) {
            response.text().then(function(text) {
                poemDisplay.textContent = text;
            });
        })
        ```

        fetch返回一个`promise`对象！在fetch后面的`then（）`函数将解析从服务器返回的响应。

        当fetch（）的promise解析时，这个函数会自动将响应从服务器传递给参数。在函数内部，我们获取响应并运行其`text()`方法。这将响应作为原始文本返回。`text()`返回一个promise对象，所以我们连接另外一个`.then()`在她上面，在其中我们定义了一个函数来接收text()promise解析的生文本。

        当使用链式写法：

        ```javascript
        fetch(url).then(function(response) {
        return response.text()
        }).then(function(text) {
        poemDisplay.textContent = text;
        });
        ```

        注意上面前一个then函数的回调函数中有一个`return`。

    2. 使用fetch

        1. 注意事项

            1. 当接收到一个代表错误的http请求状态码时，从`fetch()`返回的promise**不会被标记为reject**，即使该http响应的状态码是404或500。相反，它会将promise状态标记为resolve（但是会将resolve的返回值ok属性设置为false），仅当网络故障时或请求被阻止时，才会标记为reject。

            2. 默认情况下，fetch不会从服务端发送或接收任何cookies，如果站点依赖于用户的session，则会导致未经认证的请求（要发送cookies，必须设置`credentials`选项）。


