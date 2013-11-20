---
title: ng.module()
layout: page
category: ngzend
date: 2013-11-06
modifiedOn: 2013-11-20
---

## angular的modules
机智如我，早已发现之前的几个例子全是在全局下定义controller。但咱都知道滴，全局状态不可以多用呀，必然杀敌一千，自损八百呀，有木有。它不仅伤害到了应用的结构，让代码更难维护、测试、阅读，更重要的是苦了程序猿。angular中，提供了一系列API用来定义模块（modules）并且让我们可以在模块上注册（添加）各种各样的对象。

### **angular.module()**

让我们把之前的`HelloCtrl`变下形看看：

	angular.module("hello", [])
	  .controller("HelloCtrl", function($scope) {
	  	$scope.name = "World";
	  })

从上面短短的几行代码我们来分析看看：

1. angular本身定义了一个全局的命名空间，就叫 `angular`
2. 在`angular`这个命名空间下游很多实用工具和函数
3. `module`就是`angular`中的一个函数，它的主要作用是作为angular其他对象（如controllers，services等）的一个容器，可以理解为二级命名空间。定义一个module需要两个参数，第一个作为module的名字，第二个则是指出这个module都依赖哪些别的modules
4. `angular.module()`方法的调用返回一个新创建的module实例，而我们定义控制器就是在这个返回的实例中进行的
5. 在返回的module实例中直接调用其`controller()`方法，提供两个参数：一个是控制器名，另一个则是构造函数

经过以上，一个定义了控制器的module就准备好了，那么要怎么告诉具体的HTML页我这个页用的是哪个module呢？答案如下：

	<body ng-app="hello">

这里，你可以将`ng-app`理解为它是使整个angular应用启动的指令，同时也声明了命名空间。

### **angular的依赖注入**
我们很早之前就把`$scope`当作controller中的构造函数的参数进行使用了，但是到底是什么机制可以那么神奇地让我们在没有任何地方定义到这个东西，但是却可以使用呢？之前也讲过，ng-controller时就触发了`Scope.$new`创建了一个新的scope实例，但是它是怎么被塞进来的呢？因为从头到尾，我们其实就只做了一件事：“要让我的函数正常运行，我需要依赖$scope，所以，angular，给我搞一个来，不管你怎么弄。”

通过上面简短的介绍，之所以我们能够获得想要的依赖（即：**协作对象**），是因为angular内置了依赖注入引擎，它可以实现下面的操作：

- 知悉某个对象有对别的协作对象的依赖的需求
- 找到需要的协作对象
- 将发起需求的对象和协作对象绑到一块组成一个具有完整功能的应用

以上几点基本展示了声明式表达依赖的作用，它可以让各个协作对象不用去考虑其他对象的生命周期，有时候进行一些swap，替换等，又可以创造新的应用了，而且这也是angular能高效完成单元测试的原因。（*以上这句感觉跟扯淡似的，也许是因为还没理解到精髓*）。

### **DI的好处**
这一小节事实上写了个小demo，讲**推送消息的**，如何用好DI，现在还不太理解，只好先作罢，不翻。
to be continued ...

## angular的services
上一节的讲的内容是我觉得到目前为止这本书最无厘头的地方，既没有出现完整的示例，也没有给人予清晰的感觉，完全就像是凭空造物，更应该放在讲完services之后出现。现在让我们来讲讲angular中的services。

由于angular只能将其能够明白的对象进行组装/连接，所以在能够做DI之前，我们首先需要将要用的对象注册到angular的module中。另外，强调了一个**注册的对象不是直接注册对象的实例，而是丢一个创建对象的谱**，这个讲的跟传世剑谱似的，暂时没有想到破解的招，得等理解了$injector和$provide之后再来回顾。

angular中可以看作是services的有好几种：values、services、factories、constants、providers

### **Values**
最简单的注册一个对象的方式就是用`value()`了，而且它注册的事一个*预先初始化*的对象。下面来看一下推送消息这个例子中的第一个service：

	var myMod = angular.module("myMod", []);
	myMod.value("notificationsArchive", new NotificationsArchive());

通过上面的代码，我们知道，任何要通过DI机制管理的service都需要有自己唯一的名称。这也是`value()`中的第一个参数，第二个参数则是创建**菜谱**的一个函数或是对象。

