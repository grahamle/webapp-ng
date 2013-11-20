---
title: 模块与文件的对应关系
date: 2013-11-11
category: build
layout: page
modifiedOn: 2013-11-20
---

## angular中的module和文件对应
整个应用现在已经被很有序地组织好了，现在是时候看看各个文件里的具体内容，他们和angular的模块式怎么关联的。

### **一个文件，一个模块**
一般来讲modules和files的对应关系可能会有如下三种：

- A：多个angular模块在一个js文件中
- B：一个angular模块散落在多个js文件中定义
- C：一个angular模块严格对应一个js文件

理论上和实际上来看，无疑C选项是最好的，它的好处太多了，当然还包括在单元测试中可以单独加载每个模块，最多还能容忍的是放少部分几个有相似功能/性质的模块在同一个文件中，如几个模块构成一个总功能（或是模版中的一个小组件），但是绝对不允许把一个模块分布在多个文件中定义。

### **模块内部呢？**
一个文件里面声明好一个模块之后，那就可以往里头注册“菜谱”了，也就是各种各样的`providers`了，至于注册哪一种，当然是看要炒什么菜，就放什么样的“谱”了。

*往模块中注册多个providers的不同实现语法*

1. 定义一个变量用来作为返回的模块实例的引用，然后用这个变量来注册多个providers（或是controllers等），这其实就是利用一个中间变量来作为引用，但是这个方式的缺点很明显：这个变量是作为全局变量暴露在全局名称空间下的，这样我们就不得不采取一些别的补救策略，例如把整个module声明放到闭包里，单独声明一个命名空间之类的。不管怎么，来看看这种方法的实现，如下：
	
		var adminProjects = augular.module("admin-projects", []);

		// 第一个controller 
		adminProjects.controller("ProjectsListCtrl", function($scope) {
			// controller logic goes here
		});
		// 第二个controller
		adminProjects.controller("ProjectsEditCtrl", function($scope) {
			// controller logic goes here
		});

2. 理想情况下，我们可以直接把这个中间变量省去，如果你看了下面的代码，其实也没有多少优化，`angular.module("admin-projects")重复出现。代码重复大多数情况下都是不好的，比如，如果某天心血来潮要给module换个名字，尼玛，那你慢慢换吧。而且声明模块和引用模块的代码只差了一个`[]`的依赖包，很容易让人忽略这之间的细微差别导致别的错误。

		augular.module("admin-projects", []);

		angular.module("admin-projects").controller("ProjectsListCtrl", function($scope) {
			// controller logic goes here
		});

		angular.module("admin-projects").controller("ProjectsEditCtrl", function($scope) {
			// controller logic goes here
		});

3. 幸运的是，我们有最后一种方法可以解决以上所讲的种种尴尬，**链式调用**，直接看下下面的代码：

		angular.module("admin-projects", [])
			.controller("ProjectsListCtrl", function($scope) {
				// controller logic goes here
			})
			.controller("ProjectsEditCtrl", function($scope) {
				// controller logic goes here
			})
			.factory()
			.service();

*声明配置阶段与运行阶段的语法*

第一章中介绍过，启动一个angular应用时是被分成两个阶段，不记得回去巩固去。但是当时没跟大家透露说，每个模块可以有多个配置和运行的代码块，不一定只在一棵树上吊死呀，骚年。

angular支持两种不同的声明**配置代码块**（同时运行代码块也适用）的方式：

1. 作为`angular.module()`的第三个参数进行配置，这样子的配置函数只能注册一个配置块，而且由于多了第三个参数，这让整个module的声明看起来非常冗杂。

		angular.module("admin-projects", [], function() {
			// configuration logic goes here
		})

2. 第二种方式是通过在返回的module实例后通过链式调用串上去多个配置块：

		angular.module("admin-projects", [])
			.config(function() {
				// config block 1
			})
			.config(function() {
				// config block 2
			});