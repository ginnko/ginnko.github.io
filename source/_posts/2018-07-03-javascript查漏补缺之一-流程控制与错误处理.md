---
layout: post
title: javascript查缺补漏之一 ——《流程控制与错误处理》
date: 2018-07-03
tag: javascript
---

[原文地址](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Control_flow_and_error_handling)

1. 如果finally块返回一个值，该值会是整个try-catch-finally流程的返回值，不管在try和catch块中语句返回了什么。

    下面代码中有说明，catch中的return会挂起，直到finally中的代码完成，finally中的return会覆盖catch中的return。return之后的console都不会执行。

<!-- more -->

    ```
    function f() {
      try {
        console.log(0);
        throw "bogus";
      } catch(e) {
        console.log(1);
        return true; // this return statement is suspended
                    // until finally block has completed
        console.log(2); // not reachable
      } finally {
        console.log(3);
        return false; // overwrites the previous "return"
        console.log(4); // not reachable
      }
      // "return false" is executed now  
      console.log(5); // not reachable
    }
    f(); // console 0, 1, 3; returns false
    ```

2. catch中可以再次抛出异常，但如果finally中有返回值，则这个返回值会覆盖catch中抛出的异常。

    ```
    function f() {
      try {
        throw 'bogus';
      } catch(e) {
        console.log('caught inner "bogus"');
        throw e; // this throw statement is suspended until 
                // finally block has completed
      } finally {
        return false; // overwrites the previous "throw"
      }
      // "return false" is executed now
    }
    ```

3. 嵌套try...catch

    任何给定的异常只会被离它最近的封闭 catch 块捕获一次。

    ```
    try {
      try {
        throw new Error("oops");
      }
      catch (ex) {
        console.error("inner", ex.message);
        throw ex;
      }
      finally {
        console.log("finally");
      }
    }
    catch (ex) {
      console.error("outer", ex.message);
    }

    // Output:
    // "inner" "oops"
    // "finally"
    // "outer" "oops"
    ```