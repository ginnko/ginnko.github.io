---
layout: post
title: javascript查漏补缺之十一 ——《对象模型的细节》
date: 2018-07-23
tag: javascript
---

[链接地址](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Details_of_the_Object_Model)

1. JavaScript 是一种基于原型而不是基于类的面向对象语言。

<!-- more -->

2. 对象模型使用（创建Employee层级结构）

    Employee的层级结构见下图：

    ![employee层级结构](/images/js/6.png)

    1. 创建Employee的构造函数

        ```javascript
        function Employee () {
          this.name = "";
          this.dept = "general";
        }

        ```

    2. 创建Manager和WorkerBee

        ```javascript
        function Manager () {
          Employee.call(this);
          this.reports = [];//增加了reports属性
        }

        Manager.prototype = Object.create(Employee.prototype);

        function WorkerBee () {
          Employee.call(this);
          this.projects = []//增加了projects属性
        }

        WorkerBee.prototype = Object.create(Employee.prototype);
        ```

    3. 创建SalesPerson和Engineer

        ```javascript
        function SalesPerson () {
          WorkerBee.call(this);
          this.dept = 'sales';//给这个属性重新赋值
          this.quota = 100;//增加quota属性
        }

        SalesPerson.prototype = Object.create(WorkerBee.prototype);

        function Engineer () {
          WorkerBee.call(this);
          this.dept = 'engineering';//给这个属性重新赋值
          this.machine = '';//增加了machine属性
        }

        Engineer.prototype = Object.create(WorkerBee.prototype);
        ```

    4. 利用构造函数创建对象

        ```javascript
        var mark = new WorkerBee;
        ```

        上面的代码经历了下面的创建过程：

        >当javascript发现`new`操作符时，它会创建一个**通用对象**,并将其作为关键字`this`的值传递给`WorkerBee`的构造函数。该构造函数显式地设置projects属性的值，然后隐式地将其内部的`[[Prototype]]`属性设置为`WorkerBee.prototype`的值。内置的`[[Prototype]]`属性决定了用于返回属性值的原型链。一旦这些属性设置完成，javascript返回新创建的对象，然后赋值语句会将变量mark的值指向该对象。

        >这个过程不会显式的将 mark所继承的原型链中的属性值作为本地变量存放在 mark 对象中。当请求属性的值时，JavaScript 将首先检查对象自身中是否存在属性的值，如果有，则返回该值。如果不存在，JavaScript会检查原型链（使用内置的 [[Prototype]] 属性）。如果原型链中的某个对象包含该属性的值，则返回这个值。如果没有找到该属性，JavaScript 则认为对象中不存在该属性。

        ```javascript
        mark.name = "";
        mark.dept = "general";
        mark.projects = [];
        ```
        
        >mark 对象从`mark.__proto__`中保存的原型对象中继承了`name`和`dept`属性的值。并由`WorkerBee`构造器函数为`projects`属性设置了本地值。 这就是`JavaScript`中的属性和属性值的继承。

        使用`Object.hasOwnProperty`检验上面三个属性，返回`true`，可见不论是构造函数显式设置的属性还是通过原型链隐式传回的属性，这些属性都是对象自身的属性，如果此时修改这些属性，只是针对这个对象，不会影响原型，不会影响相同原型的其他对象。

    5. 对象模型的另一种使用

        1. 创建可以传参数的构造函数 

        ```javascript
        function Employee(name, dept){
          this.name = name || '';
          this.dept= dept || 'general'
        }
        ```

        2. 在下一层构造函数中调用上层构造函数

        ```javascript
        function WorkerBee(name, dept, projs){
          this.base = Employee;
          this.base(name, dept);
          this.projects = projs || [];
        }

        WorkerBee.prototype = new Employee;
        ```
        注意上面的代码！先将`Employee`赋值给`this.base`，然后再通过`this.base`调用，这时，Employee内部的`this`指向`this.base`中的`this`也就是使用`new`关键字创建的对象。

        3. 使用`new`调用构造函数创建对象

        ```javascript
        const mark = new WorkerBee("smith, mark", 'training', ['javascript']);

        function Engineer (name, projs, mach) {
          this.base = WorkerBee;
          this.base(name, "engineering", projs);
          this.machine = mach || "";
        }

        Engineer.prototype = new WorkerBee;

        var jane = new Engineer("Doe, Jane", ["navigator", "javascript"], "belau");
        ```

        javascript会按以下步骤执行:

        1. new 操作符创建了一个新的通用对象，并将其 __proto__ 属性设置为 Engineer.prototype。

        2. new 操作符将该新对象作为 this 的值传递给 Engineer 构造器。
        
        3. 构造器为该新对象创建了一个名为 base 的新属性，并指向 WorkerBee 的构造器。这使得 WorkerBee 构造器成为 Engineer 对象的一个方法。

        4. 构造器调用`base`方法，将传给该构造器的参数中的两个，作为参数传递给base方法，同事还传递一个字符串参数`engineering`。显式地在构造器中使用`engineering`表明所有Engineer对象继承的`dept`属性具有相同的值，且该值重载了继承自`Employee`的值。

        5. 因为`base`是`Engineer`的一个方法，在调用`base`时，javascrit将在步骤1中创建的对象绑定给`this`关键字。这样，`WorkerBee`函数接着将`Doe， Jane`和`engineering`参数传递给`Employee`构造器函数。当从`Employee`构造器函数返回时，`WorkerBee`函数用剩下的参数设置`projects`属性。

        6. 当从`base`方法返回后，`Engineer`构造器将对象的`machine`属性初始化为`belau`

        7. 当从构造器返回时，javascript将新对象赋值给`jane`变量。

        **重要说明：**

        ```javascript
        function Engineer (name, projs, mach) {
          this.base = WorkerBee;
          this.base(name, "engineering", projs);
          this.machine = mach || "";
        }
        var jane = new Engineer("Doe, Jane", ["navigator", "javascript"], "belau");
        Employee.prototype.specialty = "none";
        ```
        如果代码写成上面这样，对象jane不会继承speciality属性。必须显式地设置原型才能确保动态的继承，也就是添加下面这行代码：

        `Engineer.prototype = new WorkerBee;`

        此刻，对象jane就可以动态的继承speciality属性了。

3. instance操作符

    instanceof 操作符可以用来将一个对象和一个函数做检测，如果对象继承自函数的原型，则该操作符返回真。
