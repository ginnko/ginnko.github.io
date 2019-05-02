---
layout: post
title: javascript查漏补缺之十六 ——《客户端内存》
date: 2018-07-30
tag: javascript
---

[链接地址](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Client-side_web_APIs/Fetching_data)

客户端存储由js的api构成，这些api允许你在客户端存储数据，当需要的时候再获取它们，比如：

  1. 个性化站点

  2. 保存先前站点的活动状态，比如从先前的session中存储购物车中的内容，或者记住用户先前是否已经登录

  3. 保存数据或是assets到本地，以便离线使用或者同样的网站再次加载能以更快的速度

  4. 保存网页应用生成的文件到本地以便离线使用

客户端存储在现代浏览器中可以取代cookie诶。

<!-- more -->

1. web storage

  用来存储简单的小型数据。web storage包含两种结构，一种是sessionStorage，一种是localStorage。

  - sessionStorage：数据存在的时长是浏览器打开的这段时期，关闭浏览器则数据丢失；

  - localStorage：使用这种类型保存数据，即使浏览器关闭再打开数据也依然存在。**localStorage是按domain分隔的！！！**

      1. 存储数据：`localStorage.setItem('name', 'bobo')`

      2. 拿到数据：

          ```javascript
          var myName = localStorage.getItem('name');
          myName
          ```
      
      3. 移除数据：`localStorage.removeItem('name');`

2. Indexed DB

  用来存储复杂的大型数据。

  一个简单的使用案例：

  1. 创建一个变量用来存储数据库

    ```javascript
    let db；
    ```
  
  2. 创建IDB是异步的操作，所以一般写在`window.onload`的回调函数中

    ```javascript
    window.onload = function() {

    }
    ```
  
  3. 创建名叫`notes`的第`1`个版本的数据库

    ```javascript
    window.onload = function() {
      let request = window.indexedDB.open('notes', 1);
    }
    ```

  4. handle创建结果

    ```javascript
    window.onload = function() {
      let request = window.indexedDB.open('notes', 1);

      request.onerror = function() {
        console.log('Database failed to open');
      };

      request.onsuccess = function() {
        console.log('Database opened successfully');

        db = request.result;

        displayData();
      };
    }
    ```

DB说IDB暂且用不到，搁置。