title: Spring+SpringMVC+Hibernate 基本Demo（注解、Maven管理）
tags:
  - Spring
  - SpringMVC
  - Demo
categories: Spring MVC
date: 2019-01-15 19:58:50
---
## 介绍
&emsp;&emsp;本片博客主要是介绍一下Spring+SpringMVC+Hibernate框架的大致工作流程和其基本的配置代码，在博客最后提供了一个基本的Demo下载，本Demo采用Maven管理依赖。
## 环境、工具
&emsp;&emsp;Java开发环境、IntelliJ IDEA 或者 MyEclipse  
&emsp;&emsp;Tomcat服务器（7，8，9均可）
## 工作流程
&emsp;&emsp;Spring容器想必不用多说，大家手头端的参考书写的估计很清楚，百度一下也有很多大神写的也很好，我就不赘述了。在我们实际使用过程中，我们使用Spring来管理事务，Hibernate就全部托管给Spring，而Spring提供的MVC开发模块即SpringMVC当然也是由Spring管理的，其实也不一定要使用SpringMVC，Spring也可以管理Struts2等。Spring的控制反转和依赖注入这里也不再多说，大家可以自己去专门学习了解。这里主要简单说一下SpringMVC的工作流程，因为这与下面的配置有一定的关系。大家结合着工作原理来看配置文件，两相印证之下应该会对SpringMVC工作流程以及配置为何这么配置有一些新的认识。  
&emsp;&emsp;首先，在说SpringMVC之前，我们先来看一下一张经典的MVC开发模式的图。  
![](http://ou7jocypf.bkt.clouddn.com/17-8-5/40753646.jpg)  
&emsp;&emsp;所谓MVC开发模式，Model，View和Controller三者分离，三者分离的目的主要是降低耦合程度。从图中可以明显看出，当我们进行Web开发时，我们采用MVC开发模式，应该要有一个清楚的层次认知（“分层”是一个说法，没有太过具体的明确规定，不一定非要奉某种方法为真理，合适有理有据惯用好用即可），在我的理解以及项目中，主要如图中分为了dao层（数据库数据处理层），service层（业务逻辑层），controller层（控制层），前端即view，大家也可以去了解其它分层方法，喜欢就好。  
&emsp;&emsp;上面那张图就是本Demo项目结构是这样的原因：  
![](http://ou7jocypf.bkt.clouddn.com/17-8-5/81263064.jpg)  
&emsp;&emsp;了解了项目为何这么分层之后，来看一看SpringMVC的工作流程，这部分的介绍将会结合配置文件一起说，先来看下面一张图：  
![](http://ou7jocypf.bkt.clouddn.com/17-8-5/10349208.jpg)  
&emsp;&emsp;其实SpringMVC工作流程按照图中的步骤来看就非常清楚啦！如果你是第一次接触可能不太熟悉每个部分的单词，但不用担心时间久了慢慢就记得了，只需要大致有个概念就行了，因为这些都是框架帮助我们在后台完成的，我们只要按照既定模式来配置好就好了。  
&emsp;&emsp;首先，当收到来自用户的请求时，就像你要去某公司找某人有事，你就是那个请求，DispatcherServlet首先拦截了所有的请求,就像个门卫大爷似的，进门就把你拦住了，不讲清楚你是来干啥的就不放你过去啦。DispatcherServlet然后把请求交给HandleMapping，HandleMapping就做了一件事，我来看看你是找谁的，弄明白你是找谁的我才能派人通知他不是？因为SpringMVC的Controller里面是有很多处理方法的。就像你说你们找谁的，那门卫大叔就要把你交传达室小伙子，查查你要找的人在不在，在哪里。
在HandleMapping找到以后，接下来就到了HandleAdapter，HandleAdapter对Controller中具体的处理进行的封装，可以理解为现在找到你要找的那个人啦，现在去见他把，不过不能直接见，得在我们给你的房间见，这个房间就是HandleAdapter，在这个房间进行py交易之后，你搞不清白你交易的结果是啥，接下来就需要对内容进行解析，viewResolver来对你拿到的内容进行解析，就相当于你对交易的结果搞不懂啊，人家用的人家专用的格式，你得找人家专门的部门帮你翻译一下，最后翻译成功，找到view，一个完整的工作流程结束。  
&emsp;&emsp;例子不一定恰当，但是能在一定程度上来表明问题。  
&emsp;&emsp;接下来结合上面了解的工作流程来看看配置文件如何配置的吧。
## 新建Maven JavaWeb项目
&emsp;&emsp;IntelliJ IDEA创建这一点我不打算自己讲，因为其他博主讲的真的是太好了，我不认为自己复述一遍有什么意义，链接如下：  
&emsp;&emsp;  http://www.cnblogs.com/sy270321/p/4723139.html  
&emsp;&emsp;MyEclipse或者Eclipse请大家自行百度啦，这方面博客很多的！
## 主要配置代码
&emsp;&emsp;先看pom文件，导入的jar包，在这里我声明，和别人乱七八糟的依赖关系不同，在导入之后我花了很长的时间把所有不必要的依赖关系都去掉了，能简单的绝不复杂，这里我所导入的包都是我能找到最简单最方便快捷的导入方式了，导入的依赖看起来不多，但是几乎已经涵盖了所有用到的内容，每一个依赖是有什么作用哪里能用到都有说明，你可以根据自己实际使用情况作出修改。  
pom.xml
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>example</artifactId>
  <packaging>war</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>example</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>

    <!--JavaEE 7.0-->
    <dependency>
      <groupId>javax</groupId>
      <artifactId>javaee-api</artifactId>
      <version>7.0</version>
      <scope>provided</scope>
    </dependency>

    <!-- 这个必须要加，不然连jsp页面都打不开 jsp页面第一行引入了这个-->
    <dependency>
      <groupId>org.glassfish.web</groupId>
      <artifactId>javax.servlet.jsp.jstl</artifactId>
      <version>1.2.2</version>
    </dependency>

    <!--MySQL 5.1.38-->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.38</version>
    </dependency>

    <!--SpringMVC 4.3.5.RELEASE -->
    <!--配置第一个之后会自动导入一些spring mvc相关的依赖包，但是有可能我们需要包的没有被自动导入
        因此，之后的第二个及以后是根据项目需要添加需要导入的spring mvc相关的包-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>4.3.5.RELEASE</version>
    </dependency>

    <!--这个包不会随spring-webmvc导入，这个包作用是管理Hibernate资源-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-orm</artifactId>
      <version>4.3.5.RELEASE</version>
    </dependency>

    <!--这个也不会随spring-webmvc导入，作用是为spring-aop提供支持-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-aspects</artifactId>
      <version>4.3.5.RELEASE</version>
    </dependency>

    <!--Apache Commons dbcp 2.1.1  Apache提供的一些组件，dbcp配置数据源时需要使用-->
    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-dbcp2</artifactId>
      <version>2.1.1</version>
    </dependency>

    <!--支持SpringMVC文件上传功能-->
    <dependency>
      <groupId>commons-fileupload</groupId>
      <artifactId>commons-fileupload</artifactId>
      <version>1.3.2</version>
    </dependency>


    <!--Hibernate 4.3.8.Final-->
    <dependency>
      <groupId>org.hibernate</groupId>
      <artifactId>hibernate-core</artifactId>
      <version>4.3.8.Final</version>
    </dependency>

    <!-- FastJson 1.2.24 -->
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>fastjson</artifactId>
      <version>1.2.24</version>
    </dependency>

    <!-- jackson-core Spring MVC配置RequestMappingHandlerAdapter时用到了-->
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-core</artifactId>
      <version>2.8.6</version>
    </dependency>

    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.8.6</version>
    </dependency>



  </dependencies>
  <build>
    <finalName>com.example</finalName>
    <plugins>
      <!--这个插件作用是指定编译这个项目的Java版本和Project Language Level ，针对Intellij IDEA-->
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>2.3.2</version>
        <configuration>
          <source>1.8</source>
          <target>1.8</target>
        </configuration>
      </plugin>

    </plugins>
  </build>
</project>

```
&emsp;&emsp;看起来很少对吧，实际上你还可以根据自己情况再删去一些自己不需要的！这些包具体干什么的注释里都有介绍，就不再多说了。最后提供下载的文档里注释也有的。  
&emsp;&emsp;然后是web.xml
web.xml

```
<!DOCTYPE web-app PUBLIC
        "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
        "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
         version="3.0">

  <!--配置默认进入页面-->
  <welcome-file-list>
    <welcome-file>/WEB-INF/views/index.jsp</welcome-file>
  </welcome-file-list>


  <!-- 配置Spring监听 -->
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>


  <!-- 加载spring配置文件 -->
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:spring/applicationContext.xml</param-value>
  </context-param>

  <!-- 配置Spring MVC-->
  <servlet-mapping>
    <servlet-name>spring</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>

  <servlet>
    <servlet-name>spring</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:spring/spring-mvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
    <async-supported>true</async-supported>
  </servlet>

  <!-- 配置Hibernate Session -->
  <filter-mapping>
    <filter-name>openSession</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>

  <filter>
    <filter-name>openSession</filter-name>
    <filter-class>org.springframework.orm.hibernate4.support.OpenSessionInViewFilter</filter-class>
  </filter>

  <!-- 配置字符集 -->
  <filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
      <param-name>forceEncoding</param-name>
      <param-value>true</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
</web-app>
```
&emsp;&emsp;在这里我们看到了DispatcherServlet，他就是图中那个看门大爷！然后我们看一看applicationContext.xml  
applicationContext.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
                        http://www.springframework.org/schema/beans/spring-beans-3.1.xsd 
                        http://www.springframework.org/schema/tx 
                        http://www.springframework.org/schema/tx/spring-tx.xsd
                        http://www.springframework.org/schema/context
                        http://www.springframework.org/schema/context/spring-context.xsd
                        http://www.springframework.org/schema/aop
                        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- 扫描注解,过滤掉Controller -->
    <context:component-scan base-package="com.example">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    <!-- 引入数据库资源文件 -->
    <bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="location" value="classpath:properties/database.properties" />
    </bean>

    <!-- 配置数据源 -->
    <bean id="dataSource"
          class="org.apache.commons.dbcp2.BasicDataSource">
        <property name="driverClassName" value="${driverClassName}"></property>
        <property name="url" value="${url}"></property>
        <property name="username" value="${userName}"></property>
        <property name="password" value="${password}"></property>
    </bean>

    <!-- AOP -->
    <aop:aspectj-autoproxy/>

    <!-- 配置 hibernate session -->
    <bean id="sessionFactory"
          class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="hibernateProperties">
            <props>
                <prop key="hibernate.dialect">org.hibernate.dialect.MySQLDialect</prop>
                <prop key="hibernate.hbm2ddl.auto">update</prop>
                <prop key="hibernate.show_sql">true</prop>
                <prop key="hibernate.format_sql">true</prop>

                <prop key="hibernate.cache.use_second_level_cache">false</prop>
                <prop key="hibernate.cache.use_query_cache">false</prop>
                <prop key="current_session_context_class">thread</prop>
                <prop key="hibernate.current_session_context_class">org.springframework.orm.hibernate4.SpringSessionContext</prop>
            </props>
        </property>
        <property name="packagesToScan">
            <list>
                <value>com.example.entity</value>
                <!--千万注意此处，要跟着项目名字变化！否则扫描不到实体，自然无法完成与数据库的映射-->
            </list>
        </property>
    </bean>

    <!-- 配置事务管理器 -->
    <bean id="transactionManager" class="org.springframework.orm.hibernate4.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>

    <!--下面的路径要换成自己具体的路径！不然会出错的-->
    <aop:config proxy-target-class="true">
        <aop:advisor pointcut="execution(public * com.example.service.*Service.*(..))" advice-ref="txAdvice"/>
    </aop:config>

    <tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true"/>

    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="get*" read-only="true" propagation="REQUIRED"/>
            <tx:method name="list*" read-only="true" propagation="REQUIRED"/>
            <tx:method name="find*" read-only="true" propagation="REQUIRED"/>
            <tx:method name="save*" propagation="REQUIRED" />
            <tx:method name="delete*" propagation="REQUIRED" />
            <tx:method name="update*" propagation="REQUIRED" />
            <tx:method name="add*" propagation="REQUIRED" />
            <tx:method name="*" propagation="REQUIRED" rollback-for="Exception"/>
        </tx:attributes>
    </tx:advice>
    
   </beans>
```
&nbsp;&nbsp;&nbsp;&nbsp;然后是spring-mvc.xml
spring-mvc.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                        http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
                        http://www.springframework.org/schema/context
                        http://www.springframework.org/schema/context/spring-context-4.0.xsd
                        http://www.springframework.org/schema/mvc
                        http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd">

    <!-- 扫描注解 过滤掉Service和Repository，只扫描Controller -->
    <context:component-scan base-package="com.example" >
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Service"/>
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Repository"/>
    </context:component-scan>

    <!--配置静态资源-->
    <mvc:resources location="/static/js/" mapping="/js/**"/>
    <mvc:resources location="/static/img/" mapping="/img/**"/>
    <mvc:resources location="/static/css/" mapping="/css/**"/>

    <!--配置SpringMVC注解对应的内容-->
    <!--这个是RequestMappingHandlerMapping拦截器，对应即为Controller前面的@RequestMapping注解-->
    <!--HandlerMapping 作用如下：
        SpringMVC工作过程中，DispatcherServlet收到请求之后会把请求交给HandlerMapping，
        HandlerMapping根据配置（此处为注解）找到请求对应的Handler-->
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"/>

    <!--这个是RequestMappingHandlerAdapter-->
    <!--HandlerAdapter 作用如下：
        SpringMVC工作过程中，HandlerMapping找到Handler，Handler对具体的处理（Controller）进行封装，
        然后由HandlerAdapter对Handler进行具体处理
        在这里我们定义了返回对象自动转换为Json数据-->
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
        <property name="cacheSeconds" value="0"/>
        <property name="messageConverters">
            <list>
                <ref bean="mappingJackson2HttpMessageConverter"/>
                <ref bean="mappingStringHttpMessageConverter"/>
            </list>
        </property>
        <property name="webBindingInitializer" ref="webBindingInitializer">
        </property>
    </bean>

    <!--设置文件上传的bean，如果项目中不需要上传文件可以去掉-->
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <property name="maxUploadSize" value="10485760" />
    </bean>

    <!--这里就是Spring对http消息格式转换提供的接口-->
    <bean id="mappingStringHttpMessageConverter" class="org.springframework.http.converter.StringHttpMessageConverter">
        <property name="supportedMediaTypes">
            <list>
                <value>text/plain;charset=UTF-8</value>
                <value>application/json;charset=UTF-8</value>
            </list>
        </property>
    </bean>

    <!--同上-->
    <bean id="mappingJackson2HttpMessageConverter" class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
        <property name="supportedMediaTypes">
            <list>
                <bean class="org.springframework.http.MediaType">
                    <constructor-arg index="0" value="application"/>
                    <constructor-arg index="1" value="json"/>
                    <constructor-arg index="2" value="UTF-8"/>
                </bean>
            </list>
        </property>
    </bean>

    <!--注册转换器到SpringMVC ，并注册在了上面的RequestMappingHandlerAdapter中-->
    <bean id="webBindingInitializer" class="org.springframework.web.bind.support.ConfigurableWebBindingInitializer">
        <property name="conversionService">
            <bean class="org.springframework.core.convert.support.DefaultConversionService"></bean>
        </property>
    </bean>

    <!-- 添加viewResolver -->
    <!--viewResolver的作用是接收到返回的ModelAndView的时候把它处理成视图（jsp文件）-->
    <bean id="viewResolver" class="org.springframework.web.servlet.view.ContentNegotiatingViewResolver">  
        <property name="viewResolvers">
            <list>
                <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
                    <property name="order" value="2"/>
                    <property name="prefix" value="/WEB-INF/views/" />  
                    <property name="suffix" value=".jsp" /> 
                </bean>
            </list>
        </property>
         
    </bean>  
</beans> 
```
&emsp;&emsp;里面我写的注释挺多了，记得结合上面的工作流程理解配置，两相比对！
## 资源下载
&emsp;&emsp;代码我已经上传到百度云啦，传送门如下：  
&emsp;&emsp;[点我下载](https://pan.baidu.com/s/1nzZFhqNHtalKmgg8q0QnRA)    
&emsp;&emsp;OK，好好学习，天天向上，加油！