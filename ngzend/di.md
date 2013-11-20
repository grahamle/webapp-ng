---
title: 模块依赖
layout: page
category: ngzend
date: 2013-11-06
modifiedOn: 2013-11-20
---

## modules间的依赖
如果说协作对象之间的相互依赖（object dependencies）是angular中的intra依赖体系，那么modules之间的依赖就是属于inter依赖体系。modules之间的依赖粒度更大，但依赖程度视具体的模块之间的功能决定。

你可以把单个的module理解成是许多个service组合起来的一个小库，暂且称它为`service library`（服务集合仓库）。那么一个module，可能需要依赖别的仓库集，也可能不需要。以我们之前推送消息的例子来看，我们可以将notifications和archive剥离出来，让它们单独成为module（当然这个有点小题大做，只为了展示效果而已）：

	angular.module("application", ["notifications", "archive"])

像上面的这种方式，每个service或是每个services集合都能够成为可重用的实体，也就是一个module。然后有一个最顶层级（应用层级的）的module，在上面的例子中就是`application`了，它可以声明自己都依赖于哪些别的模块。当然，别的模块也可以声明自己依赖于哪些别的模块或是它自己的子模块，这样，模块也就构成了一个父子级别的层级架构。

综上所述，我们在处理angular的模块时，我们需要考虑两个独立却有有联系的分层：

- modules hierarchy（模块层）
- services hierarchy（服务层）

有了这样两个分层结构，就会出现很多有趣的问题：

1. 定义在模块`modA`下的服务`serviceA`能否依赖于定义在模块`modB`下的服务？
2. 有父子两个模块，`modP`是`modC`的父模块，那么在定义在子模块`modC`下的服务能否依赖于定义在父模块`modP`中的服务？还是说只能依赖于同级子模块下的服务？
3. 是否能够定义只对某个特定模块可见的私有服务？
4. 在不同模块下的某些服务可以有相同的名称么？

### **services及其跨模块间的可见性**
子级模块中的services是可以被父级模块中的services访问到并引入作为依赖的。看下面一段代码：

	// 应用级（父级）模块：app
	angular.module("app", ["engines"])
		.factory("car", function ($log, dieselEngine) {
			return {
				start: function() {
					$log.info("Starting " + dieselEngine.type);
				}
			};
		});
	//子级模块：engines
	angular.module("engines", [])
		.factory("dieselEngine", function() {
			return {
				type: "diesel"
			};
		});

由于`app`模块声明了自己依赖`engines`模块，所以`app`模块下的`car`服务的第二个参数（实例化）可以注入`dieselEngine`的**一个实例**。

另外**同级模块之间的服务，也是相互可见的**。把上面的代码做一个小小的改动：把`car`服务剥离出来成为一个单独的模块，然后改写一下模块层的依赖，如下：

	angular.module("app", ["engines", "cars"])

	angular.module("engines", [])
		.factory("dieselEngine", function() {
			return {
				type: "diesel"
			};
		});

	angular.module("cars", [])
		.factory("car", function($log, dieselEngine) {
			return {
				start: function() {
					$log.info("Starting " + dieselEngine.type);
				}
			};
		});

综上所述，一句话：**定义在一个模块中的服务，对于别的模块，总是可见的**。模块的分层并不影响服务之间的通讯和可访问性。这样的分层只是为了在逻辑上更为直观一些，在代码书写上更为清晰一些罢了。一个angular应用启动的时候，angular就把它所包含的所有服务（无论是模块内的还是模块间的）都整合起来到全局的命名空间下了。

如上所述，既然最后是整合到一个全局的命名空间，那么显然，**不同模块之间的*服务名*是不能相同的**，那样选择级别更高（优先级）的服务会重写另外一个同名服务的内容。看以下的例子：

	angular.module("app", ["engines", "cars"])
		.controller("AppCtrl", function($scope, car) {
			car.start();
		})
	// engines模块
	angular.module("engines", [])
		.factory("dieselEngine", function() {
			return {
				type: "diesel"
			};
		});
	// cars模块
	angular.module("cars", [])
		.factory("car", function($log, dieselEngine) {
			return {
				start: function() {
					$log.info("Starting " + dieselEngine.type);
				}
			};
		})
		// 直接在cars模块下又多添加了一个dieselEngine服务
		.factory("dieselEngine", function() {
			return {
				type: "custom diesel"
			}
		});

上面的代码跑起来的过程是这样的：

1. 首先，`app`模块启动，加载`engines`和`cars`两个模块
2. 然后，在应用级模块`app`下注册了一个控制器，用来初始化整个应用程序的数据模型及操作逻辑
3. 之后，进入控制器，控制器用第二个参数（构造函数）开始初始化对象，这时候控制器加载`cars`模块下的`car`服务，进入代码内部，启动`car`服务，调用其`start()`方法
4. 接着，`start()`方法的调用就需要执行`dieselEngine`服务取到相应的`type`，那这时候angular就需要决定是去取哪个`dieselEngine`服务了，因为在同级`cars`模块下有这个服务（属于模块内的调用），在跨模块间的`engines`模块也有
5. 最后，考虑到模块内的服务调用优先级肯定要高，所以自然最后在日志中log出来的内容是：**custom diesel**

最后，在目前的angular版本下，是不支持能够定义只对某个特定模块可见的私有服务。至此，之前的4个问题陆陆续续也就解答完毕了，我在原书的基础上写了自己的理解，自己去对照哪个问题在哪里解决吧，应该不难找的。

至于这时候肯定会有人问啦，为什么要在angular中使用modules？不要问我，接着看下去，书本的第二章。这也算是吊胃口的设计吧。我能说的就是分成各个小的单独的子模块，你不觉得逻辑更清晰么？而且对单元测试有益。虽然最后angular还是把所有的modules整合到一个大包里。