---
layout: post
title: React学习
date: 2017-06-19
tag: React
---
这篇是React入门的学习总结，主要参考阮一峰大神的教程（如下）和官方文档，用codepen完成了教程里的[demo](https://codepen.io/ginnko/pen/EXWaaw)。

<!-- more -->

## 基础学习

### 1. 资料
- [阮大角虫的教程](http://www.ruanyifeng.com/blog/2015/03/react.html)
- [官网](https://facebook.github.io/react/docs/installation.html)
- [Dan Abramov大神的实例](https://codepen.io/gaearon/#)

### 2. 问题
- 不太清楚React具体能做什么
> 通过这段时间的练习，感觉React确实很适合UI，大神们说的没错。

- 教程里有个使用Promise的例子，虽然明白Promise最主要的有点就是将执行过程和结果分开，但是并没有觉得这有什么特别好处，只是看起来更清晰了？关于Promise的部分参考了[廖大角虫的教程](http://www.liaoxuefeng.com/wiki/001434446689867b27157e896e74d51a89c25cc8b43bdb3000/0014345008539155e93fc16046d4bb7854943814c4f9dc2000)。
- 当使用setInterval()和Ajax的同时使用了bind（）函数，不太理解这个地方。
> bind(this)是将当前作用域传入相应的函数，在ES6中被要求要明确的表示，因为不再自动匹配，在下面的[“是否使用ES6的方法对比”](https://facebook.github.io/react/docs/react-component.html)中有说明。  

- receive box这个应用中，自己写的代码是把所有的功能放进一个组件类中，别人的代码是像函数那样将完成一个功能的代码组合放进一个单独的组件类，在其他组件类中进行调用。后面的练习要使用这种方式。

### 3. 注意
- 组件类的变量名的首字母要**大写**！！！
- rent中的顶级标签只能有一个，所以示例中的全部用<div>标签包裹
- 关于[handle事件和this](https://facebook.github.io/react/docs/handling-events.html)  
如果一个方法后面没有直接跟着一个括号，就要手动绑定this和这个方法。解决办法有三：  
> 1. 手动在constructor方法里绑定，例如：this.handleClick = this.handleClick.bind(this);
> 2. 使用属性初始化语法 handleClick = () =>{//其他代码}**结束没有分号**
> 3. 使用箭头函数，前面不需要事先绑定或使用属性初始化语法，在元素中使用箭头函数： <button onClick={(e) => this.handleClick(e)}>Click me</button>

## 应用
### 1. Leaderboard
这个[demo](https://codepen.io/ginnko/full/bRqXaN/)是[FreeCodeCamp](https://www.freecodecamp.com/challenges/build-a-camper-leaderboard)的一个练习，使用React实现（2017.6.23完成）。
#### 一个方法：componentDidUpdate()
第一版使用如下代码，但现在看来是自己对componentDidUpdate()这个方法理解有问题，虽然能够实现下述的功能，但这个方法的真正使用方法应该不是这样。
>这个方法用来实现当属性或状态发生变化时，对页面中的相应内容进行变更。[demo](https://codepen.io/ginnko/full/bRqXaN/)中点击30天内的分数排名和总分数排名的切换就是通过这个方法实现的。实现的代码如下：  



	componentDidUpdate(){
	  this.props.promise.then(
	    value => this.setState({loading: false, data: value}),
	    error => this.setState({loading: false, error: error})
	  );
	},
	handleClick: function(event){
	  if(flag === 0){
	    flag = 1;
	    this.props.promise = $.getJSON(alltime);
	  }else{
	    flag = 0;
	    this.props.promise = $.getJSON(recent);
	  }
	}

>handleClick属性传给标题中的30天内排名和总分数排名两个标题，当点击时，会相应的改变ajax请求的api并把返回结果传递给promise属性，发生的改变会被componentDidUpdate()方法捕捉，然后使用新的数据重新渲染模板。

#### 一个属性handleClick
这个[demo](https://codepen.io/ginnko/full/bRqXaN/)最终是通过handleClick属性完成切换是根据30天分数排名还是根据总分数排名，还可以选择是正序显示还是倒序显示，不再使用componentDidUpdate()方法。具体代码如下。

	handleClick: function(event){
	  var id = event.target.parentElement.getAttribute('id');
	  if(id === "alltime"){
	    this.props.promise = $.getJSON(alltime);
	    if(flag2 === 1){
	      flag1 = 1;
	      flag2 = 0;
	      this.props.promise.then(
	        value => this.setState({loading: false, data: value, mark2 :"↓", mark1 :""}),
	        error => this.setState({loading: false, error: error})
	      );
	    }else{
	      flag1 = 1;
	      flag2 = 1;
	      this.props.promise.then(
	        value => this.setState({loading: false, data: value.reverse(), mark2 :"↑", mark1 :""}),
	        error => this.setState({loading: false, error: error})
	      );
	    }
	  }
	  if(id === "recent"){
	    this.props.promise = $.getJSON(recent);
	    if(flag1 === 1){
	      flag2 = 1;
	      flag1 = 0;
	      this.props.promise.then(
	        value => this.setState({loading: false, data: value, mark1 :"↓", mark2 :""}),
	        error => this.setState({loading: false, error: error})
	      );
	    }else{
	      flag2 = 1;
	      flag1 = 1;
	      this.props.promise.then(
	        value => this.setState({loading: false, data: value.reverse(), mark1 :"↑", mark2 :""}),
	        error => this.setState({loading: false, error: error})
	      );
	    }
	  }
	}

有两处引用了handleClick，使用**event.target.parentElement.getAttribute('id')**来判断是哪个发生了点击，并据此显示是根据哪类分数显示。  
这条代码的使用参考[此处](https://stackoverflow.com/questions/39559222/react-onclick-event-on-link)。event.target可以获得发生点击事件的DOM node本身。然后使用操纵DOM node的方法[parentElement](https://developer.mozilla.org/en/docs/Web/API/Node/parentElement)以及[getAttribute()](https://developer.mozilla.org/en-US/docs/Web/API/Element/getAttribute)进行判断。event.target真是方便啊！
#### 一些想法
1. 第一次加载时，由于需要渲染，页面是完全白色的，渲染完成后加载相应的内容。但是在初始加载完成后，后面更新数据重新渲染不会再重新加载页面而只是渲染页面变更的部分，之前一直不太明白React的用途，只是听过适合开发UI，或许上面的描述就是一个证据。
2. 本例中，使用componentDidMount方法来处理ajax请求，根据现在掌握的资料，推断这个方法只能使用一次，也就是在组件加载完成后，之后属性或状态再发生变化要通过componentDidUpdate()方法（有待研究）或handleEvent系列属性来实现。另一个handleEvent属性的实例：[阮大角虫的教程](http://www.ruanyifeng.com/blog/2015/03/react.html)中第9个demo是一个实时显示的应用，使用的是handleChange属性，属性函数里用event.target.value来获取实时改变的值并显示。

#### 一些问题  
1. 最初的代码是，关于图标和表格标题的部分都写在html文档中，页面一加载就能看到这两项内容，表格的数据部分用json获取后通过React返回。当React使用render返回一个被DOM元素包裹的数组时，React会自动将其展开一条一条列出。但是，render函数中要求只能有一个顶级标签，导致返回的表格数组和已经写好的表格标题不一样大小，只占其第一列的宽度。没有查到解决办法，参考别人的项目，也没有这样写的，最后只能让所有的部分都由React来实现。希望能找到解决上述问题的方法。
2. colspan 属性名在JSX中要写成colSpan才会被识别。
3. ES6发布后，React的一些方法有了改变，阮一峰大神的教程使用的是ES6发布前的方法。这个demo也没有使用ES6，但是在之后的练习中都要使用最新的方法。下面两个链接是React官方文档和是否使用ES6的一些方法的改变对比：  

 - [常用方法](https://facebook.github.io/react/docs/react-component.html)
 - [是否使用ES6的方法对比](https://facebook.github.io/react/docs/react-without-es6.html)  


### 2. Recipe game
这个[demo](https://codepen.io/ginnko/full/XgzqKG/)是FreeCodeCamp的一个练习，使用React实现（2017.6.28完成）。大结构使用了Bootstrap的[Modal](https://v4-alpha.getbootstrap.com/components/modal/)和cards中的[list group](https://v4-alpha.getbootstrap.com/components/card/)。使用动态Modal弹出一个窗口接受和修改数据，list group用来组织数据，静态Modal用来显示数据。
#### localStorage
参考资料：[localStorage接口](https://developer.mozilla.org/zh-CN/docs/Web/API/Storage)、[JSON.stringify()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)、[JSON.parse()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse)  
1. 能存入localstorage的数据没有限制原数据结构，只要转成字符串都可以存入。存入前使用JSON.stringify（）将原对象转成字符串，取出后使用JSON.parse()还原成原数据结构。  
2. 使用localStorage.setItem(key, value)存入对象，localStorage.getItem(key)获取对象。  
3. 对于有默认初始值得情况，使用下面的方法进行第一次赋值，借鉴了[Awesomen3ss 大神](https://codepen.io/awesom3/pen/Hlfma?editors=1010)的方法。  

项目中的代码如下：  

	var recipeList = [["Braised string bean with noodle", "garlic", "noodle", "string bean","coriander", "salt", "light soy sauce"], ["Braised hairtail", "hairtail", "vinegar", "scallion", "garlic", "ginger", "distilled spirit", "salt"], ["Scrambled eggs with tomatoes", "tomatoes", "egg", "scallion", "salt"]];  
	var condition = localStorage.getItem("recipeList");
	if(condition === null){
      localStorage.setItem("recipeList", JSON.stringify(recipeList));
	}

#### handle事件的绑定和使用
- React的ES6版本中，handle事件要在初始化中明确绑定，参考[官方文档](https://facebook.github.io/react/docs/handling-events.html)。  

下面的代码是这个项目中的使用：

	constructor(props){
	  super(props);
	  this.state = {list: localStorage.getItem("recipeList"), value: [temRecipe,temIngredients]};
	  this.handleClick = this.handleClick.bind(this);
	  this.handleChange = this.handleChange.bind(this);
	}  

- "this"位于一个函数中时，它的作用域就是这个函数，正常函数的写法无法接触到需要的DOM对象，导致无法加载DOM对象，页面一篇空白的错误，解决方法是使用[arrow function](https://stackoverflow.com/questions/41174980/handling-clicks-on-dynamically-generated-buttons)，handle函数和后面的应用函数都使用箭头函数的形式。**关于this的使用还是不太理解！**

下面的代码是这个项目中的使用：  

	handleChange = (event) => {//。。。}
	handleClick = (event) => {//。。。}
	var recipeStructure = JSON.parse(this.state.list).map((val, index) => {//。。。}
	var ingredientsList = val.slice(1).map((ele, index) => {//。。。}

- 如果一个map函数循环一个数组并作为组件对象输出到最后的DOM结构中时，要在每次输出过程中包含一个类似id的成为key的属性，来帮助React识别哪一条发生了改变，参考[官方文档](https://facebook.github.io/react/docs/lists-and-keys.html)。

#### Forms的Controlled Components
"input"、"textarea"标签使用value属性定义页面加载后输入框中的内容，要想实现能够被编辑，需要再添加onChange属性，参考[官网文档](https://facebook.github.io/react/docs/forms.html#default-value)。

### 3. Game of Life
这个[demo](https://codepen.io/ginnko/full/YQvVRw/)是FreeCodeCamp的一个练习，使用React实现（2017.7.05完成）。在做这个项目之前，把官方文档的基础部分全部看了一遍，这个项目中，尽量应用了学到的一些原则，比如把有具体单一功能的代码放到一个组件或函数中。  

- 函数
> search：获取某个存活细胞周围的8个邻居  
> execute: 判断这当代细胞的存活  
> repeat： 重复执行判断过程  

- 组件
> NewTable：制造board  
> TableConstruct: 加载board以及三类点击事件（控制、board大小、速度）  

感觉TableConstruct的内容依然过多，可以进一步分离。
#### setTimeout()
- 在React中，如果想要实现通过点击某个按钮来实现改变某个元素变化的速度，就要通过[SetTimeout()](https://stackoverflow.com/questions/1280263/changing-the-interval-of-setinterval-while-its-running)来实现。setTimeout()允许在运行过程中改变时间间隔，setInterval()没有这项功能。
- 如果要在setTimeout()或setInterval()的回调函数中直接使用所在组件的this，因为[作用域](https://stackoverflow.com/questions/26348557/issue-accessing-state-inside-setinterval-in-react-js)的原因，要var self = this;通过self传入this或手动绑定this。
- setInterval()的循环不能直接放进for或while循环体中，要是用类似迭代的方式。  
如下代码：

		function repeat(){
		  execute();
		  timer = setTimeout(repeat, self.state.speed);
		}

#### state & props
本例中，NewTable组件里同时使用了props和state，忘记官方文档里是不是有说尽量把state和props分开使用了，这个有待确定。代码如下：

	  class NewTable extends React.Component{
	    constructor(props){
	      super(props);
	      this.handleClick = this.handleClick.bind(this);
	      this.state = {};
	    }
	    handleClick(e){
	      var flag = e.target.getAttribute("class");
	      if(flag === "white"){
	        e.target.setAttribute("class", "red");
	      }else{
	        e.target.setAttribute("class", "white");
	      }
	    }
	    render(){
	      var row = [];
	      var rowLen = this.props.rowNumber;
	      var columnLen = this.props.columnNumber;
	      var key = 0;
	      var flag = 0;
	      for(let j = 0; j<rowLen; j++){
	        var tableData = [];
	        for(let i = 0; i<columnLen; i++){
	          if(start === 0){
	            flag = Math.round(Math.random());
	          }else{
	            flag = 0;
	          }
	          tableData.push(<td id={key++} r={0} onClick={this.handleClick} className={flag===0?"white":"red"} role="button"></td>);
	        }
	        row.push(<tr>{tableData}</tr>);
	      }
	      start = 1;
	      return <tbody>{row}</tbody>;
	    }
	  }

#### 关于jQuery和DOM的操作
- 使用jQuery删除属性，方法：`$("p").removeAttr("id");`即可成功删除id属性。
- DOM元素使用`getAttribute()`和`setAttribute()`两个函数来操作属性；而jQuery使用`.attr()`来操作属性

#### JSX中的属性名称
官方文档中，有说JSX属性名称使用驼峰命名法，这个项目中NewTable组件里的这行代码`tableData.push(<td id={key++} r={0} onClick={this.handleClick} className={flag===0?"white":"red"} role="button"></td>);`最开始属性名**r**是写作**relive**但是在后面获取时失败，这明明是一个单词，不知道该在哪里驼峰。。。有待确认。  

### 4. [Roguelike Dungeon Crawler Game](https://codepen.io/ginnko/full/JydBWO/)(2017.7.28完成)
这个小项目是目前最耗时的一个，写了很久，也有很多想法。最重要的一个想法是基础太重要了，所以决定后面拿出一部分时间来复习css、js基础。下面来说这个项目。  

#### 过程：  
界面可以分成五个部分，左侧的状态、中间的游戏板、上方的阴影遮挡、右侧的解释以及下方的说明。  
- 下方的说明：说明部分没有借助React，是使用html节点直接显示
- 左侧状态栏、右侧解释栏：独立组件，使用React生成，但是其中信息的更新使用了全局变量，没有通过React的状态更新（因为不知如何用React处理游戏板更新但状态栏和解释栏不变的情况）。
- 中间游戏板： 独立组件，使用React生成，每一层的随机设置使用Math.random()函数，层与层的转换借助floor state。
- 上方阴影遮挡： 独立组件，使用React生成，代码类似中间游戏板，去掉了随机布置的部分。透明部分使用jQuery和css完成定位和透明度转换。  

#### 遇到的问题和解决办法
1. 第一个问题是在<table>标签中使用onKeyDown事件，搜索发现，有两种解决办法。第一种是，在table类的标签中增加**tabindex="0"**属性，用来解决table类的节点不能focused的问题，但失败了，其没有找到原因，猜测和作用域有关系；第二种办法是，在componentDidMount中设置事件监听函数，进行窗口监听，获取onKeyDown事件，成功，监听代码见[MozillaMDN](https://developer.mozilla.org/zh-CN/docs/Web/API/KeyboardEvent/key)。  
2. 在外部函数中改变组件的state，来re-render。解决办法是利用全局变量floor和新建一个变量var self = this;来消除作用域的隔绝。  

			 floor = 1;//全局变量  
			 var self = this;//用self变量消除作用域隔绝问题
			 move (id, nav, self)//将self作为函数参数传入外部move函数  
			 floor++;
			 self.setState({floor: floor});//在move中改变组件的state
3. 地图死角问题。使用外部函数fix来解决，开始想在地图生成组件的最后来解决，但是发现捕捉到的节点全是未定义，由于React在加载前构建的全都是空节点，所以没办法使用这种方法，所以在componentDidMount、componentDidUpdate中调用fix函数通过jQuery来对地图调整。
4. 左侧状态栏、中间游戏板、右侧解释栏以及上方阴影遮挡的布局问题。最后使用绝对位置和相对位置成功解决。**基础就是这么的重要啊！！！**
5. 上方阴影遮挡空白处随方向键移动问题。通过外部函数eye（），利用jQuery和css解决，使用了jQuery中我不太常用的.filter（）函数。另外，在实现这个功能的过程中，开始fighter从一层进入下一层时，上一层的空白可见区域会遗留而不跟随新的fighter位置变化。开始以为这个问题是因为阴影遮挡组件没有状态改变，所以不会重新加载的原因，但是通过获取fighter的更新位置，发现在地图生成组件和componentDidUpdate中获取的位置是一致的，而且只使用了jQuery和css实现，应该不是React的render问题。最后发现是在事件监听部分，先把fighter的id获取传给了一个变量，再将这个变量传给位于move（）后面的eye（）函数。move（）函数中已经变化了层数，更新了fighter的位置，但是传入的id是上一层的door的位置，所以此处不借助中间变量，直接使用`$(".red").attr("id")`获取最新的位置，解决了这个问题。  

#### 遗留的问题
1. 地图生成使用了最简单的随机生成，生成复杂的规整地图有些乏力；
2. 目前没有实现固定小窗口，在小窗口内移动变换空间的功能，猜测使用了overflow属性，但查看资料overflow只对block元素有效，table类元素调整起来貌似有些困难。
