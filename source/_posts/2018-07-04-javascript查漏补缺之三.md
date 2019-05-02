---
layout: post
title: javascript查缺补漏之三 —— 《循环和函数》
date: 2018-07-04
tag: javascript
---

1. break

    1. 当在循环中执行break时，会立即终止当前所在的循环。比如下面这几行代码：

        ```
        for (i = 0; i < 3; i++) {
            for ( j = 0; j < 3; j++) {
                if (i === 0) {
                    console.log('最里面:', i);
                    break;
                }
                console.log('里面：', i);
            }
            console.log('外面：', i);
        }
        console.log('最外面');

        //输出顺序：
            最里面: 0
            外面： 0
            里面： 1
            外面： 1
            里面： 2
            外面： 2
            最外面
        ```

        说明break只跳出了内层循环（所在循环），并没有跳出外面包裹的循环。

    2. break label 语法

        自己从来没有使用过这种语法，但通过下面这个列子感觉这个语法还是蛮方便的：

        ```
        var x = 0;
        var z = 0
        labelCancelLoops: while (true) {
        console.log("外部循环: " + x);
        x += 1;
        z = 1;
        while (true) {
            console.log("内部循环: " + z);
            z += 1;
            if (z === 10 && x === 10) {
            break labelCancelLoops;
            } else if (z === 10) {
            break;
            }
        }
        }

        //如果满足 z === 10 && x === 10 的条件，整个循环就会结束，就是使用了break labelCancelLoops 这条语法。
        ```
2. `for in` vs `for of` vs `for`

    1. for...in 语句循环一个指定的变量来循环一个对象**所有可枚举**的属性。

    2. for...of语句在可迭代的对象上创建了一个循环(包括Array, Map, Set, 参数对象（ arguments） 等等)

    3. 数组适合用传统for循环来操作

    ```
    let arr = [3, 5, 7];
    arr.foo = "hello";

    for (let i in arr) {
    console.log(i); // logs "0", "1", "2", "foo"
    }

    for (let i of arr) {
    console.log(i); // logs "3", "5", "7" // 注意这里没有 hello
    }

    for (let i = 0; i < arr.length; i++) {
    console.log(i); //logs "0", "1", "2"// 注意这里没有 foo
    }
    ```

    上面代码中，传统for循环和for of循环的结果相似。

3. 函数表达式

    函数表达式也可以提供函数名，并且可以用于在函数内部代指其本身，或者在调试器堆栈跟踪中识别该函数。

    ```
    var factorial = function fac(n) {return n<2 ? 1 : n*fac(n-1)};

    console.log(factorial(3));
    ```

    注意下面代码中`square`并非函数名，下面这个函数是匿名函数。

    ```
    var square = function(number) { return number * number; };
    var x = square(4); // x gets the value 16
    ```
4. Arguments对象

    arguments 是一个对应于传递给函数的参数的类数组对象。

    arguments对象是所有**非箭头**函数中都可用的局部变量。可以使用arguments对象在函数中引用函数的参数。

    arguments的元素可读可写。

5. apply()函数

    `func.apply(thisArg, [argsArray])`，其中`[argsArray]`可以为数组也可以是类数组

6. 类数组

    只要有一个 length 属性和[0...length) 范围的整数属性。比如：

    ```
    {'length': 2, '0': 'eat', '1': 'bananas'}
    ```

7. 查找一系列数字中的最大最小值

    1. 使用apply和原生Math.max()、Math.min()方法

        ```
        var numbers = [5, 6, 2, 3, 7];
        var max = Math.max.apply(null, numbers);
        var min = Math.min.apply(null, numbers);
        ```
    2. 使用数组扩展符

        ```
        var arr = [1, 2, 3];
        var max = Math.max(...arr);
        ```

8. 类数组转为数组

    1. 方法1： 对性能有影响
        `Array.prototype.slice.call(arguments)`，这行代码的意思是将slice方法绑定在arguments对象上。

        简写形式:

        `[].slice.call(arguments)`，依然是将slice方法绑定在arguments对象上。

    2. 方法2：

        `const args = Array.from(arguments);`
    
    3. 方法3：使用扩展运算符

        `var args = [...arguments];`

9. `arguments.length` vs `function.length`

    arguments.length：表示实际传入函数的参数的个数；

    function.length: 指该函数有多少个必须要传入的参数，即形参的个数。形参的数量不包括剩余参数个数，仅包括第一个具有默认值之前的参数个数。

10. 剩余参数 vs 默认参数 vs 解构赋值

    1. 剩余参数

        如果函数的最后一个命名参数以...为前缀，则它将组成一个数组。**剩余参数是货真价实的数组而arguments对象只是类数组。**

        ```
        function foo(...args) {
        return args;
        }
        foo(1, 2, 3);//[1, 2, 3]
        ```

    2. 默认参数

        **默认参数是在函数被调用时解析的**，所以如果默认参数是可变动的，则每次的结果可能是不一样的。
    
    3. 解构赋值

        **数组的解构赋值**:目测是按数组元素的顺序。（此处仅记录了数组的解构赋值）。
        
        1. 利用数组的解构赋值可以轻松交换值。

            ```
            let a = 1, b = 3;
            [a, b] = [b, a];
            console.log(a);//3
            console.log(b);//1
            ```
        2. 解析一个从函数返回的数组

            这个自己写代码已经遇到过多次返回数组的情况，但是都使用的传统获取数组对应位置的元素的方法，明显使用数组解构更简洁。

            ```
            function f() {
            return [1, 2];
            }

            var a, b; 
            [a, b] = f(); 
            console.log(a); // 1
            console.log(b); // 2
            ```

        3. 忽略一些值
            ```
            function f() {
            return [1, 2, 3];
            }

            var [a, , b] = f();
            console.log(a); // 1
            console.log(b); // 3
            ```
        4. 将剩余数组赋值给一个变量
            ```
            var [a, ...b] = [1, 2, 3];
            console.log(a); // 1
            console.log(b); // [2, 3]
            ```
        
        5. 用正则表达式匹配提取值

            ```
            var url = "https://developer.mozilla.org/en-US/Web/JavaScript";

            var parsedURL = /^(\w+)\:\/\/([^\/]+)\/(.*)$/.exec(url);
            console.log(parsedURL); // ["https://developer.mozilla.org/en-US/Web/JavaScript", "https", "developer.mozilla.org", "en-US/Web/JavaScript"]

            var [, protocol, fullhost, fullpath] = parsedURL;

            console.log(protocol); // "https"
            ```
11. 函数递归

    递归函数就使用了堆栈：函数堆栈。

    1. 应用1：获取树结构中所有的节点

        ```
        function walkTree(node) {
        if (node == null) // 
            return;
        // do something with node
        for (var i = 0; i < node.childNodes.length; i++) {
            walkTree(node.childNodes[i]);
        }
        }
        ```

12. 箭头函数

    箭头函数会捕捉闭包上下文的this值，以词法方式绑定this。
    ```
    function Person(){
    this.age = 0;

    setInterval(() => {
        this.age++; // |this| properly refers to the person object
    }, 1000);
    }

    var p = new Person();
    ```