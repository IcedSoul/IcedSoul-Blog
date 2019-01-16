title: ajaxFileUpload报错$.ajaxFileUpload is not a function解决方法
tags:
  - ajaxFileUpload
categories: 错误解决
date: 2019-01-15 19:44:20
---
## 报错信息

&emsp;&emsp;Uncaught TypeError:$.ajaxFileUpload is not a function  
&emsp;&emsp;大意就是，ajaxFileUpload这个函数未定义
## 错误背景
&emsp;&emsp;我使用了ajaxFileUpload这个js来实现不借助form表单的不刷新页面提交文件的功能（但是其实这个js内部还是用的是form表单提交的233，伪ajax，和jQuery ajax是不一样的）。  
&emsp;&emsp;在找了一个ajaxfileupload.js下载引用并且写好上传代码之后，一直报这个错误，我打开控制台看了一下，ajaxfileupload.js这个文件是引用成功的，可以在控制台打开，在其中可以看到ajaxFileUpload这个函数是定义了的，但是不管重启服务器多少次，都会报这个错误。然后开始各种查资料，开始看大家所说，以为是ajaxfileupload.js这个文件本身的问题，然后我去github下载了fork最多的那个，替换之前下的那个，还是不对，又在不同的几个地方找了几个不同版本的ajaxfileupload.js都试了试，仍旧是这个错误。然后看网上资料说与jquery.min.js也有关，因为我使用了Bootstap，所以jquery.min.js是之前导入的有的，怀疑是这个js版本导致的错误，我又试了几个不同版本的jquery.min.js，然而并无卵用。  
&emsp;&emsp;最后，终于在一个不起眼的小旮旯里找到了一个说法：js引用版本冲突可能会导致这种错误，后来又在CSDN问答里看到了一个人也是遇到这种错误，说与include有关，最后仔细看include才找到了真正导致这个错误的原因。

## 解决方法

&emsp;&emsp;首先，我们需要明确几个前提（别人的博客里基本都讲过这些啦，但感觉还是有必要说一下）：  
　　1. 确认你使用的ajaxfileupload.js不是不知道从哪里搞来的可用性待商榷的。因为这个js较简单很多人是自己写的，所以可能有很多不是普遍使用的版本。我在这里给出我使用的js。[点我下载](https://pan.baidu.com/s/1MrhisHvt1Us8HKsoHUKcYg)  
　　2. 确定你导入了jquery.min.js和ajaxfileupload.js这两个js，并且路径没有问题，并且jquery.js在ajaxfileupload.js之前导入。  
&emsp;&emsp;如果因为路径错误或者配置错误什么的导致js没有成功导入上述两个js那就是别的地方有错误。判断自己是否成功导入js很简单，用开发人员工具看一下对应的js是否能打开，或者刷新一下页面在network里面看一下js文件的http请求状态，200或者304表示导入成功（这两个状态的不同可以自己百度，但是都表示成功）。  
　　3. 以上两点确认之后，那么重点就来了，你要看一下你的页面中有几个导入的jquery.min.js和ajaxfileupload.js，如果确保本页面这两个js都导入了一次，那么，你应该是使用了include标签，如果你在include标签里面导入了jquery.min.js或者ajaxFileUpload.js的话，浏览器就会因为在同一个页面中导入了两个同名的js而不知道该用哪一个（尽管导入的是同一个js文件！），所以干脆就不用了。。。然后就会报上述错误了，**确保你所有的include以及iframe子页面里面没有导入其它jquery.min.js**，那么这时就应该已经解决这个错误了。


----------


&emsp;&emsp;OK，就这样了，具体关于ajaxFileUpload.js实现无刷新文件上传的过程及代码我会另写一篇博客来记录，这一篇博客仅仅是描述并且解决一个报错而已。  
&emsp;&emsp;好好学习，天天向上，加油！