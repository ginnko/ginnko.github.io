---
layout: post
title: javascript查漏补缺之六——《字符串》
date: 2018-07-09
tag: javascript
---

[译文链接](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide)

<!-- more -->

1. String对象方法

    1. `String.prototype.charAt()`：从一个字符串中返回指定的字符。

        ```
        var anyString = "Brave new world";

        console.log("The character at index 0   is '" + anyString.charAt(0)   + "'");
        console.log("The character at index 1   is '" + anyString.charAt(1)   + "'");
        console.log("The character at index 2   is '" + anyString.charAt(2)   + "'");
        console.log("The character at index 3   is '" + anyString.charAt(3)   + "'");
        console.log("The character at index 4   is '" + anyString.charAt(4)   + "'");
        console.log("The character at index 999 is '" + anyString.charAt(999) + "'");

        //输出结果：
        The character at index 0 is 'B'
        The character at index 1 is 'r'
        The character at index 2 is 'a'
        The character at index 3 is 'v'
        The character at index 4 is 'e'
        The character at index 999 is ''
        ```

    2. `String.prototype.charCodeAt(index)`: 能正确返回一个两个字节组成的字符的码点值。

    3. `String.prototype.pointCodeAt(index)`： 能正确返回一个四个字节组成的字符的码点值。

    4. `String.prototype.indexOf()`: indexOf() 方法返回调用  String 对象中第一次出现的指定值的索引，开始在 fromIndex进行搜索。如果未找到该值，则返回-1。**这个方法区分大小写**。

        ```
        'Blue Whale'.indexOf('lu')
        //1

        //判断某个字符串是否存在于另一个字符串中
        "Blue Whale".indexOf("Blue") !== -1; // true
        "Blue Whale".indexOf("Bloe") !== -1; // false
        ```
    5. `String.prototype.lastIndexOf()`: lastIndexOf() 方法返回指定值在调用该方法的字符串中最后出现的位置，如果没找到则返回 -1。从该字符串的后面向前查找，从 fromIndex 处开始。

        其中， fromIndex从调用该方法字符串的此位置处开始查找。可以是任意整数。默认值为 str.length。如果为负值，则被看作 0。如果 fromIndex > str.length，则 fromIndex 被看作 str.length。

    6. `String.prototype.startsWith()`: 用来判断当前字符串是否以另外一个给定的子字符串开头，返回true或false。

        ```
        var str = 'To be, or not to be, that is a question.';

        alert(str.startsWith('To be'))//true
        alert(str.startsWith('not to be'))//false
        alert(str.startsWith('not to be', 10))//true
        ```

    7. `String.prototype.endWith()`: endsWith()方法用来判断当前字符串是否是以另外一个给定的子字符串“结尾”的，根据判断结果返回 true 或 false。

        ```
        var str = "To be, or not to be, that is the question.";

        alert( str.endsWith("question.") );  // true
        alert( str.endsWith("to be") );      // false
        alert( str.endsWith("to be", 19) );  // true

        注意最后一个alert，结束位置是19而不是18,19表示的是be后面的那个逗号。
        ```

    8. `String.prototype.includes()`: 判断一个字符串中是否包含另一个字符串，返回true或false。

        ```
        var str = 'To be, or not to be, that is the question.';

        console.log(str.includes('To be'));       // true
        console.log(str.includes('question'));    // true
        console.log(str.includes('nonexistent')); // false
        console.log(str.includes('To be', 1));    // false
        console.log(str.includes('TO BE'));       // false
        ```
    9. `String.prototype.concat()`:字符串拼接函数，由于性能问题，mdn建议使用`+`代替这个函数拼接字符串。自己写代码好像还真没用过这个函数。

    10. `String.fromCharCode()`：这个函数是String类的静态方法，必须这样使用。返回**字符串**。

        ```
        String.fromCharCode(65,66,67)//'ABC'
        ```
    11. `String.fromCodePoint()`: 这个函数同样是String类的静态方法。参数是高位码点。

    12. `String.prototype.split(seperator[,limit])`: 使用指定的分隔符字符串将一个String对象分割成字符串数组，以将字符串分隔为子字符串，以确定每个拆分的位置。separator 可以是一个字符串或正则表达式。

        - 例1：使用空格作为Seperator
        ```
        "Webkit Moz O ms Khtml".split( " " )   // ["Webkit", "Moz", "O", "ms", "Khtml"]
        ```

        - 例2：使用正则表达式作为Seperator,正则表达式不需要加`g`标识符。

        ```
        var myString = "Hello 1 word. Sentence number 2.";
        var splits = myString.split(/(\d)/);

        console.log(splits);

        //[ "Hello ", "1", " word. Sentence number ", "2", "." ]
        ```

        - 例3：设置第二个参数

        ```
        var myString = "Hello World. How are you doing?";
        var splits = myString.split(" ", 3);

        console.log(splits);

        //["Hello", "World.", "How"]

        ```
    13. `String.prototype.slice(begin[, end])`:返回一个子字符串，提取范围是[begin, end),其中end为可选参数，如果begin或end为负值，则实际为length+begin/length+end。

    14. `String.prototype.substring(begin[, end]`: 返回一个子字符串。类似上面的slice方法。当begin>end时，会交换begin和end的位置执行。

    15. `String.prototype.substr(start[, length])`：返回一个字符串中从指定位置开始到指定字符数的字符。

    16. `String.prototype.match()`： `match（）`函数是**字符对象**的方法！！！

        - 接受一个**正则表达式**

            1. 正则表达式没有加`g`时，返回的数组会带有一个`index`属性，表示匹配结果的索引；

            ```
            var reg = /matc(h)/;

            var str = 'this is a match function test.';

            var re = str.match(reg);

            re;

            // ["match", "h", index: 10, input: "this is a match function test."]
            ```

            2. 如果加了`g`，则只会返回匹配项组成的数组（不包含括号）。

            ```
            var reg = /matc(h)/g;

            var str = 'this is a match function test.';

            var re = str.match(reg);

            re;

            // ["match"]
            ```

    17. `String.prototype.replace()`:replace() 方法返回一个由替换值替换一些或所有匹配的模式后的新字符串。模式可以是一个字符串或者一个正则表达式, 替换值可以是一个字符串或者一个每次匹配都要调用的函数。(自己写代码有用到这个函数多次，还算熟)。

        >str.replace(regexp|substr, newSubStr|function)

        1. 第二个参数是**字符串**时, 可以使用下图中所示变量名。对于最后一个变量名`$n`，要特别说明下，第一个括号用1表示而不是0。

        ![特殊变量名](/images/js/3.png)

            ```
            //交换一个字符串中两个单词的位置

            var re = /(\w+)\s(\w+)/;
            var str = "John Smith";
            var newstr = str.replace(re, "$2, $1");
            // Smith, John
            console.log(newstr);
            ```

        2. 第二个参数是**函数**时，每次匹配成功都会调用这个函数，将返回结果作为替代字符串。

        ![函数参数](/images/js/4.png)

    18. `String.prototype.search()`: 如果匹配成功，则 search() 返回正则表达式在字符串中首次匹配项的索引。否则，返回 -1。

        使用**-1**来判断是否有匹配成功。

        ```
        function testinput(re, str){
        var midstring;
        if (str.search(re) != -1){
            midstring = " contains ";
        } else {
            midstring = " does not contain ";
        }
        console.log (str + midstring + re);
        }
        ```

    19. `String.prototype.repeat()`：记得以前看python的时候好像字符串和乘号能直接翻倍字符串，js中好像没有这种功能。

    20. `String.prototype.trim()`: **trim方法**只会删除开头和结尾的空白，对于字符中的空白无能为力。

2. 模板字符串

    1. 换行：这个超方便。超方便！




