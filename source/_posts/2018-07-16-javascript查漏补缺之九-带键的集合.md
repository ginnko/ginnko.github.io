---
layout: post
title: javascript查漏补缺之九 ——《带键的集合》
date: 2018-07-16
tag: javascript
---

### [1.Map](http://es6.ruanyifeng.com/#docs/set-map#Map)

1. 创建Map

  1. 使用`set`方法添加

    ```
    const m = new Map();
    const o = {p: 'Hello World'};

    m.set(o, 'content')
    ```

    *set方法返回的是当前的Map对象，因此可以采用链式写法。*

    ```
    let map = new Map()
      .set(1, 'a')
      .set(2, 'b')
      .set(3, 'c');
    ```

  2. 调用构造函数的时候传入一个二维数组

    ```
    const map = new Map([
      ['name', '张三'],
      ['title', 'Author']
    ]);
    ```

2. 判断Map结构中是否包含某个键值，使用`has`方法

  `map.has('name') // true`

3. 获取Map结构某个键对应的值，使用`get`方法

  `map.get('name') // "张三"`

4. 删除某个键值对，使用`delete`方法

  `m.delete(o) // true`

5. 清除所有成员，使用`clear`方法
  ```
  let map = new Map();
  map.set('foo', true);
  map.set('bar', false);

  map.size // 2
  map.clear()
  map.size // 0
  ```
6. 对同一个键多次赋值，后面的值将覆盖前面的值。

7. 读取一个未知的键返回undefined。

8. 只有对同一个对象的引用，Map 结构才将其视为同一个键。**陷阱！大陷阱！！！**墙裂注意下面的代码。原因：**Map 的键实际上是跟内存地址绑定的，只要内存地址不一样，就视为两个键。这就解决了同名属性碰撞（clash）的问题.**

  ```
  const map = new Map();

  map.set(['a'], 555);
  map.get(['a']) // undefined
  ```
9. 如果 Map 的键是一个简单类型的值（数字、字符串、布尔值），则只要两个值严格相等，Map 将其视为一个键。虽然NaN不严格相等于自身，但 Map 将其视为同一个键。

10. 三个遍历器生成函数

  1. 遍历的顺序就是插入的顺序。

  2. 使用这三个函数返回的是这种形式的结果：`MapIterator {"F", "T"}`。这就是传说中的遍历器？！**感觉可以将这个遍历器视为一个数组？下面第11项感觉可以佐证这个猜想。**

    然后利用这个遍历器的方法如下：

    ```
    for (let key of map.keys()) {
      console.log(key);
    }
    ```
    用`for of`循环来利用这个遍历器。

  3. Map结构的默认遍历器接口就是entries方法。

  ```
  for (let [key, value] of map.entries()) {
      console.log(key, value);
    }
    // "F" "no"
    // "T" "yes"

    // 等同于使用map.entries()
    for (let [key, value] of map) {
      console.log(key, value);
    }
  ```

11. Map结构转数组

  ```
  const map = new Map([
    [1, 'one'],
    [2, 'two'],
    [3, 'three'],
  ]);

  [...map.keys()]
  // [1, 2, 3]

  [...map.values()]
  // ['one', 'two', 'three']

  [...map.entries()]
  // [[1,'one'], [2, 'two'], [3, 'three']]

  [...map]
  // [[1,'one'], [2, 'two'], [3, 'three']]
  ```

12. Map结构的遍历函数`forEach`

  1. 使用方法类似数组的`forEach`

    ```
    map.forEach(function(value, key, map) {
      console.log("Key: %s, Value: %s", key, value);
    });
    ```

  2. forEach方法还可以接受第二个参数，用来绑定this

    ```
    const reporter = {
      report: function(key, value) {
        console.log("Key: %s, Value: %s", key, value);
      }
    };

    map.forEach(function(value, key, map) {
      this.report(key, value);
    }, reporter);
    ```
