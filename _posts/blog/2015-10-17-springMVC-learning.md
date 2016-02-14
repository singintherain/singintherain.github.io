---
layout: post
title: SpringMVC 学习记录
category: blog
description: 梳理SpringMVC的工作机制
---

Web容器在启动时，SpringMVC的IoC容器加载策略如图:

![spring_context_load](/images/springMVC/spring_mvc.png)

## Web容器上下文

![一定要区分Ioc容器和Web容器的区别]

SpringMVC是建立在Ioc容器之上的，tomcat在加载web.xml的时候，根据web.xml的配置的Web容器拦截器将IoC容器载入到Web环境来, tomcat是个Web容器

DispatcherServlet起到分发请求的作用, 后面的servlet-mapping定义了这个DispatcherServlet对应的URL映射
这些URL映射为这个Servlet指定了需要处理的HTTP请求, 可以有多个DispatcerServlet, 每个DispatcherServlet对应不同的url

ContextLoaderListener监听器与Web服务器的生命周期相关联，由ContextLoaderListener负责完成IoC容器在Web环境中的启动工作
ContextLoaderListener实现了ServletContextListener接口，它是Tomcat启动一个Webapp的入口

DispatcherServlet和ContextLoaderListener提供了在`Web容器`中对Spring的接口

ServletContext为Spring IoC容器提供了一个宿主环境, 在宿主环境中，SpringMVC建立起
一个IoC容器的体系，这个体系通过ContextLoaderListener初始化建立的，IoC容器建立起来后，
把DispatchServlet作为SpringMVC处理Web请求的转发器建立起来，从而完成响应http请求的准备

为方便在`Web环境`中使用IoC容器，Spring为Web应用提供了上下文的扩展接口WebApplicationContext
该类的方法getServletContext()返回当前`Web容器`的Servlet上下文环境

* ViewResolver
视图解析器负责将action返回的逻辑视图名字符串解析为具体的视图对象

* request的参数绑定

如果是表单提交数据，是如何做到数据到具体对象的映射的？如果涉及对象比较复杂，还可不可以自动映射对象关系？

## 配置文件加载

WebApplicationContext是子容器，ApplicationContext是父容器

Web容器在加载的时候，默认的WebApplicationContext的配置文件是/WEB-INF/applicationContext.xml
同时他还会自动加载在/WEB-INF目录下的所有以.xml结尾的xml文件

WebApplicationContext是根上下文，但是它有双亲上下文

DispatcherServlet持有一个上下文，它是根上下文的子上下文

## 疑问

- 区分server.xml和applicationContext.xml

- servlet-mapping和注解指定的action/url映射有什么关系

- 什么情况下配置多个servlet

- 一个简单的spring webapp如何发布到tomcat中

