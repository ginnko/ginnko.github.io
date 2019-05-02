---
layout: post
title: javascript查漏补缺之十——《使用对象》
date: 2018-07-18
tag: javascript
---

[链接地址](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Working_with_Objects)

1. JavaScript中的对象只能使用String类型作为键类型。

2. 对对象的枚举

    ```
    function showProps(obj, objName) {
      var result = "";
      for (var i in obj) {
        if (obj.hasOwnProperty(i)) {
            result += objName + "." + i + " = " + obj[i] + "\n";
        }
      }
      return result;
    }
    ```

    上面的列子使用了`for in` 和 `hasOwnProperty`，自己几乎没有这样写过代码，都是先用`Object.keys`获取所有的键，然后循环键组成的数组进行操作。下次可以试一波。

3. 枚举一个对象属性的三种方法

    1. `for in`依次访问一个对象及其原型链中所有可枚举的属性。

    2. `Object.keys(o)`返回一个对象 o 自身包含（不包括原型中）的所有属性的名称的数组。

    3. `Object.getOwnPropertyNames(o)`该方法返回一个数组，它包含了对象 o 所有拥有的属性（无论是否可枚举）的名称。**注意！不论可否枚举！！！不论可否枚举！！！不论可否枚举！！！**

4. 创建对象的方法

    1. 使用对象字面两

    2. 使用构造函数

    3. 使用 Object.create 方法

      ```
      // Animal properties and method encapsulation
      var Animal = {
        type: "Invertebrates", // Default value of properties
        displayType : function() {  // Method which will display type of Animal
          console.log(this.type);
        }
      }

      // Create new animal type called animal1 
      var animal1 = Object.create(Animal);
      animal1.displayType(); // Output:Invertebrates

      // Create new animal type called Fishes
      var fish = Object.create(Animal);
      fish.type = "Fishes";
      fish.displayType(); // Output:Fishes
      ```

5. getters与setters

    ```
    var o = {
      a: 7,
      get b() { 
        return this.a + 1;
      },
      set c(x) {
        this.a = x / 2
      }
    };

    console.log(o.a); // 7
    console.log(o.b); // 8
    o.c = 50;
    console.log(o.a); // 25
    ```
  
    1. 上面定义getters使用了get

    2. 定义setters使用了set

    3. 使用getter和setter时的方式和普通对象属性相同

6. 第二种使用getters和setters的方法

    ```
    var d = Date.prototype;
    Object.defineProperty(d, "year", {
      get: function() { return this.getFullYear() },
      set: function(y) { this.setFullYear(y) }
    });
    ```

7. delete操作符

    **注意！**`delete`操作符只能用于删除一个不是继承来的属性。