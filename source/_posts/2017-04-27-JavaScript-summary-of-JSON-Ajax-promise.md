---
layout: post
title: Ajax和Promise的简单理解
date: 2017-04-27
tag: 
- javascript
- Ajax
- Promise
- 异步
---

### 1. Ajax

Ajax技术能够向服务器请求额外的数据而无需卸载页面，会带来更好的用户体验。

    function success(text) {
    var textarea = document.getElementById('test-response-text');
    textarea.value = text;
    }

    function fail(code) {
        var textarea = document.getElementById('test-response-text');
        textarea.value = 'Error code: ' + code;
    }

    var request = new XMLHttpRequest(); // 新建XMLHttpRequest对象

    request.onreadystatechange = function () { // 状态发生变化时，函数被回调
        if (request.readyState === 4) { // 成功完成
            // 判断响应结果:
            if (request.status === 200) {
                // 成功，通过responseText拿到响应的文本:
                return success(request.responseText);
            } else {
                // 失败，根据响应码判断失败原因:
                return fail(request.status);
            }
        } else {
            // HTTP请求还在继续...
        }
    }

    // 发送请求:
    request.open('GET', '/api/categories');
    request.send();

    alert('请求已发送，请等待响应...');

上述代码引自网络[点击这里](http://www.liaoxuefeng.com/wiki/001434446689867b27157e896e74d51a89c25cc8b43bdb3000/001434499861493e7c35be5e0864769a2c06afb4754acc6000)  

Ajax技术基于对象XMLHttpRequest，依赖于4个属性：onreadystatechange、readyState、status、responseText以及2个方法：open()、send()。

- open()方法  
open()方法接受三个参数，如上述代码，**第二个参数只能向同一个域发送请求**。第三个表示是否异步发送请求的布尔值。open()方法启动一个请求准备发送。

- send()方法  
send()方法被调用后，请求会被发送至服务器。

- readyState属性

|属性值|状态|说明|
|:---:|:---:|:---:|
|0|未初始化|尚未调用open()方法|
|1|启动|已经调用open()方法，但尚未调用send()方法|
|2|发送|已经调用send()方法，但尚未接收到相应|
|3|接收|已经接收到部分相应数据|
|4|完成|已经接收到全部响应数据，而且已经可以再客户端使用了|

- onreadystatechange属性  
只要readyState属性的值由一个值变成另一个值，都会触发一次onreadystatechange事件，执行回调函数。

- status属性  
该属性表示响应的HTTP状态。收到响应后，检查status属性，将HTTP状态代码为200作为成功的标志。

- responseText属性  
被返回的文本包含在这个属性里
  
**发送的异步请求能让JavaScript继续执行而不必等待相应。** 

### 2. Promise

**Promise最大的好处是在异步执行的过程中，把执行代码和处理结果的代码清晰的分离了。**

    function test(resolve, reject) {
        var timeOut = Math.random() * 2;
        log('set timeout to: ' + timeOut + ' seconds.');
        setTimeout(function () {
            if (timeOut < 1) {
                log('call resolve()...');
                resolve('200 OK');
            }
            else {
                log('call reject()...');
                reject('timeout in ' + timeOut + ' seconds.');
            }
        }, timeOut * 1000);
    }
    new Promise(test).then(function (result) {
        console.log('成功：' + result);
    }).catch(function (reason) {
        console.log('失败：' + reason);
    });

上述代码引自网络[点击这里](www.liaoxuefeng.com/wiki/001434446689867b27157e896e74d51a89c25cc8b43bdb3000/0014345008539155e93fc16046d4bb7854943814c4f9dc2000)

- test函数相当于定义了一个异步执行的操作，函数内清楚的表明了异步操作成功的结果处理和异步操作失败的结果处理。

- test函数接收了两个参数，两个参数都是函数，在test中并未给出具体定义。

- new Promise(test)定义了一个Promise对象，并把一个名为test的异步操作任务传递给了这个Promise对象。

- Promise对象可以使用两个方法then()和catch()，当调用then()方法时，才真正发出异步请求。then()中定义的函数才是resolve实际代表的函数，当异步操作成功
  执行其中的函数，并返回一个Promise对象；当异步操作失败，执行catch()中定义的函数，这个函数是reject的实际定义，并返回一个Promise对象。





