title: Web端即时聊天项目实现（基于WebSocket）
author: IcedSoul
tags:
  - WebSocket
  - 即时聊天
categories:
  - 项目实战
date: 2019-01-16 09:59:00
---
## 项目背景
&emsp;其实这个项目算是我做过的花时间最长也投入心血最多的一个项目了，当时决定开始做这个的时候我几乎什么都不会，那时我个人的情况是：
- **JavaEE方面：**  
会jsp+servlet，也简单使用过Struts，Spring仅仅只是听说过。
- **前端方面：**  
html，css有一些基础，会使用Bootstrap前端工具开发集，js基本不了解。
- **数据库相关**  
那时还没有学习数据库这门课程，但是熟悉jdbc与数据库的连接，会基本的增删改查SQL语句，简单使用过Hibernate
- **网络编程相关：**  
通信相关的知识也仅限于计算机网络课程上学习的简单的Socket。

&emsp;在那种情况下，我决定来做这个即时聊天的项目，先定下使用SpringMVC+Hibernate作为后端框架，然后一步一步查资料寻找和学习通信和前端相关的知识和解决方案，最终花了几个月时间完成了这样的一个项目，基本达到了预期的功能。但是更重要的是在这个过程中，我学会了很多东西，比如遇到问题应该怎么解决，常用开发借助哪些工具之类的，学会了使用SpringMVC框架来快速进行开发，学会了js原生的语法等等。尽管在没人指导自己瞎摸的情况下完成这个项目真的花了很多的时间和精力，但是最后的成果和学到的东西让我感觉这些都是值得的。  
&emsp;废话这么多的原因一个方面是想让以后的自己记得当时完成这个项目的心情，另一方面也是想告诉可能看到这篇博客的人，或许你也想做一个Web端即时聊天的项目，在漫无边际的搜索中从某一个旮沓角落里发现了这篇默默无闻的博客，你要相信尽管现在可能你会的很少，但是只要花时间踏实地去学习去研究，最终一定能够做出来自己想要做的项目，也能从这个过程中学习到很多东西，站在这个项目的基础上以后快速的开发出更多的更优秀的项目，学习更多的技术~  
&emsp;前面我会把演示方式和代码放出来，之后我会把自己在学习和查阅资料的记录一一整理一下写出来，可能没有什么条理，很多地方可能会有错误和幼稚的想法，请见谅也欢迎指正。
## 项目演示
**演示地址：** http://119.23.212.211:8080/WithMe  
**演示说明：** 请在浏览器中两个标签页中打开这个网址，自行注册两个账号，然后搜索添加彼此为好友，即可进行聊天。（同一个浏览器不同标签页可以登录不同的账号不会发生冲突）  
**Bug反馈：** 左边QQ群加群反馈或者发送邮件至：guoxiaofeng_2015@163.com。
## 源码
https://github.com/IcedSoul/WithMe  
（欢迎follow，star提issue~）
## 项目功能
&emsp; 项目最开始，需要明确自己想要做是什么东西，想要完成什么功能。  
&emsp; 这里啰嗦几句，其实这个项目最开始的设想的功能列表并不是这样的，这些是最后实际实现的功能，但是因为和最初设想地也差不多，这里就直接放出来了：

- Web端：
    - 注册、登录功能
    - 查看所有好友、群组
    - 查找好友、添加好友（可以附带验证消息）
    - 一对一聊天
    - 创建群组、查看群组成员、邀请好友进群
    - 群聊
- Android端：
    - 注册、登陆功能
    - 查看所有好友、群组
    - 一对一聊天
    - 群聊  

**注意：** 这篇博客中完成的项目不支持高并发！！！甚至可以说是并发性很差，非常差，我测试过我的小菜鸡服务器200个连接的时候就会出现无法建立连接的情况，300个以上时绝大多数连接都会无法建立。这里的项目只是完成了基本的聊天功能，如果想要考虑高并发的话请去查阅更多资料。我最近也在重构项目，想要尽量提高并发性，比如使用Netty作为聊天服务器，使用Nginx来进行负载均衡，使用Redis辅助缓存，来提高数据库访问速度，使用protobuf代替json来节约流量，优化转发，查询的逻辑等等~ 有这方面需求的可以深入研究一下互相交流交流~我也正在研究中。。

## 模块划分  
&emsp; 根据自己对JavaEE项目的认知，先把项目分为几个模块，弄清楚各个模块的依赖关系，然后弄清楚从哪里入手开始做，一点一点来完成。  
&emsp; 本项目主要分为以下几个模块：

- 数据库：
    - 数据库设计与创建
- 服务器：
    - 数据库访问（DAO）
    - 业务逻辑处理（Service）
    - 请求控制（Controller）
    - 通信服务器端（WebSocket Server）
- Web端：
    - 前端UI
    - 逻辑处理(js)
    - 通信客户端（WebSocket Client）
- Android端：
    - 界面UI
    - 逻辑
    - 通信客户端（WebSocket Client）

&emsp; 模块大致分这些应该没什么大问题，因为我是自己做的，也不知道真正标准的流程到底是什么，但是我的话是从数据库开始设计的，当然最初版本的数据库是不完善的，后面根据实际需要进行了很多次的修改。所以设计数据库的时候要尽量考虑周全，考虑完善一些，可以减少返工的次数和时间。不过很多数据库技巧需要有项目经验才能积累出来的，如果刚开始学习的话尽量参考别人的数据库设计并且多思考思考，回头修改是难以避免的事情，也不用最开始就太追求完美。
&emsp; 至于这里为什么就决定使用WebScoket了，WebSocket是什么，这里可以先不用在意，我最开始也是不知道WebSocket的存在的，做项目的过程才慢慢知道了解的，看后面我杂乱的记录就知道了。

## 项目所用框架和工具
&emsp; 下面所说的工具也是我慢慢摸索最后所采用的，如果不了解的话真的建议去了解一下，可以少踩很多我当时踩过的坑。  
- 开发工具
    - 开始使用MyEclipse，后来转Intellij IDEA
    - 前端写html，css基本样式使用Sublime Text 3（修改反馈速度较快），写js并且与后端调试是使用Intellij IDEA  
    /* 个人建议：直接使用Intellij IDEA，尽管没用过可能刚开始觉得不顺手，可是熟悉以后你就会发现IDEA的强大之处。前端的话如果自己没有前端工程的基础（Node.js，Vue，React等）,以后也不想向前端发展的话，不是很建议专门去学习这些，简单的html和js使用Bootstrap足矣完成本项目。JSP是否使用可以看个人，直接使用HTML跨域也不是不行。
    */
- 依赖管理和版本管理
    - 依赖管理
        - Maven  
        /* 个人建议，千万不要因为没学过觉得麻烦就不用Maven了，否则你浪费在不必要的寻找各种jar包下载和解决各种奇奇怪怪的版本错误上浪费的时间要远远多于你学习Maven的时间，而且解决那些问题浪费的时间是真正的浪费，即便最终解决了也不会给自己的能力带来有效的提升。 */
- 数据库
    - MySQL   
    /*
        数据库个人感觉其实都可以，可以为了学习换用MongoDB也行，只是建表（数据集合）就要自己改了。同时也可以考虑添加Redis作为辅助数据库，最近我在重构这个项目，就在考虑这方面的事情。
    */
