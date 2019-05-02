---
layout: post
title: javascript查漏补缺之十四 ——《客户端web api》
date: 2018-07-25
tag: javascript
---

[链接地址](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Client-side_web_APIs)

1. javascript api的共同点

    1. 基于对象
    
    2. 有可识别的入口点

    3. 它们使用事件来处理状态的变化

    4. 它们在适当的地方有额外的安全机制

<!-- more -->

2. [操作文档](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Client-side_web_APIs/Manipulating_documents)

    1. document.querySelector(selectors)

        1. 文档对象模型Document引用的querySelector()方法返回文档中与指定选择器或选择器组匹配的第一个 html元素Element。 如果找不到匹配项，则返回null。

        2. selectors:包含一个或多个要匹配的选择器的 DOM字符串DOMString。 该字符串必须是有效的CSS选择器字符串

        3. **使用深度优先的遍历顺序**

        4. 返回Element对象

    2. document.querySelectorAll(selectors)

        1. **使用深度优先的遍历顺序**

        2. selectors 是一个由逗号连接的包含一个或多个CSS选择器的字符串

        3. 返回一个**静态的** NodeList 类型的对象

    3. document.createElement();

        `var para = document.createElement('p');`

    4. Node.appendChild()

        `sect.appendChild(para);`

    5. Document.createTextNode()

        创建一个文本节点

    6. node.textContent

    7. node.innerText

    8. node.innerHTML

        返回文本内容和标签

    9. Node.removeChild()

        当知道要删除节点以及父节点的时候可以使用这个方法：

        `sect.removeChild(linkPara);`

    10. 删除一个仅基于自身引用的节点

        `linkPara.parentNode.removeChild(linkPara);`

    11. Node.cloneNode(deep)

        1. deep: 如果为true，表示执行深复制，当前节点的所有子节点也会一并复制

        2. 如果deep参数设为false,则不克隆它的任何子节点.该节点所包含的所有文本也不会被克隆,因为文本本身也是一个或多个的Text节点

        3. 这个函数复制的节点信息包括：克隆一个元素节点会拷贝它所有的属性以及属性值,当然也就包括了属性上绑定的事件(比如onclick="alert(1)"),但不会拷贝那些使用addEventListener()方法或者node.onclick = fn这种用JavaScript动态绑定的事件

    12. HTMLElement.style

        1. 使用这个属性的结果是样式都是**内联样式**。

    13. Element.setAttribute()

        1. 使用这个方法设置的样式是普通的css样式表的样式：`Element.setAttribute(class, className)`。

    14. 获取视窗尺寸

        `window.innerWidth`：视窗宽度

        `window.innerHeight`:视窗高度
    
    15. 视窗的resize事件

        ```javascript
        window.onresize = function() {
            WIDTH = window.innerWidth;
            HEIGHT = window.innerHeight;
            div.style.width = WIDTH + 'px';
            div.style.height = HEIGHT + 'px';
        }
        ```
    - 给DOM添加节点需要上述1、2、3、4、5五个方法;

        下面这个例子里添加文本使用的是Node.textContent属性。

        ```javascript
        var sect = document.querySelector('section');

        var para = document.createElement('p');
        para.textContent = 'We hope you enjoyed the ride.';

        sect.appendChild(para);

        var text = document.createTextNode(' — the premier source for web development knowledge.');

        var linkPara = document.querySelector('p');
        linkPara.appendChild(text);

        ```


        