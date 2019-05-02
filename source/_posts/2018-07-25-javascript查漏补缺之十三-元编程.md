---
layout: post
title: javascript查漏补缺之十三 ——《元编程》
date: 2018-07-25
tag: javascript
---

[链接地址](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Meta_programming)

<!-- more -->

1. [代理对象](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)

    `let p = new Proxy(target, handler);`

    - target表示用Proxy包装的目标对象（可以是任何类型的对象，包括原生数组，函数，甚至另一个代理）。

    - handler表示一个对象，其属性是当执行一个操作时定义代理的行为的函数。（handler的属性键都是对象的原生函数？感觉是哦）

    1. 使用`get`关键字，当对象中不存在属性名时，返回缺省值

        ```javascript
        let handler = {
          get: function(target, name) {
            return name in target ? target[name] : 37;
          }
        };

        let  p = Proxy({}, handler);

        p.a = 1;
        p.b = undefined;

        console.log(p.a, p.b); // 1, undefined

        console.log('c' in p, p.c); // false, 37
        ```

    2. 无操作转发代理

        使用一个原生的js对象，代理会将所有应用到它的操作转发到这个对象上。

        ```javascript
        let target = {};
        let p = new Proxy(target, {});

        p.a = 37; // 操作转发到目标

        console.log(target.a);  // 37  操作已经被正确转发
        ```
    
    3. 验证

        通过代理验证向一个对象传的值，这个例子使用`set`。

        ```javascript
        let validator = {
          set: function(obj, prop, value) {
            if (prop === 'age') {
              if (!Number.isInteger(value)) {
                throw new TypeError('The age is not an integer');
              }
              if (value > 200) {
                throw new RangeError('The age seems invalid');
              }
            }

            obj[prop] = value;
          }
        };

        let person = new Proxy({}, validator);

        person.age = 100;

        console.log(person.age) // 100

        person.age = 'young'; // Uncaught TypeError: The age is not an integer

        page.age = 300; // Uncaught RangeError: The age seems invalid
        ```

    4. 撤销proxy

        Proxy.revocable()方法被用来创建可撤销的Proxy对象。这意味着代理可以通过revoke函数来撤销并且关闭代理。此后，代理上的任意的操作都会导致TypeError。

        ```javascript
        var revocable = Proxy.revocable({}, {
          get: function(target, name) {
            return "[[" + name + "]]";
          }
        });
        var proxy = revocable.proxy;
        console.log(proxy.foo); // "[[foo]]"

        revocable.revoke();

        console.log(proxy.foo); // TypeError is thrown
        proxy.foo = 1           // TypeError again
        delete proxy.foo;       // still TypeError
        typeof proxy            // "object", typeof doesn't trigger any trap
        ```
2. 反射

    这个mdn讲的很粗略，日后看下阮老师的[书](http://es6.ruanyifeng.com/#docs/reflect)。