- 后端框架
    - Spring MVC + Hibernate  
    /* 有基础建议使用Spring Boot，开发更为快速简洁一些，我那个时候为了学习SpringMVC而且也没听说过Spring Boot才使用的这个，WithMe重构版本就换用了Spring Boot，但是现在还在开发中，并不会放出来 */
- 通信协议
    - WebScoket  
    /* 这个可以去http://www.52im.net看一些相关的资料，WebSocket和SSE都是比较不错的选择，WebSocket与Socket的不同里面也有介绍 */
    - 第三方jar包
        - Java-WebScoekt
        - Github主页：https://github.com/TooTallNate/Java-WebSocket  
    /* 这个工具当时可以说是解决了我的燃眉之急，WithMe1.0中没有使用这个jar包，使用的是JavaEE 7.0自带的WebSocket，结果Android端建立连接一直出错（可能是我不太会用），2.0中使用的是这个jar包，如我之前所说，如果你考虑地更多想做地更好的话，你有更好的选择，不妨去了解一下socket.io，Netty，Mina等等 */

- 前后端数据交互格式
    - json  
    /* 这个真的不用说，Web开发必备知识之一，必须学习 */
- 前端框架和工具
    - Bootstrap 这是一个非常流行的前端工具开发集，可以借助这套工具进行快速得前端开发，官网主页：http://www.bootcss.com/
    - Layer.js 一个基于Jquery的弹出层解决方案，可以说我这个项目的前端基本就是依靠这个插件建立起来的，官网主页：http://layer.layui.com/  
    （注意区分LayerUI和Layer.js，前者是类似于Bootstrap的前端开发工具集，后者只是一个插件，而且后者是开源的，本项目中只是用了后者。）
    - 原生javascript语法  
    - Android端比较简陋，到时候可以自行参考代码。


> **下面开始是我在完成项目中所作的一些记录，可能会有些杂乱，我基本是想到哪里记到那里，希望能对做这个项目的人起到一些参考作用。一年前的自己，可能有很多不成熟的想法和错误的认知，重要的错误认知我会简单添加说明，但是依然可能存在错误：因为我现在依然在学习的路上，也不能说我现在知道的就是对的。如果看到错误大家能够指正，我也非常愿意修改。**


## 数据库
所用到基础知识：SQL建表语句等  
### 关于具体数据库实现：
user_main，user_detail两个表表示用户。  

而对于具体的好友关系的实现，经过查找资料和思考，目前有几种可以考虑的实现方式：
1. 建立一个friend表，当两个用户建立联系时就向表中插入一条数据，每次用户登录就查询这个表，建立好友列表。  
**优点：** 建表，好友关系插入、删除都比较简单  
**缺点：** 当用户比较多时，每次登陆会对用户数据库造成比较大的压力，当用户海量时每次登陆会消耗大量时间才能列出好友列表。
2. 在user_detail表中加一项friend，存放好友的list，每次添加好友时，取出改list修改之后存入数据库，每次登陆时取出该list。  
**优点：** 列出好友列表快捷。  
**缺点：** 对于数据库如何设置list类型虽然有解决方法，但是比较繁琐（如序列化，Json转String等等），所以解决方案为不建立list，仅仅将好友的id（或者user_name存下来），根据资料。MySQL的varchar长度最长为65535个字节，5000以内好友的存储应该没有太大问题，足以满足需求。这样可以在取出数据库信息后经过一个方法将其转换为list，实现“伪list存储”。但是这种方案产生的结果就是：好友关系不太明确，添加审核无法实现，查询好友关系也不容易实现。
3. 另一种替代方案是动态创建一张好友表，每注册一个用户就为其新建一个好友表，用来存储好友关系。  
**优点：** 同2，列表简单，查询也简单。  
**缺点：** 每添加一个用户就多出一张好友表，开销较大。  
4.	因此经过考虑，我提出了第四种方案，仍旧采用将好友id存储为String放在user_detail表的方法，但是同时也建立一张好友关系表。user_detail表中的好友字段作为缓存好友表，设置一个标志位，当用户没有添加好友的操作标志位为0，直接读取user_detail表中好友id创建好友列表，当用户有过添加好友的操作时，标志位改为1，此时从好友关系表中查询此人的好友关系，并且对user_detail表中的好友id进行更新，同时将标志位置0.  
**优点：** 建立好友表不是很麻烦。（因为添加好友操作不是频繁性操作），查询好友关系简单（直接查询好友关系表）。可以实现，添加好友审核好友的功能。  
**缺点：** 开销虽然没有动态创建表大，但是依旧较大。

**2017/1/22**  

今天对建表数据库进行重构，做了以下调查工作：
http://yaocoder.blog.51cto.com/2668309/1567715/  

**遇到的问题为：**  
用户的唯一标识时拿一个逻辑主键呢，还是用一个业务主键。在项目中，具体表现为：是以用户名为唯一标识，id也为唯一标识，通过用户名确定用户（建立索引），另一个是直接以用户名为唯一标识同时也作为主键。  
而且在具体问题中，当采用id标识并且采取自增也会出现一些问题。而我之前的建表语句则会出现更大的问题：因为两个表都采用自增，不能互相修改，也就是说一旦有一个出错那么在其之后注册的账户统统会错误，user_main和user_detail会全部对应错误。同时，在之后功能的实现中，我发现了一个问题，因为User类（即对应user_main）里面含有密码信息，而在实际登录、添加好友、聊天的过程中，需要经常把User类传来传去，以至于当我查找好友的时候当获取他的user_id和user_name的时候，带着user_password也一起传过来了，虽然实际上可能没什么感觉，不输出或者使用user_password这个值就好了，但是很显然在理论上存在重大的安全问题，这是无法接受的。  
基于以上两个问题，因此有必要对这两个表进行重构。首先，现在有两种选择：  
一、使用uuid代替自增，用String类型的id作为主键，同时设置user_name为索引。  
二、仍旧使用int id自增作为主键，同时设置user_name为索引。但是对之前的插入模式做一些修改，避免问题一的出现。  
前者性能不如后者，但是在分表分库的情况下方便（因为生成的uuid是时间空间上唯一的），而后者虽然之前当前表唯一，但是性能优于前者。  
综合起来，选用第二种办法，同时对user_main表作出修改。

在重构过程中我在user_detail里面添加了手机、邮箱等信息，避免以后使用到这些信息是再次重构数据库。  

而在添加好友功能的数据库设计上，我新建了一张好友关系表，并且在user_detail表里面添加了user_is_add_new_friend 和user_friends两项，用于实现上面所说的第四种好友关系管理的实现。  

