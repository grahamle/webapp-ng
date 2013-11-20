---
title: 示例应用介绍
date: 2013-11-11
category: build
layout: page
modifiedOn: 2013-11-20
---

## 示例应用介绍
示例应用在GitHub上的[托管地址](https://github.com/angular-app/angular-app)，可以先抓一份下来看看目录结构。这里顺便说下，个人认为**技术选型**和**文件目录组织**同等级别，技术路线确定后马上要考虑的就是文件目录组织形式了。

### **问题域**
利用SCRUM方法去管理项目。当然，我们的这个sample是属于小型的项目，不会说去满足SCRUM的所有方面。以下是我们即将完成的app的屏幕截图：

![app-screen-shot](https://raw.github.com/grahamle/grahamle.github.com/master/media/image/angular-ebook/app-screenshot.png)

从上面的截图我们可以看到这个sample-app是一个典型的CRUD的web应用，它包括以下几个功能：

- 获取、展示及编辑从一个永久存储的数据库里传来的数据
- 认证及授权
- 相对简单的导航，包括应该有的UI元素，如菜单栏，提示信息等

### **技术栈**
这里我们选用的各项技术都是基于js的，这样才能使其融成一个整体。

*持久性存储*

选用基于文档的存储策略的MongoDB数据库，它是一种所谓的最近很火的NoSQL存储。它很好地契合了js开发环境下的存储需求：

- 文档是被存储于JSON风格的数据格式中
- 查询和数据操作可以通过js及相关API完成
- 可以将数据以REST的形式提供给客户端

网上能够找到的基于云存储策略的数据库托管服务，而且要是免费的，推荐[MongoLab](https://mongolab.com)，易上手，而且小于500MB以下的空间是免费提供的。这对于我们这个sample是足够了。MongoLab有一个不能忽略的优势：**它是通过一组REST接口来暴露数据库的**。我们将会使用这一组API来看看angular是如何与提供REST服务的节点进行通讯的。要使用这个服务，只需到网上注册一下，so easy。

*服务器端开发环境*

- 简单情景：如果只是简单的小项目，你完全可以用MongoLab提供的REST的API让angular直接从托管在MongoLab上的数据库上抓数据，没有任何问题。
- 复杂情景：但是如果项目规模大了，需要考虑安全性等问题时，上面的简单粗暴直接的方式显然是不行的。所以，在实践中，angular写的UI常常是通过与某种后台通信，然后通过它去取MongoLab上的数据。这就是所谓的“中间件”可以提供的功能：安全服务（认证）及判定权限（授权）。

显然，我们的sample-app虽然简单，但是也需要有一个简单粗暴的后台，这里我们用node.js作为我们的后台解决方案。node.js不是我们这本书要讲的内容，我们只需要了解基础的以及关于npm的一些内容即可，因为关键是我们项目中不会直接去用原生的node.js去写一个server，我们会用到相关的node.js的库去搭建我们的服务器端（所谓的后台）的开发环境，典型的`node.js`库有：

- [Express](http://expressjs.com/)：作为服务器端web-app框架来提供路由服务、数据服务及静态资源
- [Passport](http://passportjs.org)：作为`node.js`的安全中间件
- [Restler](https://github.com/danwrong/restler)：作为`node.js`的HTTP客户端库

这一节其实是为我们开始学习后端提供一个引子，尤其是我自己，所以**重点mark**。

*第三方js类库*

angular完全可以独自使用，但是如果有现成的轮子呀，车轱辘，那干嘛还需要再用angular再闭门造一次车呢？对吧？所以，项目中我们会教你如何使用在angular中去用jQuery，`underscore.js`等第三方类库，着重是`jQuery`。

*css库*

用css类库来将我们的UI进行样式化会省去我们裸写一大堆css代码的麻烦，当然，angular并没有强行指定需要使用具体哪个css类库。项目中，我们会使用[Twitter Bootstrap](http://twitter.github.com/bootstrap/)这个类库。同时，我们引进了`LESS`这个css预处理器。