---
title: ng中模块的生命周期
layout: page
category: ngzend
date: 2013-11-06
modifiedOn: 2013-11-20
---

## angular的modules的生命周期
前面两节讲了angular中的module以及在其下面注册了各种各样的services，最后可以通过DI机制将所有协作对象整合到一块，成为一个非常灵活的app。这一切都是发生在modules命名空间之下的。那么modules的整个生命周期是怎么样的呢？

从上面services中的最后一个`provider()`方法的实现我们可以看到angular为了更好的支持providers，将modules的生命周期划成两段：

1. 配置阶段：所有的“配方”（食谱）被集齐并进行配置的阶段
2. 运行阶段：完成所有的初始化生产出合格的对象实例的阶段

让我们具体来看看这两个阶段具体是干啥的。

### **配置阶段**
providers只能在配置阶段进行配置。你可以这么想把，你总不可能把煮好的东西再拿去换个“食谱”再重新弄吧？只能是把“食谱”搞定好，才开始炒菜呀啥的，然后搞定。除了上一节中讲的可以在`provider()`之中进行配置，还可以利用单独的一个用于对各种各样的services进行配置管理的API（`config()`)：

	myMod.config(function(notificationsServiceProvider) {
		notificationsServiceProvider.setMaxLen(5);
	})

从上面可以看到，对于该API就是给一个函数对象作为参数进行配置，而函数对象中的参数则是**服务名+服务类型名**，以服务类型为后缀结尾，以示你是要对哪一种服务进行配置，而在函数对象的逻辑中，则可以调用具体的服务里面的已有的可对外暴露的配置相关的方法，如上面例子中的`setMaxLen()`就是属于`provider()`中的。

最后提示下，通过`config()`方法进行配置，是我们在真正进入初始化要产生对象实例之前最后一次机会对具体要产生对象的构造函数进行怎么变化的最后机会。

### **运行阶段**
`run()`方法允许我们告诉angular，我想让我的app在启动时跑起来哪些东西。你可以把它当作是C语言中的main()函数，但是又不尽相同，因为angular允许我们可以有多个配置和运行块，**也就是说有多个函数入口点**。

假设我们需要在app运行时显示程序运行倒计时，我们可以这样做：

	angular.module("upTimeApp", []).run(function($rootScope) {
		$rootScope.appStarted = new Date();
	})

	// 在模版中使用上面注册的属性
	Application started at: { {appStarted} }

上面这个例子其实我是没有看出有哪里解释到了`run()`，反正我们知道这个方法会让一个service跑起来，之后找到更好的例子再补上。

### **不同的服务注册方法在两阶段中的差异**

<table width="800px">
	<tr text-align="center">
		<th width="200px">方法</th>
		<th width="200px">注册了啥</th>
		<th width="200px">配置阶段可注入否</th>
		<th width="200px">运行阶段可注入否</th>
	</tr>
	<tr text-align="center">
		<td width="200px">Constant</td>
		<td width="200px">常数值</td>
		<td width="200px">可以</td>
		<td width="200px">可以</td>
	</tr>
	<tr text-align="center">
		<td width="200px">Value</td>
		<td width="200px">变量值</td>
		<td width="200px">不能</td>
		<td width="200px">可以</td>
	</tr>
	<tr text-align="center">
		<td width="200px">Service</td>
		<td width="200px">构造函数创建的新对象</td>
		<td width="200px">不能</td>
		<td width="200px">可以</td>
	</tr>
	<tr text-align="center">
		<td width="200px">Factory</td>
		<td width="200px">factory函数返回的新对象</td>
		<td width="200px">不能</td>
		<td width="200px">可以</td>
	</tr>
	<tr text-align="center">
		<td width="200px">Provider</td>
		<td width="200px">$get函数返回的新对象</td>
		<td width="200px">可以</td>
		<td width="200px">不能</td>
	</tr>
</table>