为了实现通信功能，我新建了message表，这个表和客户端与服务端通信的json格式的数据是不完全一致的，在服务端与客户端通信的json数据中，from是发送者id，为整型，发送对象是int型数组，可能有多个发送对象。但是本数据库设计为from为int表示发送者id，to也为int表示为接收者id，之所以数据库这样设计不同于json数据格式原因如下：  
1.	无论发送对象是一个人还是多个人，WebSocket服务器（Server）进行的操作都是取出to，然后分别给每个发送对象发送消息。所以实质上服务器进行的操作还是一个一个发送数据。数据库操作为了记录服务器真正操作最好也为单条数据。
2.	便于实现查询离线消息功能。如果数据库设计和json数据格式一致，那么群聊消息将以一条消息形式保存在数据库，那么如何判断单个群聊用户是否接收到了这条消息呢？（eg.消息发送时用户不在线，当用户上线时需要查询数据库看自己有没有没有没接收的消息）单条记录使得无论是单人聊天还有群组聊天，表示是否接收的消息变得方便。事实上，表明单个群聊用户是否接收到了某一条群消息也只能够分条来。
3.	便于实现查询聊天记录功能，从上面看来查询聊天记录功能似乎不可为之，都是单条记录，如何区分单人聊天消息和群组聊天消息呢？事实上，有type可以作为区分，我们规定，单人聊天type为0或者-1，群组聊天type为1，其他类型的通信type也有分别的值与其对应。
也就是说，这个message表不仅仅是一个聊天信息表，它存储着所有的通信操作：比如说上下线通知、单人聊天信息、群组聊天信息、好友申请、群组邀请等等。它不仅仅记录消息，也记录着各种各样通信相关的操作。那么这时就有一个新的问题了，为何不分表。将这个message表分为：单人聊天记录表single_message表、群组聊天表froup_message表、上下线消息通知表online_status_message表、好友关系申请表relation_apply_message。这样的话可以将众多行为区分开来，分门别类进行存储，查询或者其它行为都会方便一些。但是这样子代码工作量比较大，不同表的项设计也是一个问题。依据项目进度，暂且搁下，日后再说。当前便使用一个message表。（以后修改时可以使用message）。
关于单人聊天和群组聊天是否有必要分表：首先，WebSocket服务器进行的是单独操作，群组消息也是一个一个发的，在这个意义上应该和单人聊天一样放在message表，但是这样会造成一些问题：一条群组消息就产生了很多个记录，群组成员每发送一条消息，群组有多少人就要像群组里面插入多少条数据，这样的开销显然是不合理的。在这个意义上有必要分表，一个群组消息就一条记录。那么这样当单人上线时，就没办法知道自己是否有未接受的群组消息了。不过这也有替代方法来实现，比如说在用户群组关系表里面添加一列表示是有未接收的消息。

为了实现创建群功能，添加了群组表group_main表  

为了实现加群功能，添加了群组用户关系表user_group_relation表  

群组聊天功能的具体实现仍然依赖message表。  

根据实际情况，添加了业务主键唯一约束。并将所有表的必要项添加了外键。
去掉了user_detail表里面缓冲好友列表的两项。  
(上面不是所有记录都加了时间，两句话之间可能隔了很久，很多前面想的东西后面因为实际情况就没做，其实就是怕麻烦hhh，看看就好。)

SQL建表语句：

```
create database withme;
use withme;
create table user_detail(
	user_detail_id int not null,
	user_detail_name varchar(20) not null,
	user_detail_nickname varchar(20) not null,
	user_detail_password varchar(20) not null,
	user_detail_role int not null,
	user_mail_number varchar(30),
	user_phone_number varchar(20),
	user_register_time datetime not null,
	primary key (user_detail_id),
	unique(user_detail_name)
);

create table user_main (
	user_id int not null,
	user_name varchar(20) not null,
	user_nickname varchar(20),
	user_is_online int not null,
	user_role int not null,
	primary key (user_id),
	unique(user_name),
	foreign key (user_id) references user_detail(user_detail_id)
);

create table user_relation (
	user_id_a int not null,
	user_id_b int not null,
	relation_status int not null,
	relation_start datetime,
	primary key(user_id_a,user_id_b),
	foreign key (user_id_a) references user_main(user_id),
	foreign key (user_id_b) references user_main(user_id)
);
create table message (
	id int not null,
	from_id int not null,
	to_id int not null,
	content varchar(5000) not null,
	type int not null,
	time datetime not null,
	is_transport int not null,
	primary key(id),
	foreign key (from_id) references user_main(user_id),
	foreign key (to_id) references user_main(user_id)
);
create table group_main(
	id int not null,
	group_id varchar(10) not null,
	group_name varchar(20) not null,
	group_creater_id int not null,
	group_create_time datetime not null,
	group_introduction varchar(50),
	group_user_count int not null,
	group_members varchar(5000),
	primary key(id),
	unique(group_id),
	foreign key (group_creater_id) references user_main(user_id)
);
create table user_group_relation(
	user_id int not null,
	group_id int not null,
	group_level int not null,
	group_user_nickname varchar(20) not null,
	enter_group_time datetime not null,
	primary key(user_id,group_id),
	foreign key (user_id) references user_main(user_id),
	foreign key (group_id) references group_main(id)
);
```


## 后端框架
额，后端框架当时真的是搞得太抓狂了，没留下来什么记录，我把惨痛的经历总结一下。
1. 用Maven管理依赖，IDE是MyEclipse也还，Intellij IDEA也好，习惯那个用哪个，不过推荐后者。
2. 配置找了网上的配置以后自己一一查查哪个配置的具体作用是什么。才能大概理解SpringMVC的整个工作流程。

其它也没什么，为了给大家节省时间，直接放我曾经写过的一篇SpringMVC基本Demo的博客，当然现在的SpringBoot项目的环境搭建更简单，想学习的可以去自行了解一下。我以后可能会写一写吧，现在没写过。。。    

后端的增删改查什么的，我也没记录，虽然当时也踩了很多坑，如果用的SpringMVC具体就请参考我的代码吧，大佬就请随意，用SpringBoot可试一下JPA。

