---
title: 示例目录组织概览
date: 2013-11-11
category: build
layout: page
modifiedOn: 2013-11-20
---

## angular-app目录组织结构
最后，让我们来概览一下angular-app的整个项目结构吧。

	angular-app
		client 										-- 应用前端代码文件夹
			src 									-- 项目主要业务逻辑的js和html代码都在这里
				app 								-- 项目特定页面的html和js放在这里
					admin 							-- 管理模块逻辑代码
						projects 					-- 项目管理逻辑的js代码
						users 						-- 用户管理逻辑的js代码
						admin.js 					-- 声明整个项目的admin模块，依赖于上面两个模块
					dashboard 						-- dashboard模块
						dashboard.js 				-- 模块声明
						dashboard.tpl.html  		-- 模块模版
					projects 						-- projects模块
						projects.js 				-- 模块声明
						projects-list.tpl.html  	-- 
						sprints 					-- sprints模块
							sprints.js 				-- 模块声明
							sprints-list.tpl.html 	-- 
							sprints-edit.tpl.html   -- 
							tasks					-- tasks模块
								tasks.js 			-- 模块声明
								tasks-list.tpl.html -- 
								tasks-edit.tpl.html --
					projectsinfo 					-- projectsinfo模块
						projectsinfo.js 			-- 模块声明
						list.tpl.html 				-- 
					app.js 							-- 依赖于以上声明的那么多模块，作为唯一往外暴露的入口点
					header.tpl.html 				--
					notification.tpl.html 			-- 
				assets 								-- 存放项目所用到的资源
					img 							-- 图片资源
				common  							-- 项目用到的脚本
					directives 						-- 指令集
						gravatar.js 				--
						modal.js 					--
						crud 						--
							crud.js 				--
							crudButton.js 			-- 
							edit.js 				-- 
					services 						-- 服务集
						breadcrumbs.js 				--
						crud.js 					--
						crudRouteProvider.js 		--
						exceptionHandler.js 		--
						httpRequestTracker.js 		--
						i18nNotification.js 		--
						localizedMessages.js 		--
						notifications.js 			--
					resources 						-- 资源集
						backlog.js 					--
						projects.js 				--
						sprints.js 					--
						tasks.js 					--
						users.js 					--
					security 						-- 安全集
						login 						-- 登录模块
							form.tpl.html 			--
							login.js 				--
							LoginFormController.js  --
							toolbar.js 				--
							toolbar.tpl.html 		--
						authorization.js 			--
						index.js 					--
						interceptor.js 				--
						retryQueue.js 				--
						security.js 				--
				less                                -- 存放less预处理器的变量
					stylesheet.less 				--
					variables.less 					--
				index.html 							-- 应用唯一入口
			test 									-- 测试代码
				config 								--
				unit                                --
				vendor               				--
			vendor                                  -- 第三方依赖
				angular 							--
				angular-ui 							--
				bootstrap 							--
				jquery 								--
				mongolab 							--
			node_modules 							-- npm安装后出现的文件夹（开发的依赖包）
				grunt等 							--
			dist 									-- 打包后产生的文件夹，可以用作部署
				// 一堆打包好的html和js 			--
			gruntFile.js 							-- 打包用的grunt配置
			package.json 							-- npm会根据这里面的列表安装开发依赖包

		server 										-- 服务器端代码
			cert 									-- 证书相关文件
			lib 									-- 第三方类库
				routes 								-- 页面切换
				initDB.js 							-- 初始化MongoDB
				mongo-proxy.js 						-- 
				mongo-strategy.js 					-- 
				projectJSON.js 						--
				security.js							--
				xsrf.js								--
			test 									-- 测试相关
				mongo-initdb.js 					--
				mongo-proxy.js 						--
				mongo-strategy.js 					--
				security.js 						--
			node_modules 							-- npm安装后出现的文件夹
				express 							-- 
				grunt 								--
				passport 							--
				open 								--
				rewire 								--
			config.js 								--
			gruntFile.js 							--
			initDB.js 								--
			package.json 							--
			server.js 								--

		LICENSE 									-- MIT认证
		README.md 									-- 项目指南
