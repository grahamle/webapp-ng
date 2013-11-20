---
title: ng中的MVC
layout: page
category: ngzend
date: 2013-11-06
modifiedOn: 2013-11-20
---

## angular中的MVC模式
现在市面上的web-app大多数都是基于某种MVC模式或其变式。但是MVC的问题在于它不是一个很精确的模式，相反它是一个相对顶层的设计，属于架构层次的。而基于MVC而衍生的变种也很多，如MVP、MVVM是属于大众人气比较高的。记住一点，**不同的开发团队，他们理解的MVC模式是不尽相同的**。这一点，Martin Fowler在[这篇文章](http://martinfowler.com/eaaDev/uiArchs.html)里进行了总结。

### **麻雀虽小，当应五脏俱全**
之前hello-world的例子中把数据模型初始化、逻辑和视图混在一个文件中，这是典型的没有展现分层的策略。那如果要把angular团队强调的MVW模式引入进来，那么这个例子要如何重写呢（以后书中所有代码均略去初始化及脚步包含部分）？

	<div ng-controller="HelloCtlr">
		Say hello to: <input type="text" ng-model="name"><br>
		<h1>Hello, { {name} }!</h1>
	</div>

我们移除了 `ng-init` 属性，取而代之的是一个指向了一个js函数的 `ng-controller` 指令，它的代码如下：

	var HelloCtrl = function ($scope) {
		$scope.name = "World";
	}

逻辑很简单，就是把原来通过 `ng-init` 准备的数据模型 `name` 现在我们用另外一个指令来完成，这个指令就是 `ng-controller` ，它是通过往 `$scope` 对象中添加属性的方式来准备好我们需要的数据模型。也就是说，现在我们的数据模型都存放到了 `$scope` 对象之中了，那么这个对象又是何方神圣呢？

### **$scope对象基础**
关于 `$scope` 对象的几点：

- `$scope` 对象负责把*数据模型对象(集)*暴露给模版对象/视图对象
- 为 `$scope` 对象增加属性可以是数据，也可以是函数操作，但是它们都是有具体的视图对象范围的，也就是说我们可以通过一个 `$scope` 对象实例将具体的UI操作逻辑暴露给模版对象
- `$scope` 对象允许我们可以精确地控制整个对象的哪些部分是可以从视图/模版中获取到的（如以下的getName）

上面的 `HelloCtrl` 可以新增加一个 `getter` 函数用来取name值，这样就可以把name值当作私有属性而不直接暴露在视图对象中了，代码如下：

	var HelloCtrl = function ($scope) {
		$scope.name = "World";
		$scope.getName = function() {
			return $scope.name;
		}
	}

然后就可以在模版中使用这个getter了：

	<h1>Hello, { {getName()} }!</h1>

### **controller**
上面我们讲是通过 `ng-controller` 替代了 `ng-init` 来进行数据模型的准备，而这就是angular中的又一重要组成部分：**controller**，也就是控制器。它的首要任务就是初始化 `$scope` 对象，进而为储存数据模型对象集做准备。而这个初始化过程又主要包括这两个任务：

1. 提供初始化数据模型值
2. 用具体的UI操作逻辑（即函数）来扩展 `$scope` 对象

controller（控制器），说到底其实就是普通的js函数。它的运转是无需扩展特定的框架类或是调用angular的APIs。另外，就如我前面所说，`ng-controller` 无非就是用js的方式来表达了 `ng-init` 这样一个过程，而且避免了将初始化逻辑（这里是数据模型）与HTML模版混杂。

### **model**
Model（模型）就是普通的js对象。通过上一节中介绍的controller初始化的两个任务，其实可以看到，它们就是MVC架构中的Model（模型）的两个重要部分：

- 数据模型（私有数据）
- UI操作逻辑（公有函数）

最后Model通过暴露自己的UI操作逻辑来实现封装数据，使用安全。

而Model，说白了，就是被当作属性添加到 `$scope`对象上的，这样就能暴露给模版对象了。

### **$scope对象进阶**
通过以上各小节从：`$scope` 对象基础 --> controller --> model 这样的流程，我们可以来更深地学习一下这个对象。首先我们必须明确一点：** `$scope` 对象是 `Scope` 类的一个实例**。而 `Scope` 类拥有诸多方法如控制着它的实例的生命周期，提供事件传播机制，以及支持模版渲染进程。

*scopes的层次*

重新看一遍 `HelloCtrl` 的代码，与普通js的构造函数没有什么区别，只有唯一的一个值得注意的地方就是函数参数 `$scope`，那么这个参数是哪里来的呢？这就牵扯到 `$scope` 对象的生命周期了：

- 首先一个新的scope可以通过 `ng-controller` 指令通过调用 `Scope.$new()` 函数来创建的
- 能够调用上述方法的前提是我们拥有一个scope的实例，angular中有：`$rootScope`，这个对象是整个应用其他 `scopes` 的父亲，它在应用启动时被创建
- scopes形成了一个以 `$rootScope` 为根节点父子节点组成的树状图，这和DOM树是一致的，也就是说scopes中的各个scope是在对应着DOM结构

*能够创建scopes的指令*

`ng-controller` 是其中一个可以可以创建scope的指令，其他还有很多类似的创建scope的指令。angular在扫描DOM树时每遇到一个这样可以创建scope的指令的时候，都会创建一个 `Scope` 的实例。新创建的scope都有一个 `$parent` 属性指向它们的父级scope。

现在来看另一个能够创建子级scope的指令：`ng-repeat`

	// the controller
	var WorldCtrl = function ($scope) {
		$scope.population = 7000;
		$scope.countries = [
			{name: 'France', population: 63.1},
			{name: 'United Kingdom', population: 61.8}
		];
	};

	// the markup fragment
	<ul ng-controller="WorldCtrl">
		<li ng-repeat="country in countries">
			{ {country.name} } has a population of { {country.population} }
		</li>
		<hr>
		World's population: { {population} } millions
	</ul>

`ng-repeat` 创建了两个新的scope，在视图中，也就对应着两个新的`<li>`，在Chrome中你可以通过Batarang插件看的一清二楚。`ng-repeat` 中指定的新的变量 `country` 成为了新的两个子级scope中的数据。这时候你会看到两个 `<li>` 它们对应的scope中有重名函数，但这不会导致命名冲突，因为**每个 `<li>` 拥有它自己的scope也就有它自己的命名空间。

*scopes的继承*

定义在高层scope中的属性是可以被其子级scopes继承，因为angular规定：子级scope不重新定义父级scope中已有的属性。以上面的国家人口例子为开始，现在我们想要显示各个国家占全球总人口的百分比，那么在父级scope（也就是`ng-controller`创建的scope，对应模版中的`<ul>`）定义一个计算百分比的逻辑，那么在两个`<li>`中均可以调用到该函数（可以理解为继承了），而不用分别在两个`<li>`中做冗余的函数定义了，代码如下：

	// the controller
	var WorldCtrl = function ($scope) {
		$scope.population = 7000;
		$scope.countries = [
			{name: 'France', population: 63.1},
			{name: 'United Kingdom', population: 61.8}
		];
		$scope.worldsPercentage = function (countryPopulation) {
			return (countryPopulation / $scope.population) * 100;
		}
	};

	// the markup fragment
	<ul ng-controller="WorldCtrl">
		<li ng-repeat="country in countries">
			{ {country.name} } has a population of { {country.population} },
			{ {worldsPercentage(country.population)} } % of the World's population
		</li>
		<hr>
		World's population: { {population} } millions
	</ul>

总结来讲，angular中的scope的继承遵循js的原型继承的相关规则（也就是说当我们试着读取一个属性时，原型链机制启用，一层层往上直到这个属性找到）。

*scopes继承的潜在危险*

scopes的继承机制在**读属性**这个操作时没有问题简单易懂，而在**写属性**时，事情就变得十分复杂。还是拿hello-world的那个例子来看：

	// the markup fragment
	<body ng-app  ng-init="name='World'">
	<h1>Hello, { {name} } </h1>
	<div ng-controller="HelloCtrl">
    	Say hello to: <input type="text" ng-model="name">
    	<h2>Hello, { {name}} !</h2>
	</div>

	// the controller
	var HelloCtrl = function ($scope) {

	};

运行如上代码首次加载时，都是出现 *Hello, World!*，但是在用户通过`<input>`进行输入之后，`<h2>`中的显示改变了，跟随用户输入而变，这是因为 `ng-controller` 创建了子级的scope，虽然 `HelloCtrl` 什么初始化也没有做。

上面的例子中，用户输入通过 `ng-model` 进行了scope上面的name属性的**重写**，但是幸运的是，只影响到了我们事先想要影响的模版/视图区域。那这里有没有可能影响到 `<h1>`中的模版对象呢？答案是有的。如下，将 `ng-model`的引用指向父级scope中的name属性即可：

	<input type="text" ng-model="$parent.name">

虽然通过上面的方法，视图中三个使用了\{\{name\}\}模版的区域都得到了同步的更新，但是这个解决方案毕竟是脆弱的。因为通过 `ng-model` 这样的方法是建立在对于全局的DOM结构的假设的基础上的，如果在`<input>`上动态地添加了一个新的DOM节点，那么上面的 `$parent` 就会指向这个动态添加的（之前未曾料想到的）的节点，那么就会出现意料之外的效果。所以，**尽量减少使用 `$parent` 是王道呀。

那么既然上面的方法行不通，有没有什么别的方法可行呢？答案还是：有的。那就是让 `ng-model` 绑定到的变量是一个对象的属性，也就是说之前绑定的变量name是直接裸露在 `$scope` 的第一层，也就是 `$scope.name` ，那么这时候我们通过把 name存成**对象的属性**的形式，这样就可以大大提高安全性，而且达到目标效果。

	<body ng-app ng-init="thing = {name : 'World'}">
	<h1>Hello, { {thing.name} }</h1>
	<div ng-controller="HelloCtrl">
    	Say hello to: <input type="text" ng-model="thing.name">
    	<h2>Hello, { {thing.name} }!</h2>
	</div>
	</body>

以上原则是angular提倡的，即：*避免直接绑定到scope的属性，多加一层，绑定到**scope的属性的属性**是更好的方法*。

### **scopes领衔的事件系统**
scopes的组织架构（层次）图可以当作一辆bus来运载angular中的事件，那么也就有了事件bus。angular允许我们远着scope的层次传递自定义姓名的事件，可以往上（`$emit`），也可以往下（`$broadcast`）。

angular中的两大组成部分 `services` 和 `directives` 就是利用这个事件bus在应用中发送全局状态改变的消息。比如，我们可以监听从 `$rootScope` 广播出的 `$locationChangeSuccess` 事件，这样就可以知道URL是否改变，好做出相应的策略。

	$scope.$on("$locationChangeSuccess", function(event, newUrl, oldUrl) {
		// react on the location change here
		// for example, update breadcrumbs based on the newUrl
	})

关于事件的几个注意点：

- `$on` 方法是每个scope实例对象都可以调用的，并且为其注册一个事件处理器。这个事件处理器可以有三个参数，第一个是发起的事件，第二个和第三个是视具体的事件发出的消息而定
- 可以通过 `$emit` 或 `$broadcast` 发出事件
- 与传统的DOM事件一样，也可以使用 `preventDefault()` 和 `stopPropagation()` 方法作用于事件对象上。`stopPropagation()` 能阻止事件沿着scope层向上冒泡，但是它只对那些通过 `$emit` 发起的事件有效
- 除了以上两个函数一样，angular的事件机制和DOM事件机制完全是两个独立的个体，没有交集
- 在你确定想要使用自定义事件时，一定要斟酌再三，因为在angular里常常通过双向数据绑定就可以解决问题了
- angular中的内置事件：
	+ $emit: $includeContentRequested, $includeContentLoaded,  $viewContentLoaded
	+ `$broadcast`: $locationChangeStart,  $locationChangeSuccess,  $routeUpdate, $routeChangeStart,  $routeChangeSuccess,  $routeChangeError,  $destroy

### **scopes的生命周期**
scopes是用来提供独立的命名空间和避免变量名冲突的。scopes是有生命周期的，当其不再被需要时，可以被摧毁，这样通过scopes暴露的数据模型和UI操作逻辑也就可以被回收。

正如 `directives` 那一节所讲的，scopes通常是经过可以创建scope的指令进行创建，当然摧毁也一样。你也可以通过 `Scope.$new()` 和 `Scope.$destroy()` 这两个方法人工进行scopes的创建和摧毁。

### **View**
前面讲述了MVC模式中的Controller和Model，而且重点介绍了angular中用来暴露Model的 `$scope` 对象。接下来讲讲View，也就是视图对象。angular视图有如下几个特点：

- angular中的模版语言是HTML（视图可以看作是各种模版拼接起来的一个大模版）
- angular中的HTML语法是允许扩展的
- angular中的模版是允许局部更新的，而且这种局部更新完全是数据模型驱动的自动更新，无须手动干预
- angular应用的视图渲染过程分三步：
	+ 浏览器解析各个模版块中的文本及标准的HTML标签，生成DOM树
	+ DOM树生成后angular杀进来遍历这个已经被解析好的DOM结构
	+ 每当angular遇到一个directive，它就执行这个directive（指令）的逻辑将其动态地渲染成视图的一部分输出到屏幕

*声明式模版*

angular提倡声明式的方法来进行UI的构建。它意味着在实践中模版把焦点放在**叙述出**一个UI效果而非去关注如何实现这个效果上。也就是说，模版重点体现想要达到的视图效果，而背后实现的逻辑则是交给controller去完成。

书上的一个例子，罗里吧嗦一堆，其实就是个微博输入框的例子，字数限制，到最后几个字给提示，超过字数不给发送的一个 `<textarea>` 的表单。先看下第一个版本的实现代码：

	// the markup fragment
	<div><span>Remaining: { {remaining()} }</span></div>
	<div class="container" ng-controller="TextAreaWithLimitCtrl">
		<div class="row">
			<textarea ng-model="message">{ {message} }</textarea>
		</div>
		<div class="row">
			<button ng-click="send()">Send</button>
			<button ng-click="clear()">Clear</button>
		</div>
	</div>

	// the controller
	var TextAreaWithLimitCtrl = function ($scope) {
		$scope.message = "";
		$scope.remaining = function () {
			return MAX_LEN - $scope.message.length;
		}
		$scope.hasValidLength = function() {
			if ($scope.message.length <= MAX_LEN)
				return true;
		}
	}

基本版本完成，记住：`ng-model` 不能完成数据准备，它赋值的右端的那个变量也需要通过在 `ng-init` 或者controller里面准备好。接下来是要将发送按钮失活，代码如下：

	<button ng-disabled="!hasValidLength()" ng-click="send()">Send</button>

通过以上代码中 `remaining()` 和 `hasValidLength()` 的使用我们可以看出，要去进行UI操作，我们只需要去动模版中的一小部分，直接通过指令去描述一个想要的效果，这也就更加证实了angular的模版更关注描述效果，而把关注效果实现的部分丢到后面去解决。虽然我们在HTML标签中嵌入了代码，但是这个代码和js代码的耦合是隐式的，是通过注册在scope下的数据模型进行连接的，但是在实际的js代码维护中，我们完全不用去自己建立指向DOM元素的引用以及直接操作DOM，这样就在js代码中省去了很多麻烦。**以上两个方法的状态变化，都源自数据模型 `message` 的值的变化，进而影响UI中的效果**。接下来的一个数字倒数的显示效果的操作也是一样（代码在第一版本基础上进行修改）：

	<div><span ng-class="{'text-warning' : shouldWarn()}">Remaining: {\{remaining()}\}</span></div>

	// code should be added to TextAreaWithLimitCtrl
	$scope.shouldWarn = function () {
		return $scope.remaining() < WARN_THRESHOLD;
	}

`ng-class` 是能够直接改变css类选择器的，那么上面的代码直接可以看出：

 `message` 影响 `remaining()` 影响 `shouldWarn()` 影响 `ng-class` 最终影响 `UI效果`

 在上述过程中，这个 `<span>` 新添加了css类 `text-warning` 以至于UI重绘，但是整个过程在 `TextAreaWithLimitCtrl` 的js代码中完全没有见到DOM操作。如果用户或是代码协作者看这个模版和controller的话，完全就可以理解为：“哇操，太牛B了，在HTML中说了一声‘我想要得到警告提示’，然后UI就重新绘制了，而且js代码中完全没有对DOM元素的操作，擦嘞，angular太牛了，直接把这部分工作给我做了”。

 *场景比较*

1. 有ng-class：我们是通过它说`text-warning`这个css类需要被加到`<span>`中，这样每次用户输入字符时就会有警告提示
2. 无ng-class：也就是传统的实现方式，不用angular的情况。我们是这样做的，当用户输入一个字符了，而且总的字符数超出限制了，那好，我想找到那个用来做提示的`<span>`标签，然后给它加上一个css类叫`text-warning`，然后做出相应的显示

可以看到，上述两种场景是用angular（1）和不用angular（2）的区别。显然它们几乎是一个相反的过程。

- 用到angular的场景1相当于自动已经在`<span>`上安装了一个陷阱，等着用户输入触发的事件往里跳，DOM操作都交给angular了，我们只要关注想要什么效果，然后说一声“我们想要有警告提示”，然后就有警告提示了。
- 而没有用angular的场景2，则是要每次都要去找到那个`<span>`，然后给它加css类，然后改变显示，相当于一心二用，既要关心要加的效果，又要关心效果怎么加上去（即取到那个DOM元素的操作），而后一步，就是多出来的DOM操作的逻辑，在js代码中这样就会显得没有angular来的利索。

如果用专业术语来讲：场景1就是所谓的**declarative approach**，而场景2则是所谓的**imperative approach**。显然，场景1仿佛就只要说一句话，效果就分分钟的事，就好像是在说：*“亲爱的angular先生，当我的数据模型以xxx状态结束时，我想让我的UI看起来像这个样子。所以现在你去给我搞定它，怎么搞/何时搞我可不管。就这么简单。”*

显然，场景1（声明式编程）显得更具语义化且更具表现力，它使程序员可以摆脱去写大量重复的且需要复杂控制的更底层的指令和代码。声明式编程最后的代码非常简洁，而且易读。当然，这就需要有能够正确解析这些更高层的指令（来自声明式）的机制，这里，angular就是这样的机制。

另外，场景2（命令式编程）也有它自己的优点，那就是可以控制全局而且在微调和优化上更容易操作。虽然获得了更高的控制权，但是代价也是巨大的，那就是写一大堆底层的重复的代码。

其实，熟悉SQL的人也知道，它也是一种声明式编程。就是简单声明一个我想要的结果，然后让数据库去搞定给我送回来，就是这么简单。

angular模版中的指令声明式地表达自己想要的效果，这样我们就可以从一步一步地写具体的DOM操作指令中解放出来。而jQuery恰恰相反，它正是那种去一步步进行DOM操作，一个属性一个属性的去修改DOM元素上的属性，最后达到想要的效果。

需要注意的是：angular虽然在模版编程中大力提倡使用声明式的，但是在js代码中，它提倡的命令式编程，这就包括用命令式一步步去完成controller和业务逻辑的代码（但肯定不包括DOM操作）。当然，**唯一的特例**，也就是说在代码中能有DOM操作的就是在directives中。但是无论何时，永远不要在controller中进行DOM操作。当然，我们需要明白和理解angular底层是如何去完成这些我们交给他们的操作的，这样我们能在遇到性能问题的时候，能够给angular更多的提示和解决方案。