[Spring+SpringMVC+Hibernate 基本Demo（注解、Maven管理）](http://blog.csdn.net/qq_33171970/article/details/68924277)


## 通信模块
通信模块实现过程：  
 1. 进行Java Socket TCP和UDP，客户端与服务端通信的学习
 2. 编写简单的客户端和服务端代码
 3. 在服务端使用多线程，可以实现响应多个客户端的基础上，已经将代码成功添加到项目中
 4. 将代码整合到Service中，整合完成之后因为SpringMVC管理的特性出现错误，因此改回正确之后准备在Controller中直接使用Server和Client类
 5. 测试正常，但是有以下问题：
    1. 服务端无法随Tomcat服务器启动而开启
  	2. 服务端无法区分不同的客户端（）
 	3. 测试仅为固定消息，如果需要从网页获取用户输入信息需要对代码进行简单修改（简单）
 6. 准备依次解决以上问题，网上查找博客，多次试验解决问题5.i，第一个问题有两种解决方法
    1. 自定义一个Listener，在web.xml配置之后，写一个类实现ServletContextListener，在类中开启ServerSocket
  	2. 自定义一个Servlet，在web.xml配置之后，继承HttpServlet，在init()方法中开启ServerSocket
  7. 经过测试，6.i实现失败，转而使用6.ii，成功配置Servlet，使ServerSocket可以随Tomcat启动而开启
  8. 6.ii虽然成功，但是产生了新的问题，因为ServerSocket accept()方法进行监听之后会阻塞线程，导致整个Tomcat服务器挂起
  9. 经过查找之后解决了问题8，一个解决方法为在HttpServlet的init方法中不直接开启ServerSocket监听，而是使用继承Thread（实现Runable不行）将这个线程设置为保护线程Thread,setDaemon(true)，这样线程里面阻塞不会影响Tomcat启动，而后台能够一直保持监听
  10. 问题8解决之后又产生了新的问题，无法关闭ServerSocket监听，端口一直被占用，Tomcat服务器关闭ServerSocket仍在运行，导致一次启动之后除非重启电脑否则端口一直被占用，无法再次正常启动Tomcat，使项目无法正常进行
  11. 寻找问题10解决方法失败，网上难以找到解决方法，自己思考也没有找到解决办法.  
  /* 来自现在的注释：其实当初这个问题的解决方法很简单，在Servle，destroy里面关闭监听线程就可以。。。当时知道的太少，这个根本不算什么关键问题，有兴趣的同学去查一下BIO，伪异步通信IO，NIO，AIO可以了解得更多 */
  12. 简单查看了一下问题5.ii的解决办法，据说可以通过端口、获取的ip地址等方法区分，但是因为此方法已经无法继续，所以没有具体实现，不知道可行性。
  13. 重新查找资料，决定采用HTML5新的协议WebSocket来实现通信功能，查看了WebSocket和Socket的区别
  14. WebSocket与SpringMVC结合也有两种不同的实现方法：
        1. 使用websocket-api，建立WebSocketHanlder和HandShakeInterceptor以及WebSocketConfig类进行配置，并且使用sockjs在前端页面建立客户端
  		2. 使用JavaEE 7.0的ServerEndpoint实现WebSocket
  15. 初步决定采用14.i的方法，在查找了大量资料，对第一种方法的实现框架有了基本的认识之后将其在项目中进行了实现
  16. 最终代码工作基本完成，但是运行时一直报一个bean的错误，多方查找发现缺少一个依赖包，网上找资料发现只有maven依赖的代码，根本找不到该jar包的下载
  17. 为此重新建立了一个简单的maven测试项目，将通信模块的代码复制过去之后能够成功运行了，但是结果竟然浏览器不支持。网上资料说明Chrome是支持的，因此判定为代码问题
  18. 再次查找资料，修改代码之后，不会报浏览器不支持，但是新的错误出现了，一直处于未连接服务器状态。
  19. 再次查找资料，无果。
  20. 借此机会，对项目结构进行完全重构，使用maven进行依赖管理，具体变化见pom.xml开头注释
  21. 决定采用14.ii方法实现通信。
  22. 查找资料完成代码后，14.ii方法也出现了与14.i方法相同的错误，连接服务器错误，预估为配置错误
  23.  仍不排除配置错误可能性，查找许久，有人说是新建WebSocket时路径错误，目前已初步排除此错误可能性。（路径确认无误）  
  已经根据浏览器开发者工具确定错误位于新建WebSocket位置即 websocket = new WebSocket(ws)处
  24. 再次经过查看众多网友的Websocket遇到404时的解决办法，将错误归为以下几类：
  		1. JavaEE版本低于7.0（其实这个很早就会报错了）
  		2. 新建websocket时路径错误，比如说没有加项目名称，或者是与@ServerEndpoint之后的名称不对应
  		3. web.xml或者spring配置了拦截器，拒绝了访问请求
  		4. Tomcat版本问题（7和8的不同，Tomcat7内部有Tomcat-WebSocket包，Maven部署时会产生jar包冲突）需要引入本身的依赖包，Tomcat8可以正常运行  
  	 这里的错误就是第四种错误，在网上没有直接的说Tomcat版本会导致404的说法，因为很多博客使用Tomcat7都正常运行了。
  	 但是留意到一篇博客下面有人说用Tomcat8正常了，但是用Tomcat7的人产生了与我一样的错误，所以由此推断出Tomcat版本  
  	 有影响。具体原因应该就是Tomcat7或者8对websocket-api支持不同，但是详细的机理还不是很清楚，留待以后研究  
/* 现在我也没研究，hh，不过做的时候留意查一下这方面的资料可以避免这些奇怪的问题。 */
  25. 简单例子测试成功（阶段性标志），接下来开始编写代码  
  		http://www.cnblogs.com/xdp-gacl/p/5193279.html?utm_source=tuicool&utm_medium=referral
  26. 遇到问题：如何在websocket方法中获取httpsession，从而拿到当前用户信息。方法如下：  
  		http://www.cnblogs.com/zhaoww/p/5119706.html?utm_source=tuicool&utm_medium=referral
  27. 确定了使用@ClientEndpoint后台实现客户端的可行性（对Android端重要），资料如下：  
  		http://www.cnblogs.com/akanairen/p/5616351.html
  28. 下一步任务如下：
  		1. 将wesocket-client.js设置为静态资源并且正常引用
  		2. 修改wesocket-client.js使用户登陆之后可以向服务端发送消息
  		3. 修改wesocket-client.js和Server.java使用户登陆之后可以接收服务端消息
  		4. 进行数据json格式设计，修改Server.java使其完成转发功能
 		5. 前端页面设计，完成通信功能
 /* 在这个期间中我完成了前端的内容，前后端结合着一起做的，可以看后面的前端的学习记录参照一下 */
  29. 继续任务（date：2017/1/22）静态资源配置成功，事实证明，放在resourse文件夹下面配置不可达（可能是因为路径问题），放在webapp下面静态资源就可以正常访问。28.i完成，进行28.ii。
  30. 当js放在外部引入时部分函数会失效，现在不必放入外部，留待以后整理。现在开始28.ii，js暂且放在内部
  /* 其实是因为我当时不了解js，熟悉js以后会知道不管是内部外部都是一样的，加载先后顺序而已，只需要把握好js的加载顺序就可以随心所欲了 */  
  31. 已完成登陆之后向客户端发送数据功能28.ii完成，继续28.iii
  32. 已完成客户端从服务端接收消息的功能，接下来进行json数据数据设计
  33. json数据设计完成，客户端转发功能完成，可以实现一对一聊天（又一个阶段性标志），但是存在以下问题：
  		1. 仅仅在双方都在线时可以互相聊天，当发消息给不在线的人的时候websocket会异常关闭
  		2. 群聊未实现（但是预留了实现途径，不麻烦）
  		3. 前端以及好友列表未实现
  		（想到在线还有列表有一种实现方式：每当自己上线时就给自己的好友发送一条“消息”，更新自己在好友处的在线状态。具体实现方法有待以后查找资料）
  34. 下一步目标
  		1. 实现好友列表（即时在线不在线），同时实现前端
  		2. 不在线好友即便发送信息也不会关闭，而是在该好友上线之后发送至好友处
  		3。 新建聊天记录表并将聊天记录存储至数据库
  	在实现这些功能之前，需要先做一件事：
  重新构建数据库，重构User表，重新选用主键，添加状态，修改之前先做备份。
  35. 修改完成

其中提到的message通信格式如下：

```
数据格式：
{
	"from": "xxxx",
	"to":["xxx"],
	"content":"xxxxx",
	"type":0,
	"time":"xxxx.xx.xx xx.xx.xx"
}
from： 	整型  	这条消息发送者的userId
to：     	整型数组	这条消息接收者的userId（可能为多个）
content:	字符串	根据type不同有所变化，见type说明
type:	整型	值为0时：这条消息为普通的文本消息，content为消息内容，服务器转发时会给发出者发一份
		值为-1时：这条消息为普通文本类通知消息，content为消息内容，服务器转发时不会发给发出者
		值为1时：这条消息为群组消息，content为消息内容，服务器转发时不会发给发出者，因为发送对象含有发出者自身
			注意：当值为1时，to数组不全为发送对象的id，0为groupId
		值为2时：请注意，值为2的情况并不会出现在前端发送消息的类型中，这严格来说是值为1的一种特殊情况。这条消息为群组消息，只被记录于数据库，to为群组的Id，服务端真正进行的操作是向群组的每一个用户发送一条相同的类型为1消息，而这个类型的消息仅仅用于记录用户和群组之间有这样的消息，以便于查询用户在群组里的聊天记录。
		值为3时：这条消息为上线通知，content无意义，仅为记录性内容，服务器转发不给发出者
		值为4时：这条消息为下线通知，content无意义，仅为记录性内容，不给发出者发
		值为5时：这条消息为好友申请，content为附加消息(申请结果将以普通消息格式返回)，不给发出者发
		值为6时：这条消息为加群邀请，content为群的id
time:	字符串	这条消息的发送时间
```

## 前端
以下为前端模块，前期也对后台代码做了一些小修改：
2017/1/23
1. 现在进行后台代码完善，有一些小问题，不过预计很快能够解决。具体内容为：  
a)	修改重构数据库之后部分代码直接使用字符串不使用User类。  
b)	修改主要涉及main页面，列出好友、查找添加好友等功能。
在此过程想到了一些问题：现在实现的通信过程仅仅是打开与某人聊天的页面时才与服务器连接，不算真正的“上线”，用户登录的操作并没有告知服务器，之后应该修改为登录时就建立与服务器的连接。但是这样又会引出一些问题。这个可以在之后设计前端的时候完善。现在先完成手头任务。
11:42  
现在问题具体描述为：  
点击添加好友按钮时会报Request processing failed; nested exception is java.lang.IllegalStateException错误。  
初步怀疑错误为：使用userId   int型放在隐藏input，后台接收数据时接收数据类型错误导致异常，两次测试结果如下：  
	同样使用User类的userId属性向后台传值的聊天功能正常。  
	在bulidrelation第一句打印输出的userId未能成功输出。  