参考![deploy war to tomcat](http://www.mkyong.com/maven/how-to-deploy-maven-based-war-file-to-tomcat/)

进入tomcat的conf目录，在tomcat-users.xml中增加如下配置:

```
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<user username="tomcat" password="tomcat" roles="manager-gui, manager-script"/>
```

启动tomcat, 进入localhost:8080/manager/html，这个是对所有webapp的管理页面

`可以在该管理页面直接deploy webapp 的war包, 也可以通过在maven项目中直接发布该项目war包到远程tomcat`

maven项目中安装插件:

```
<plugin>
    <groupId>org.apache.tomcat.maven</groupId>
    <artifactId>tomcat7-maven-plugin</artifactId>
    <configuration>
    <server>myTomcatServer</server>
    <path>/simple_spring_web</path>
    <url>http://localhost:8080/manager/text</url>
    </configuration>
</plugin>
```

如果用的容器是tomcat8，也可以用该插件发布

`注意url为http://localhost:8080/manager/text`

在本机maven配置中指定myTomcatServer的配置，一般都会有用户名和密码, 修改.m2/setting.xml，

```
<servers>
    <server>
        <id>myTomcatServer</id>
        <username>tomcat</username>
        <password>tomcat</password>
    </server>
</servers>

```

`泪哈，没搞清楚jar包和war包的区别，果真半路出家都是坑啊`

在项目中，执行mvn tomcat7:deploy，

发布成功后，项目默认是启动状态，这是访问webapp时url路径发生变化，
比未发布之前开发环境启动webapp访问资源增加了webapp在tomcat容器中的名称的路径，例如

```
发布之前：
http://localhost:9000/user/register.html
发布之后:
http://localhost:8080/simple_spring_web/user/register.html
simple_spring_web是webapp在tomcat容器中的名称
```

- spring如何直接获取当前session

- 如何form表单传递的是个复合对象，如何处理

1. 情形1：存在form表单对象的属性包含多个其他的自定义对象

```
public class User {
  private String userName;
  private String password;
  private String realName;
  private Address address;
  
  get..
  set..
}
public class Address {
  private String street;
  private String country;

  get..
  set..
}
```

定义上面的两个对象User类和Address类，对属性分别定义get和set方法，则在form表单可以这样声明：

```
<tr>
<td>用户名:</td>
<td><input type="text" name="userName"/></td>
</tr>
<tr>
<td>密码:</td>
<td><input type="password" name="password"/></td>
</tr>
<tr>
<td>姓名:</td>
<td><input type="text" name="realName"/></td>
</tr>
<%--<tr>--%>
<%--<td>性别:</td>--%>
<%--<td><input type="text" name="sex"></td>--%>
<%--</tr>--%>
<tr>
<td>地址: 所属国家</td>
<td><input type="text" name="address.country"></td>
</tr>
<tr>
<td>地址: 所属街道</td>
<td><input type="text" name="address.street"></td>
</tr>
```

采用字符串以'.'号为分隔符进行级联定义，在action处可以得到正确的嵌套Address对象的User对象， 它的定义规则和于Bean类的定义规则一致
同时，这种级联方式支持多深度的级联

2. 情形2：form表单对象的属性有的是Enum枚举类型

- 为什么在applicationContext.xml中使用context:component:scan不能将服务加载，在servlet.xml中就可以加载

- Spring中的发布和接收定制的事件

1. 事件传播

ApplicationContext基于Observer模式提供了针对Bean的事件传播功能。通过Appliation.publishEvent方法，我们可以将事件通知给系统的所有的ApplicationListener.
事件传播的一个典型应用是，当Bean中的操作发生异常（如数据库连接失败），则通过事件传播机制通知异常监听器进行处理。

每个Controller都是一个Bean，如果实现定制了ControllerAdvice, controller的某个action抛出异常，通过事件传播机制传递给ControllerAdvice?（是这样的工作模式吗）

## http请求处理线程

在一个http连接上的同一个请求，后台服务器处理的线程都不一样，每次都会新建一个处理线程
来处理每次的请求
同一个http链接，sessionId是相同的，session信息保存在服务器端，仅仅向前端暴露sessionId
虽然每次请求的处理线程都不同，但是所有的线程都会共享同一个servletContext上下文

### ServletMapping VS HandlerMapping

这是两个不同的概念，ServletMapping是指明url到servlet的映射关系，HandlerMapping是标识最终的路由信息

一个url请求，先通过ServletMapping找到对应的处理servlet，在通过HandlerMapping找到最终的处理controller

#### ServletMapping的使用方法

在一个webapp中可以定义多个servlet，即在web.xml中声明多个servlet，并同时定义多个servletMapping
类似:

```
<servlet>
    <servlet-name>user</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
    <servlet-name>user</servlet-name>
    <url-pattern>/user/*</url-pattern>
</servlet-mapping>

<servlet>
    <servlet-name>student</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
    <servlet-name>student</servlet-name>
    <url-pattern>/student/*</url-pattern>
</servlet-mapping>

<servlet-mapping>
    <servlet-name>student</servlet-name>
    <url-pattern>/student/*.jsp</url-pattern>
</servlet-mapping>
```

这里需要注意:

* 每个声明的DispatcherServlet就是一个单独的servlet
* url-pattern一定不要出现重合
* 同一个servlet可以对应多个servletMapping
* 每一个servlet在加载时会自动在约定的/WEB-INF/目录下找对应的servlet配置文件，如果不特殊指定的话
* 可以在servlet声明的时候，使用<init-param>自定义配置文件，允许多个加载配置文件
* 每一个servlet都是独立的，最终都会根据名字(servlet-name)绑定到ServletContext中
* 每一个servlet都可以定义自己的各个元素，如MultipartResolver/LocaleResolver/ExceptionResolver等
* 每一个servlet实际上就是一个实现了ApplicationContextAware接口的类

Tomcat和Jetty这些Web容器只认ServletContext，不认WebApplicationContext

疑问：

在注册了多个Servlet的Web容器中，是如何根据url来查找到具体的业务处理Servlet的？

#### HandlerMapping的使用方法

一般每个HandlerMapping可以持有一系列从URL请求到Controller的映射

通过一次遍历注册的HandlerMapping来获得可以处理HttpServletRequest的Handler
最终返回的是一个HandlerExecutionChain