13. Map转JSON

  1. 当Map的键为字符串时

    ```
    function strMapToJson(strMap) {
      return JSON.stringify(strMapToObj(strMap));
    }

    let myMap = new Map().set('yes', true).set('no', false);
    strMapToJson(myMap)
    // '{"yes":true,"no":false}'
    ```

  2. 当Map的键包含其他类型的值时

    ```
    function mapToArrayJson(map) {
      return JSON.stringify([...map]);
    }

    let myMap = new Map().set(true, 7).set({foo: 3}, ['abc']);
    mapToArrayJson(myMap)
    // '[[true,7],[{"foo":3},["abc"]]]'
    ```

### WeakMap

1. WeakMap的专用场合就是，它的键所对应的对象，可能会在将来消失。WeakMap结构有助于防止内存泄漏。**一旦消除对该节点的引用，它占用的内存就会被垃圾回收机制释放。Weakmap 保存的这个键值对，也会自动消失。**

2. WeakMap只有四个方法可用：get()、set()、has()、delete()。没有size属性。


### [2. Set](http://es6.ruanyifeng.com/#docs/set-map#Set)

Set类似于数组，但是成员的值都是唯一的，没有重复的值。

1. 创建Set对象

    1. 使用`add`方法

      ```
      const s = new Set();

      [2, 3, 5, 4, 5, 2, 2].forEach(x => s.add(x));
      ```
    
    2. 传入数组或类数组参数

      ```
      //传入数组参数
        const set = new Set([1, 2, 3, 4, 4]);
        [...set]

      //传入类数组参数
        const set = new Set(document.querySelectorAll('div'));
        set.size // 56
      ```
2. 数组去重的方法

  ```
  // 去除数组的重复成员
  [...new Set(array)]
  ```

3. Set对象判断元素是否相等的算法

    算法类似`===`，但是NaN在Set中被认为是等于自身的，这点和`===`不！一！样！

4. Set对象的属性

    - size属性，查看元素个数

5. Set对象的方法

    - add(value)：添加某个值，返回 Set 结构本身。

    - delete(value)：删除某个值，返回一个布尔值，表示删除是否成功。

    - has(value)：返回一个布尔值，表示该值是否为Set的成员。

    - clear()：清除所有成员，没有返回值。

    Map对象添加元素使用的是`set`。

6. Set对象转成普通数组对象

    1. `[...setObject]`

    2. `Array.from(setObject)`

7. 遍历器

    - keys()

    - values()

    - entries()

  生成的遍历器结果可以使用`for of`来循环遍历进行下一步操作。由于Set对象键名和键值相同，所以`keys()`和`values()`的结果相同。

  Set结构的实例默认可遍历，默认遍历器生成函数就是`values（）`方法，所以可以省略写成：

    ```
    let set = new Set(['red', 'green', 'blue']);

    for (let x of set) {
      console.log(x);
    }
    // red
    // green
    // blue
    ```
8. forEach（）

    和Map结构相同，Set结构的forEach（）函数同样可以接受第二个参数，作为第一个函数参数中的this

9. 使用Set实现交集、并集、差集

    ```
    let a = new Set([1, 2, 3]);
    let b = new Set([4, 3, 2]);

    // 并集
    let union = new Set([...a, ...b]);
    // Set {1, 2, 3, 4}

    // 交集
    let intersect = new Set([...a].filter(x => b.has(x)));
    // set {2, 3}

    // 差集
    let difference = new Set([...a].filter(x => !b.has(x)));
    // Set {1}
    ```
10. Set的一个用法

    **Set的遍历顺序就是插入顺序。这个特性有时非常有用，比如使用 Set 保存一个回调函数列表，调用时就能保证按照添加顺序调用。**


### weakSet

WeakSet 的成员只能是对象，而不能是其他类型的值。


### 遗留问题

在Map和Set中都出现过iterable结构这个东西，这个要看下。