因此排除因为数据类型而传值错误的可能性。  
经过自己检查，发现导致错误的原因：修改了后台接收值id   userName为userId，但是前台虽然修改了值，但是没有修改input的name属性，name仍为userName，与后台userId名称不匹配因此导致传值失败。  
此处只想说一声我次奥。
修改正确之后经过测试添加好友功能已经正常。同时测试时发现，删除好友处同样忘记修改，已修改，功能均已正常。  

    12:07  
2. 现在开始第二个小问题完善，具体问题为：
Java的date类型获取时间转化为sql的date类型时，只保留了年份月份日期，小时分钟秒钟的值没有保留，导致数据库后面数据均为0.  

    此问题在项目中主要有两处，一处为注册时，一处为添加好友时，解决方法为查找资料、测试，预计很快解决。  
    12:13  
    原因：java.sql.Date不存储具体时间信息，具体情况可以查看以下资料：  
	http://www.cnblogs.com/zhwl/p/3750118.html  
	http://www.cnblogs.com/1130136248wlxk/articles/5238538.html  
    解决方法：使用java.sql.Timestamp代替java.sql.Date,开始替换。  
    替换成功，开始测试。  
    测试成功。  
    另外一处开始替换。  
    替换成功，开始测试。  
    测试成功。  
    12:49
3. 代码完善暂且告一段落。开始准备前端实现，现在搜集资料，确定前端框架。

    12:51
4.	看了很久，不禁感叹一句，前端深似海啊，一直有新花样。看到了一个看起来用起来都非常厉害的IM前端，LayIM，可惜要钱，还很贵。虽然可以扣码。。。可是人家也不容易，还是算了吧，看看就行，自己慢慢搞吧，不过还好LayUI的Layer组件还是可以用的，很不错。AmazeUI看了许久，最后还是决定使用Bootstrap，毕竟老熟人了，用起来速度一些不用再专门花时间去学习。同时使用Layer插件，最后一定要做的漂漂亮亮的！开始吧，先大概看一下Layer插件的使用方法，之后依次制作登陆页、主界面等等吧~确定前端框架Bootstrap+Layer.  
16:46
5.	Logo图标制作完毕。（泪目，还要自己做Logo，心里苦），主要使用到了一个在线Logo制作的网站，效果真的挺不错的，最后自己也挺满意，虽然网站只提供了一个简单的原始Logo，但是我有PhotoShop啊，所以白色图片变成了黑色，黑色又变成了蓝色，大大小小Logo全有了。网站Mark一下。  
http://www.logoko.com.cn/design  
19:07
6.	开始制作编写页面
7.	在编写介绍页面时，将logo图片水平垂直居中时遇到一些问题：  
具体描述为：  
		使用css将图片水平垂直居中，在html中使用了两个div包裹img，在一级div使用css3实现垂直居中，div高度为auto自适应img的高度，宽度为100%（只有这样img的center-block在父级元素居中才会生效），我的本意是想让二级div宽度高度都用auto，然后再使用center-block将其居中。理论上是可行的，因为auto自适应内容宽高度，二级div应该等于图片大小，这时center-block(其实就是margin:0 auto)就可以生效了。但是实际却不是这样子，二级div的宽度仍然等于一级div的宽度，使用Chrome控制带查看发现图片的margin布局右边有宽度，但是在style表里面却完全找不到设置margin的属性，真是无奈。那么这时解决问题的方法就只能是不用二级div，直接在img的class里添加center-block和img-responsive（bootstrap使图片自适应的样式）。这样写很不爽啊。  
		在写上面内容的过程中想到了一个可能性：
		是否可以吧img- responsive给二级div或者一级div，然后把图片的宽高度都设置为100%，（因为刚刚尝试过直接把img-responsive直接给一级div图片不会自适应，还以为是img-responsive的问题，现在想来很可能是因为img没设置，这样即便img溢出一级div也发现不了啊，可以尝试一下。）
		啊，失败了，忘记考虑width为100%，初始logo就会变得巨大无比了，看来这样子是不行的，那只有按照上面那样写了，问题也不是很大。我觉得产生这种与期望不符的原因可能有两种：  
        1）	bootstrap全局定义时对img或者div做了一些不可告人的事（添加了我看不懂得属性。）  
        2）	img-responsive设置了margin属性。  
        3）	个人对于CSS width auto的属性理解可能有误。  
        根据现象来看，第二种可能性更大吧，毕竟在img右边出现了margin偏移的值的，我在去找找看验证一下想法。  
        测试完了，不是Bootstrap的锅，我的锅，真的是对auto这个属性的理解出了一些问题。哎，算了，就这样吧，继续实现动态效果。
        23：12
