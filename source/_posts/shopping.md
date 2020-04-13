title: Shopping+在线购物商城的实现（Spring+SpringMVC+Hibernate）
tags:
  - Spring
  - SpringMVC
  - 购物商城
categories: 项目实战
date: 2019-01-15 20:23:14
---

## 前言
&nbsp;&nbsp;&nbsp;&nbsp;虽说之前我想要把自己做过的项目一一写成博客放出来，可是之前因为CSDN手机号验证什么的真的让我对它的不满爆发了，所以这么久以来没有写新的博客，因为我自己搭的博客还在建设中。但是既然之前说过要放项目，那么也不能食言，就把我做过的几个项目简单放上来吧，不做太多的解释。不废话了。
## 项目功能介绍
&nbsp;&nbsp;&nbsp;&nbsp;本项目是一个在线购物商城，但是请注意，此网站是单商家，也就说这里一个商家可以添加多种商品，而不是像淘宝一样可以有多个商家。而且本项目也仅供学习交流，禁止用作任何商业用途。

## 主要功能
### 普通用户：
&nbsp;&nbsp;&nbsp;&nbsp;1.登录、注册功能  
&nbsp;&nbsp;&nbsp;&nbsp;2.浏览商品功能  
&nbsp;&nbsp;&nbsp;&nbsp;3.搜索商品功能  
&nbsp;&nbsp;&nbsp;&nbsp;4.查看商品详情  
&nbsp;&nbsp;&nbsp;&nbsp;5.添加购物车  
&nbsp;&nbsp;&nbsp;&nbsp;6.购买功能（在商品详情页单独购买或在购物车批量购买）  
&nbsp;&nbsp;&nbsp;&nbsp;7.查看订单状态  
&nbsp;&nbsp;&nbsp;&nbsp;9.确认收货功能  
&nbsp;&nbsp;&nbsp;&nbsp;10.评价已购买商品功能  
### 管理员：
&nbsp;&nbsp;&nbsp;&nbsp;1.拥有普通用户所有功能  
&nbsp;&nbsp;&nbsp;&nbsp;2.查看、删除所有用户功能  
&nbsp;&nbsp;&nbsp;&nbsp;3.查看、删除所有商品功能  
&nbsp;&nbsp;&nbsp;&nbsp;4.添加新的商品功能  
&nbsp;&nbsp;&nbsp;&nbsp;5.处理订单功能  
&nbsp;&nbsp;&nbsp;&nbsp;6发货功能  

## 项目主要技术与工具
### 开发工具及环境
&nbsp;&nbsp;&nbsp;&nbsp;IDE是Intellij IDEA，clone或者下载之后可以直接用IJ打开项目  
&nbsp;&nbsp;&nbsp;&nbsp;运行环境是Tomcat 9.0可以根据自己情况配置，7.0以上应该都行  
&nbsp;&nbsp;&nbsp;&nbsp;项目版本管理使用了Maven工具，类型也为Maven项目  
&nbsp;&nbsp;&nbsp;&nbsp;数据库使用了MySQL
### 开发技术
&nbsp;&nbsp;&nbsp;&nbsp;后端框架为Spring+SpringMVC+Hibernate，注解形式  
&nbsp;&nbsp;&nbsp;&nbsp;前端框架为Bootstrap，使用插件有Layer.js等  
&nbsp;&nbsp;&nbsp;&nbsp;前后端交互方式为ajax，使用json格式传递数据

## 运行截图
![主页](http://img.icedsoul.cn/img/blog/shopping/main_page.png)
![商品详情页](http://img.icedsoul.cn/img/blog/shopping/product_detail.png)
![购物车](http://img.icedsoul.cn/img/blog/shopping/shopping_car.png)
![控制页面](http://img.icedsoul.cn/img/blog/shopping/control_page1.png)
![控制页面](http://img.icedsoul.cn/img/blog/shopping/control_page2.png)
![订单状态](http://img.icedsoul.cn/img/blog/shopping/order_page.png)
![订单处理](http://img.icedsoul.cn/img/blog/shopping/order_handle.png)
![搜索](http://img.icedsoul.cn/img/blog/shopping/search_page.png)

## 注意
&nbsp;&nbsp;&nbsp;&nbsp;本项目仅用于学习交流，安全性极差，严禁用于除学习交流外其它用途，代码本身此处不做解释，我会在代码里面添加必要的注释。  
&nbsp;&nbsp;&nbsp;&nbsp;项目下载之后需要修改相应的内容，如果完全没有开发基础请不要问我“为什么我下载了你的项目跑不起来？”“为什么我下载了你的项目运行报错了？”之类的问题，JavaWeb开发本身就比较依赖各种环境，要么你把你的环境改的和我用的一样，要么你把项目相关配置做改动使其能在你的环境下运行。没基础请先去学习相关的基础知识，然后再来了解此项目。个人精力有限，无法帮每个新手解决这种问题，我会尽量在注释里面说清楚要改哪些内容。  
&nbsp;&nbsp;&nbsp;&nbsp;**项目对应MySQL数据库需要自己根据resources/properties路径下SQL文件创建**
## 在线演示地址
[点我](http://119.23.212.211:8080/Shopping/) 服务器未过期可以访问，不保证一直可以访问。
## 源码下载
GitHub：
https://github.com/IcedSoul/Shopping

### 如果对你有所帮助，Github star一下让小透明开心下呗~