通过`value()`这个方法注册的对象不能依赖别的对象。这个方法本身就是用来注册极简单的对象的。

### **Services**
对于推送消息的这个例子中的第二个service，由于它依赖于上面用`value()`创建的archive-service，所以它不能用`value()`方法创建，而且它是要将一个依赖于别的对象的构造函数注册为service，所以可以采用`service()`的方式：

	// myMode is the module we created above
	myMod.service("notificationsService", NotificationsService);

	//NotificationsService is a constructor function
	var NotificationsService = function (notificationsArchive) {
		this.notificationsArchive = notificationsArchive;
	}

明眼人一眼就看到了用`value()`和`service()`时的第二个参数发生了不同。那就是后者的第二个参数怎么不用`new`关键字了？**因为这个service使用了angular的DI机制来处理依赖，所以它不用考虑依赖的初始化也可以接受archive-service了**。

在实践中，`service()`方法并不常用，只有在要用来注册已经存在的构造函数时它才显得比较便利。

另外，需要注意：*service这个词有两个含义：一个是指`service()`方法，一个是泛指包括所有通过DI机制管理的angular提供的公用服务*。

### **Factories**
跟`service()`方法相比，`factory()`方法显得更加灵活，因为我们不单单可以注册预先已经存在的构造函数了，而且可以注册任何用来创建对象的函数（object-creating function），看下面的例子：

	// 采用factory方法实现和上一个例子一样的service
	myMod.factory("notificationsService", function(notificationsArchive) {
		
		var MAX_LEN = 10;
		var notifications = [];

		return {
			push: function (notification) {
				var notificationToArchive;
				var newLen = notifications.unshift(notification);

				// push方法可以依赖闭包作用域
				if (newLen > MAX_LEN) {
					notificationToArchive = this.notifications.pop();
					notificationsArchive.archive(notificationToArchive);
				}
			},
			// notificationsService的别的方法
		}

	})

如上面代码所示，`factory()`方法有以下几个特点：

- 会返回一个对象，它可以是任何有效的js对象，包括函数对象
- 它是angular中最常见的把对象融入到DI机制的方法
- 非常灵活，第二个参数，即创建对象的函数可以非常灵活
- 因为`factory()`也是普通的js函数，所以我们可以在其里面使用一个词法作用域（lexical scope）来模拟私有对象，这样我们就可以将service的具体实现细节隐藏起来，只把service的方法暴露出去提供用户使用

### **Constants**
上面经过`factory()`出厂的notificationsService已经很好用了：

- 和协作对象（notificationsArchive）解耦
- 隐藏了自己的私有状态和具体的实现细节

当然，还有一点可以做的更好，那就是其中的配置参数，如`MAX_LEN`等都是硬编码在其中的常数。angular对于这个，也有解救的办法，那就是把这些常数当作协作对象注册到module之下，然后就可以通过DI机制供别的对象使用了。这主要要归功于`constant()`方法：

	// 注册一个常数service
	myMod.constant("MAX_LEN", 10);

	// 修改factory service中的第二个参数
	function (notificationsArchive, MAX_LEN) {
		...
		// creation logic doesn't change
		// var MAX_LEN = 10;这一行硬编码就可以去掉了
	}

constants常常用来存储一些配置相关的service，可以方便用户按照自己的意愿进行配置。

### **Providers**
`provider()`方法是更具有一般性的，相比于之前的几个方法都有各自的用途，它是可以做到上面所有方法想要创建的service，可以理解为一个超集。下面是`notificationsService`用`provider()`来提供的一个示例：

	myMod.provider("notificationsService", function() {

		var config = {
			maxLen : 10
		};
		var notifications = [];

		return {
			setMaxLen : function(maxLen) {
				config.maxLen = maxLen || config.maxLen;
			}

			$get: function(notificationsArchive) {
				return {
					push : function(notification) {
						...
						if (newLen > config.maxLen) {
							...
						}
					},
					// notificationsService的别的方法
				}
			}
		}
	})

- `provider()`方法是一个必须返回包含`$get`属性的对象的方法。而`$get`属性又是一个factory方法，返回一个factory service的实例。我们可以把providers看作是一个将`factory()`方法内置在自己内部的`$get`属性中的对象。
- 另外，`provider()`相当于把配置相关的选项也集成在自己体内，可以在`factory()`方法启动之前先做好相关的配置。