8.	就实现了一个简单的从没有到出现的动态效果之后，怎么看怎么不爽，这也太单调了吧。盯着看了一会儿，有了以下想法：  
将logo三部分分开，一个一个进来，最后上面的信息图标再来抖一抖，想了想觉得这个想法可以，找了一下正好有抖动的css源码，正好可以用上，我只需要写进入的动画就行了。那么问题来了，怎么实现。  
我开始的想法是，把这个图片分成三个部分呗，然后就纠结了，分成三部分那定位怎么办，之前好不容易才给弄到水平居中垂直居中，这三个碎图片定位岂不是要烦死？想想觉得害怕，不管了先切成三部分再说。  
又是百度又是切片又是裁剪小心翼翼弄了一个小时，终于弄好了三部分图片，还挺开心。  
然后开始思考，定位该怎么实现呢？最后结论是，几乎无法实现。然后忽然联想到了很久之前做的慕课网主页的项目，他们用的是什么？图层叠加，我去。。瞬间想到了在网页上应用极广的图片叠加。。。。。我只想说我。。。。。
然后又去橡皮擦擦了三张图片，现在开始实现啦！  

    2017/1/24 0:29
9.	在使用三个图片以后，搞了半天，终于实现了三个动态进入的效果，但是图标第一部分抖动的效果却没办法实现，因为不能给一个元素添加两个动画效果，设置双层也没有用，因为动画效果的属性会被覆盖，我之前项目中实现的连续动画实现的方法是弄一个长动画，前期动完中间静止，等待其他元素动画效果结束之后再继续动，三十这里就没办法这么实现了，没办法，只能放弃shake.css了。

    2017/1/25 12:23  
10.	实现动画效果之后东看西看了一会儿还想捣鼓点什么，最后看看进度还是算了，还有好多东西没做。想办法制作了几个ico图片，让浏览器顶部title前面有了小图标，效果还可以吧。其实只是一个语句的事情，关键是制作ico图标不容易，我用的PhotoShopCS6保存的时候没有ico的格式，网上说要下载一个插件，找了许久插件下载了三四种结果还是不能保存成ico格式。一怒之下直接百度找在线ico图片制作，然后就找到了。。。只能说好气啊，我Tm下什么插件。。。捣鼓这么久，最后使用在线工具制作了各种型号的logo，网址也mark一下。  
http://www.bitbug.net/
11.	ico图标制作完成之后接下来添加了两个按钮，分别是登录和注册，按钮效果直接用到暑假时花了很大功夫设计的两个按钮，如果现在重新写肯定又要花几个小时的时间。果然前端应该是越做越容易，很多效果之前都写过保存的有，现在拿出来改改样式就能满足现在的要求了，但是即便这样改成我现在需要的样式也是花了不少的时间。按钮的位置按照最初的设想应该是在右上角，可是放上去发现实在是难看，最后对布局做出了一些调整之后就放在了中间图标的下面，然后分别在左上右上加了pc下载和app下载的样式，因为时间原因没有怎么添加效果。不过这样已经差不多了，第一个页面完成。
12.	然后是登录页和注册页的制作，研究了一段时间，最后决定采用之前使用过的登录注册样式。仍是代码弄过来之后做出微调，同时根据基础Logo制作了一个横版logo，放在了登录注册页的上方。具体过程不再多说。  

    2017/1/25 17:54
13.	接下来就要开始研究主页面了哈哈，为此需要专门学习Layer弹出框的插件，因为在原来的设想中，web版本的主页是需要以弹出框为基本组件的。
在查看玩所有文档，进行许多在线调试以及记录笔记之后，已经对Layer.js有了基本的认识，接下来就要开始编写啦！加油！js我来了。  

	2017/1/25 20:40
14.	昨天研究完Layer之后就没有再继续，今天继续制作main，一定在今天之内把main页面完成！
15.	通过Layer尝试创建好友列表成功，准备开始制作聊天窗口。
16.	聊天窗口制作完毕，完善好友列表
17.	前端窗口基本已经制作完毕，开始与后端的整合。  

    2017/1/26 22:16
18.	前端视图与后端整合基本完毕，下一步开始细节完善。（这两天做前端的时候比较烦，在制作过程中当然也遇到了很多问题，不过为了节省时间就没有在这里说明，只大致说一下制作的内容。）
19.	 无  

     2017/1/27 00:04
20.	功能已经正常，开始解析接收数据，根据发送消息的类型决定窗口中消息样式。
21.	今天是除夕+_+，然而这和我有什么关系。今天完成的主要内容是完善了几个细节：
    1. 聊天窗口图标、时间和消息主题气泡（小三角形尚未实现），在实现这个的过程中排版遇到了很多问题，花了很长时间，最终使用了bootstrap的栅格布局较为妥善了实现了这个效果，也兼顾了响应式效果。但是此处遗留了几个问题，请注意。
    2. 用户头像现在是默认的，之后需要实现用户上传头像的功能，使用数据库存储
    3. 头像路径，然后在此处显示头像，好友列表处头像同理。
    4. 虽然左边消息和右边消息都实现了，但是怎么根据实时接收消息在指定区域插入这些效果，今天稍微查了一下，使用js应该可以实现，但是还需要注意的是需要实现另外几个效果，使用流加载方式加载过久的历史消息，有新消息出现应放置在最下面，并且需要把聊天窗口定位到最下面
    5. 小三角形还需要专门去实现，怎么贴紧气泡还是一个问题。
    6. 当聊天窗口最小化时，title的高度太高导致左下角最小化的区域太大，也需要进行优化。
    7. 接收消息解析留下了接收，之后问题的解决需要配合这个来实现。添加了进入主页面好友列表自己弹出来的效果
    8. 添加了当有很多个聊天窗口时，点击哪个窗口，哪个窗口置顶的效果  

    2017/1/28 01:03
22.	总结了今天做了什么之后，再次进行一下整体的总结。现在具有的缺陷以及需要完善的功能有
    1. 如20.a.ii所说，接收到消息的显示仍然存在问题。解决方法也同上。
    2. 只有在聊天双方都在线时才能够聊天，当向不在线的人发送消息时，websocket会异常关闭。需要根据数据库实现向不在线的人发送的消息会在该用户上线的时候接收。具体数据库设计可以专门再考虑。实现这个功能的前提有两点：
    3. 服务器在转发消息的时候将消息存入数据库。
    4. 用户上线会通知好友自己上线，实现好友列表在线离线的区分。
    5. 完善添加好友、删除好友功能，这个后台代码都已经实现，只是需要前台实现
    6. 实现用户上传头像的功能见21.a.i，实现需要修改数据库并学习springMVC上传文件。
    7. 实现群聊功能，实现添加群、删除群功能。
    8. 实现登录、注册的过滤、失败提示等功能
    9. 实现桌面版swing编程和AndroidUI编写
    10.最后根据需求添加其它功能  
    大概需要做的就是上面这些事情了，花了半个月了才做了这么点事儿，真的感觉很难过，接下来需要抓紧时间，尽快完成这个项目。  

    2017/1/28 01:19
