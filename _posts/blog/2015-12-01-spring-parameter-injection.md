---
layout: post
title: spring的参数注入
description: 来自于spring 3.x企业应用开发实战
category: blog
---

通过本节，你可以接触到一下内容

* spring注入参数详解
* 方法注入
* bean之间的关系
* 整合多个配置文件

## 注入参数详解

在Spring配置文件中，不仅可以将String、int等简单数据类型注入到Bean中，还可以将集合、Map等
类型的数据注入到Bean中，另外还可以注入配置文件中的其他定义的Bean.

### 参数注入声明方式

- 简单参数注入

<property name="successRate" value="2"/>

- 特指value值

```
<property name="failedRate">
    <value type="float">0.3</value>
</property>
```

- p标签

```
<bean id="abstractCar" class="com.sishuok.domain.practice.injection.Car"
p:brand="www" p:price="200000"/>
```

### 基本类型参数值

参数转换问题

- 参数转换为基本数据类型

int, float, Float, String等

boolean类型比较特殊，在传值的适合可以指定true/false，也可以指定1/0

- 参数转换为自定义类型

String类型 -> SelfClass
当是自定义类时，ref可以递归实例化bean，但是value也可以，但是它需要依赖bean提供
包含String类型的参数构造器支持

#### 定制属性编辑器

所有上面的参数转换是由spring框架自身注册的默认属性编辑器来实现的
客户定制不同类型的属性编辑器

#### 内部bean

```
<bean id="outer" class="...">
<!-- instead of using a reference to a target bean, simply define the target bean inline -->
    <property name="target">
        <bean class="com.example.Person"> <!-- this is the inner bean -->
            <property name="name" value="Fiona Apple"/>
            <property name="age" value="25"/>
        </bean>
    </property>
</bean>
```

没有id/name, 伴随着外部bean创建而创建，无scope, 不能和其他的bean协作注入等

#### Null值注入

null标签<null/>

```
<property name="brand"><null/></property>
```

#### 级联属性

必须要求预先实例化内部属性对象，实际上他是调用的getObject().setProperty()

  ```
<bean id="propertyBoss" class="com.sishuok.domain.practice.injection.HierarchyPropertyBoss">
    <property name="car.price" value="300"/>
    <property name="car.brand" value="ooo"/>
</bean>

  ```

#### 集合类型

- 集合类型属性

属性是集合类型，要求预先实例化集合对象

- 集合类型bean

如何声明本身就是集合类型的Bean, 需要借助util命名空间

#### 自动装配autowire

* 优点 使用方便
* 缺点 不能自动装配普通类型对象, 不能清晰的描述bean之间的关系

使用时注意几点：

1. byType时,不能存在多个相同类型的bean
2. byName, 通过name来查询指定bean
3. constructor, 和byType查询逻辑一致
4. 尝试autowire注解的使用

## 方法注入

解决问题：向singleton类型的bean注入prototype类型的bean，并且每次获取prototype类型的
bean时都可以获取一个新的对象

- bean实现BeanFactoryAware, 这样bean在初始化的时候会把当前的beanFactory注入到bean中

- lookup方法

bean不能是interface/abstractClass，必须是一个具体的实现类

lookup标签会激活CGLib实例化对象

- 方法替换

## bean之间的关系

bean不但可以通过ref引用别的bean，还可以声明更复杂的bean之间的关系

### 继承

将相似类的相同属性提取为抽象父类

### 依赖

depends-on区别与ref的是，ref声明了一个显式的类依赖关系，而depends-on声明的是一个隐式的
逻辑类依赖关系

spring框架在发现ref关键字时，会递归的实例化依赖的bean对象，并将bean对象赋值给主bean的参数

spring框架在发现depends-on关键字时，也会递归的实例化bean对象，但是建立两个bean之间
的引用关系，因为他们仅仅是逻辑上又依赖

### 引用

通过idRef, spring在容器启动时就可以检查在整个容器上下文是否有该id注册的bean，而不需要
在使用时才发现错误。

如果引用者和被引用者的<bean>都位于同一个xml配置文件中，使用<idref local="">的配置方式
IDE的xml分析器就可以在开发期发现引用错误

- 区分value 和 ref idref

idref同value, 传递的仅仅是字符串
ref，会递归查询引用bean

## 整合多个配置文件

使用<import>标签，在一个xml配置文件中，引入其他的配置文件，进行配置文件的集成。

```
<import resource="spring-injection.xml"/>
```

springIOC中同时import了多个id相同的bean，后面声明的bean会覆盖之前定义的bean


