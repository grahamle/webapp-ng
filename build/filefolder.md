---
title: 文件目录结构
date: 2013-11-11
category: build
layout: page
modifiedOn: 2013-11-20
---

## 文件目录组织
刚刚clone下来了sample-app的项目，配了下MongoDB，分别用npm在client和server端把需要的依赖装上之后，用node把server跑起来，然后localhost可以用MongoDB暴露的用户登录，感觉很开森，虽然还不晓得怎么个过程，但至少知道了我们访问网站时的一个过程和实际上后面的实现过程完全不同：

- 用户表面上看到的过程：从前端到后台，再从后台去DB取数据的过程。
- 背后实现的过程：先有一个数据库，然后给数据库创建可以访问的用户，然后围绕这个DB进行server端的代码编写，然后跑起server来支持前端。

接下来就让我们来扒开它的衣服看看里面都有什么。

### **文件目录结构**
在最顶层，如下图所示，整个项目包括两个文件夹还有其他一些附加物：

![顶层结构](https://raw.github.com/grahamle/grahamle.github.com/master/media/image/angular-ebook/top-level.png)

- client文件夹：包括所有的客户端angular应用
- server文件夹：包括一个基本的基于express实现的webserver来支持整个前端
- README：可以说是标配了，说明这个项目怎么跑的一个说明说
- LICENSE：说明这个项目的营业执照，你懂的

再往里一层，如下图所示，整个client文件夹包括以下结构：

![client结构](https://raw.github.com/grahamle/grahamle.github.com/master/media/image/angular-ebook/sec-level.png)

- src文件夹：包括整个app的所有源代码
- test文件夹：包括整个app中协同的测试代码
- vendor文件夹：包括所有第三方依赖（这里的依赖和后面的node_modules不同，这里的是js类库的依赖，如angular-ui之类的）
- build文件夹：包括所有的编译脚本（但是最新版本中已经没有这个文件夹了，目前尚不知整合到哪里了）
- dist文件夹：包含了整个项目的编译结果（打包结果），可以作为发布版本（这里所说的编译和传统的不同，因为大家知道js是解释性语言，所以这里编译只是借用这个词，强调的是把所有js，css等文件整合压缩等的最后过程，可以理解为打包）
- gruntFile.js：这个可能就是吸收了原来的build文件夹和原来的grunt.js之后整合的用来做打包操作的命令执行脚本
- package.js：包含了所有npm（node包依赖管理工具）要安装打包要依赖的工具时的所有工具集
- node_modules文件夹：包含了`npm install`之后根据`package.js`列的清单安装的所有打包工具依赖集

现在让我们再进一层，进到src文件夹探个究竟：

![src结构](https://raw.github.com/grahamle/grahamle.github.com/master/media/image/angular-ebook/src.png)

- index.html：这个是整个应用程序的入口点了
- app文件夹和common文件夹：包括与angular相关的代码
- assets和less文件夹：包括与angular无关的内容；assets主要放图片等资源，而less主要放的是less预处理器用到的变量。需要注意的是，为推特的bootstrap样式而准备的LESS模版是放在了上一层的vendor文件夹（即第三方依赖）下

src文件夹里面的东西，大部分是脚本和HTML，那么怎么来组织这么多的文件呢，按它们的特点来，还是按结构层次来，还是按文件类型来呢？这里面在src文件夹里的各个文件夹，我们采取混合组织的一种策略，主要先把为了相同/相似功能服务的文件组织到一块，来来来，让我们打开app文件夹看看：

![app结构](https://raw.github.com/grahamle/grahamle.github.com/master/media/image/angular-ebook/app.png)

- admin文件夹：主要负责admin账户的逻辑
- dashboard文件夹：负责产生的项目管理面板的逻辑
- projects文件夹：负责项目的所有相关逻辑
- projectsinfo文件夹：负责项目缩略信息的逻辑
- app.js：整个项目angular部分的模块划分及路由选择，项目总的js逻辑
- header.tpl.html：项目header模版
- notifications.tpl.html：项目通知系统模版

接着是common文件夹，主要是存放angular应用的别的通用部件：

![common结构](https://raw.github.com/grahamle/grahamle.github.com/master/media/image/angular-ebook/common.png)

- directives文件夹：包括所有的指令
- resources文件夹：目前还不晓得干嘛的，全部是js文件
- security文件夹：用于登录及认证等机制，与MongoDB有通信
- services文件夹：用于存放公用服务

至于src文件夹下的另外两个子文件夹：assets和less文件夹，里面就是放图片资源和less变量，就不用细看。这样，看完了src下的一层又一层，跟洋葱似的，接着来看看和src同级的test文件夹都有什么好料：

![test结构](https://raw.github.com/grahamle/grahamle.github.com/master/media/image/angular-ebook/test.png)

- config文件夹：对于配置的测试
- unit文件夹：做单元测试的
- vendor文件夹：这个命名个人感觉不好

紧接着是vendor文件夹，这里就不po截图了，简单来看看在开发时（写js代码时）都需要调用哪些第三方的API：

- angular库：包括angular.js和angular-route.js
- angular-ui库：存放为bootstrap准备的angular-ui
- bootstrap库：存放LESS模版
- jquery库：jquery.js
- mongolab库：存放mongoDB的API

而最后看一下dist文件夹，它是最后用grunt工具编译（打包）之后生成的可以用来发布的版本，也就是说用`node server.js`跑起来后，通过`localhost`访问到的就是最后的这个文件夹里面的`index.html`为入口点的angular-app。

### **罗马非一日之功也**
上面描述了一个很理想的目录结构，但是非大牛者，不能为也，不可能像本小菜这样的能够一下子就在项目还没开始的时候把目录结构统统给搞好了，然后好吧，你们就往里面填东西得了，不是这样子的，本书作者也阐述了这一观点。**一个项目总是开始于一个很简单的结构的，然后一小步一小步往最后的愿景去靠**。

### **文件命名惯例**
对于那么多文件夹那么多目录下的那么多文件，文件命名是一个大的挑战，最好在总体上有个一致性，可以试着考虑使用二级后缀。如模版文件可以用`.tpl.html`为结尾。