23.	接收消息根据发送人的不同可以显示在左边右边，并且可以读取发送信息的内容，下一步实现新消息展示在最下面，并且会将之前的消息向上顶的功能。  
Tips：这两次运行项目打开时会出现好友列表不会自动弹出的问题，看了一下似乎是zIdex值的问题，在内部也因为实现了点击谁谁置顶的功能，所以弹出的alert和msg等都在下面了。晚一下需要专门解决一下这些问题。  
目前遇到的问题是，虽然可以根据发送人的不同把消息显示在左边或者右边了，但是新的消息会替换掉上一条消息，始终只有两条消息存在。  
使用调试工具查看之后发现这种现象的原因是，插入div时js是根据id插入的，当这个id已经存在，再次插入时就会替换掉之前同id的div，所以初步设想解决这个问题的办法是，当把消息放到output区域时，抹掉它的id。  
24.	事实证明不行，除非在js内部创建，否则就是被搬运而已，并不能创建新的div
25.	Js内部创建只能创建一层div，并不能解决问题。经过寻找发现使用jquery的apend（）方法也可以向div内部插入htmi代码。试一试。
26.	Jquery语句一直出错，要疯了。。。。
27.	终于找到错误了，把小括号写成大括号了，我说怎么一直错。聊天的排版已经正常了。还需要加一个接收到新消息就滚动到最下面的效果。此外又发现了一个问题：我这里接收到消息时显示在输出区，显示到了所有人的输出区，这里应该对输出区输出做一个限定，比如说指定一个与用户id相关的动态id，这样输出起来就不会乱掉了。试一试。
28.	没网，哎，好烦，今天就这样吧，明天实现好友区分，提示层，添加/删除好友功能，赶紧弄，进度太慢了。  

    2017/1/28 20:42
29.	今天试了好多次了，似乎是不能弹出相同的层，因为我是根据id来获取div内容的，而页面却不允许多个id相同的元素出现，所以不会自动弹出新的层，和上面遇到的插入聊天内容时的问题很相似：我们需要根据id来标志元素，然而我们没办法通过id来区分他们，但是不设置id的话找都没办法找到他们，设置动态id在html里面又不现实，只能想办法在js里面看能不能使用div了。  
貌似可以。  
测试成功。  
30.	同时和多人聊天信息已经归位，不会乱跑了。用的方法略有粗糙，那就是在创建聊天窗口div的时候把输入输出区的id前面都加上聊天对象的id，无论是发送还是接收的时候再根据信息在前面拼接上id，这样就可以做到聊天的时候互相不影响了。接下来完善添加好友功能。
31.	哈哈哈哈，发现了一个新的神器！Jquery的ajax，之前还以为ajax是一个新的比较大的框架之类的东西，功能很强大学习起来也要很长时间所以一直没有接触，刚刚查了一下发现ajax是功能很强大但是它可以用Jquery的ajax来实现！这意味着什么？这意味着我以后就不用傻傻的更新一次数据就动刀动枪刷新数据了！也意味着不用为了传个值费劲心思弄个隐藏的input来使用from提交了，在这个项目中用不着频繁的存取session了，主要是好友列表的更新。（在线状态、好友人数等等）添加好友功能等等，使用了ajax就可以在不刷新的情况下就更新数据啦！（或许你会有些奇怪既然之前没有使用ajax，那么聊天页聊天是怎么实时更新数据的呢？因为websock事先设置了接收消息的方法，其背后实现的原理尚且不清楚，可能和ajax功能类似，但是好友列表什么的websocket可就不管了）。现在开始愉快的学习使用ajax吧！
32.	Hhh，学习完毕，初步测试成功，可以开始应用啦（dog脸），先睡觉,明天起来弄！  

    2017/1/30 01:33
33.	添加好友功能基本结束，下一步，实现异步通讯（即当聊天对象不在线时），解决问题22.b
34.	啊啊啊啊，异步通讯代码已经完全写完了，应该是没有任何问题的，结果在websocket的server端使用service的地方一直报空指针异常，开始我还以为是查询的问题，改啊改，试啊试，搞了这么久还是空指针，直接注释掉没想到下面插入数据的地方也报同样的异常，我的天哪，我要疯了，上网查了半天完全没有人这样做过自然也没人这样错过，抓狂。据我估计代码逻辑是没什么问题的，应该是springMVC对service的使用做了限定，websocket的server端可能是没办法使用service的，继续看看概念！！！
35.	又找了很久，原来还是有人遇到过这种无法注入service的情况的，都比较抓狂，再找找有没有什么解决方法。
36.	卧槽卧槽卧槽卧槽卧槽，解决办法有了！但是没用！找了一会儿发现我忘记给新的messageService添加Service注解了!丫的竟然也不报错！！！无形之错，最为致命。
37.	Hhhhh，经过配置bean通过bean获取service的方式的终于正常了，为了这个问题搞了四五个小时，哎。还有莫名其妙logo没了，字体变成斜体，连动画效果都变了，虽然整体没什么问题，但是很不正常，明天再看看吧，今天弄太晚了，睡觉。  
    2017/1/31 01:44
38.	今天上午弄了半天，异步通讯已经完全实现。中间纠结了很长时间ajax单独函数返回值为空的问题，可能我还是对ajax返回值不太理解，不管了，在中间实现就行了。现在实现的效果是：当你给某人发消息时，如果他不在线，那么他上线时便会在页面正上方收到消息提醒，点击提醒的消息就会打开聊天界面。如果发送给在线的人，但是对面没有打开与自己的聊天窗口，消息同样会显示在顶部消息提示区。同时，聊天消息新消息到来滚动条自动到底部也实现了，自定义滚动条样式也实现了。现在还没实现的功能就是群聊和好友列表了。尽量在今天搞定！
关于昨天晚上发现的字体变成斜体，标题栏图标变成tomcat的原因找到了，在main.jsp的最前面和最后面莫名其妙多了一个i的标签，我从来没加过，可能是之前全选使用Myeclispe的format功能时他自己给添的，去掉之后就正常了。今天为排版乱掉查了很久都找不到原因，哎，没想到竟然是这种问题。  

    2017/1/31 14:26
39.	现在开始联系人列表（“好友”列表）的重制，这次重制的目的是为了实现区分上下线、实时更新列表的功能，比如说我与一个人建立了联系，那么不用刷新页面，联系人列表里就会多出一项来。一个联系人上线了，那么好友列表区域就会显示出这个人实时的在线状态.
40.	好友列表换成ajax已经实现，现在开始制作实时在线通知。关于这个功能的实现我现在的想法是：每个人上线的时候，给自己的所有联系人发一条消息，定义type=3，意味着上线通知，下线的时候给所有人发一条type=4的下线通知。然后每个人在收到该类型的消息的时候对好友列表进行实时更新。
41.	上线通知代码基本完成，开始测试。
42.	测试成功。功能基本实现，但是还需完善，需要完善的地方为：在用户刚上线时应该获取到自己好友的在线状态，应该写入数据库，另外需完善下线通知。OK，今天就这样吧。明天稍微弄一下就能实现这个功能了。睡觉。  

    2017/2/2 01:13
43.	好友列表实时更新已经实现，现在自己上下线会所有在线的好友发出上下线通知，如果好友不在线则不会收到此通知而且这条通知不会被记录到message表。收到好友上下线通知时如果好友列表在打开状态那么会直接更新好友的在线状态，如果不在打开状态则会和消息一样放到提示区，点击提示区则会打开好友列表，然后更新该好友的在线状态。但是现在有一个莫名其妙的bug，第一次列出好友列表是正常的，关掉好友列表然后再打开。。。就会发现好友列表只剩自己一个人了。。。。还不清楚为什么，至少除此之外都实现了。中间遇到的坑就不多说了，在代码注释里面有说。  

    2017/2/2 12:20
44.	经过测试，我知道产生上述现象的原因了，这并不是因为代码有错误，而是一些东西的用法导致的错误。具体解释如下：  
我们在登录时进行了一个操作，登陆成功就把当前的用户对象放进session里面，到后面列出好友列表的时候也是在后台取出这个名为currentUser的session，获取到id，然后去查询这个id所有的好友，再返回给前台，可是现在产生了一个问题就是，session里面的currentUser原则上来说就是当前登录的用户，可是如果当前有两个用户用户A和用户B使用这同一个浏览器登录，用户A先登录，用户B后登陆，那么B的currentUser的值就会代替A的，那么如果A用户仍在线，那么他刷新一下好友列表，于是A好友列表就会变成B的了，真是蜜汁尴尬。应该想个办法不依赖session来获取当前登录的用户对象。
45.	受知识限制，不依赖session的话从login到main传值实在是找不到别的办法，不过有一个办法可以解决上面说的那个问题。Server端可以在连接刚刚建立的时候就取出session里面的用户对象，然后通过一条消息把这个对象返回到前台，前台接收解析之后赋值给全局变量，之后都拿全局变量来搞事就好了么。这确实是个办法，不过就是需要专门再定义一个消息类型。
现在好像有个更方便的方法，好像如果页面不刷新的话当前session的值是不会变的，也就是说如果我不刷新main页，main里面的currentUser是不会变的，那么就不用上面说的那么麻烦了，直接把之前的从controller里面取session换成从前台传值过去就好了么，试一试。
46.	修改成功，开始测试。
测试成功。
开始下一个工作，把添加好友的功能完善了，加一个审核好友的功能。
47.	添加好友功能基本完成。但是出现了一些问题：两个以上的人同时添加一个人为好友时，后一个人的好友申请会被前一个人的申请给覆盖掉，为此，我专门弄了一个模拟队列来存放好友消息，可是出了一些问题，代码是没问题的，测试也都运行正常，可是后面的窗口就是不弹出来，试了好久还是没办法，算了，就这样了。换个办法吧，新想到的办法是一个窗口放多个好友申请，Y方向设置允许滚动，写代码去实现吧。
48.	唉，后期测试好麻烦啊，改一个效果为了看这个效果有没有用，改了一分钟，为了看这个效果花了五分钟，发现一处错误回头改改又是五分钟。。。不会用测试工具就这样，以后学着用单元测试工具吧。多好友申请同时出现的问题已经解决，现在已经完美实现了添加好友的功能。下一步工作，也是比较复杂的工作，实现群聊功能。先备份一下项目。    

    2017/2/2 23:45
49.	要实现群聊功能，首先需要设计好维持群关系的数据库，之前的代码中已经预留了一对多发送消息的接口，因此，消息发送并不是一个难点，聊天窗口也和单人聊天基本一样，也不是很麻烦。难点在于群关系的数据库应该怎样设计，好麻烦的样子，明天解决吧，明天一定要把群聊功能，查找群，加群，退群，功能给实现了。
50.	现在开始群数据库的设计，准备新建一个group_main表，含有id，group_id,group_name,group_creater_id,group_create_time,group_introduction,group_user_count,group_members，另外建一个群用户关系user_group_relation表，含有user_id,group_id,group_level,group_user_nickname,enter_group_time,大概就这样设计吧，性能问题先不管，这已经是我能想到的比较好的设计方案了。开始动手吧。  

    2017/2/3 13:09
51.	两个表新建完成，entity、dao、service对应的代码也都完成了，下一步实现前端并且实现与前端对应的controller。  

    2017/2/3 15:57
52.	哎，几天忘记写这个进度笔记了，不过内容继续做了，群聊功能代码已经实现、加群功能实现，获取聊天记录功能实现。现在具体功能基本完善，然后就是优化提示窗口，增加输入过滤功能，实现头像上传功能了。  

    2017/2/7 12:30
53.	不过上面说的那些东西我打算等等再弄，毕竟都是一些小细节，不至于大动干戈的，今天花了一天时间把项目成功配置到了云服务器上面，现在已经完全配置成功。  

    2017/2/7 23:33
54.	今天简单看了Android的实现，相对来说还是比较简单的，晚上实现了登录注册时验证的功能，同时修改了loginController和registerController返回Map类型的数据（现在还不确定SpringMVC有没有自动把map类型数据转为json，看样子应该是没转）。只有把必要的功能的Controller全部改成这种类型的，JavaSE端和Android端才能也使用SpringMVC的框架来进行数据获取，OK，今天就这样。睡觉。  
55.	已经过去很长时间了，寒假结束，已经开学一周了，不过项目中间一直也没停过，只不过中间没有继续记录下来。我大概把这么多的遇到的比较重要的问题描述一下。像上面说的那种鸡毛蒜皮的小错误就不再描述了。
56.	项目进展到之前停止记录时已经完全实现了Server端了。接下来我所做的工作就是Android端，在Android端使用WebSocket协议时我遇到了一个比较重大的问题：Android无法使用我在Server端使用的WebSocket协议，经过查找资料，最终我得到了两个相对来说比较可行的解决方案

## 总结
  &emsp;其实记录到这里已经没了，可能记录到前端部分开始对大家的帮助已经不大了，大部分是我自言自语性质的记录，没学过前端的也看不懂，学过的没必要看，不过我还是放了上来，毕竟是当时的真正心情的记录，或许你看了这些觉得对你毫无帮助，那么很抱歉浪费了你这么多时间来看这一篇没用的东西。如果对你有所帮组，那么我也很开心。  
  如果要自己从头开始做这个项目，建议有自己的主见和想法，可以做出与我不一样的选择，最终做出来的成果或许比我的效果要好得多。  
  &emsp;伸手党直接拿我的代码去交作业的同学请注意了，我不是对这种行为有什么偏见，我个别课也曾经这样干过，但是我想让你明白的是，我把项目放出来，是为了让之后做这个项目的同学可以少走一些我当初走过的没必要走的弯路，哪怕是一个或者几个也好，同时也是为了总结一下我的项目。我在做项目的过程中使用到了很多开源工具，参考了很多其它人的项目和博客，最终开源也是理所应当。虽然代码已经开源任君自取，但是也请不要更名换姓将其称之为自己的作品，即便是开源项目，这也是不被允许的。我也不想因此害了某些同学，特此说明。

  &emsp;那么上面所说的可行的方案是什么呢，当然就是我上面所提到的Java-WebSocket了，可以自己了解一下然后对照代码理解一下。

  &emsp;这篇博客杂乱无比，估计也没啥人有耐心看，不过就这样吧。

  &emsp;好好学习，天天向上。