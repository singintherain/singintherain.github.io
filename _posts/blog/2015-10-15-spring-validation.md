---
layout: post
title: Spring 校验器
category: opinion
description: 探究Spring是如何实现对象的属性校验功能的，同时对比Ruby的实现策略
---

## 检验器

校验器是最常遇到的一项业务规则，例如校验用户从前端传递的对象是否符合特定的规则，后者是将对象保存到数据库之前，校验对象是否满足特定的规则。下面简单看下在Rails中是如何定义Validation的.

### Rails Validation

Rails的Validation是为了保证对象在写入数据库之前是合法的，因此只针对于ActiveRecorde对象，即数据库关联对象。

```
假设数据库关联对象User中有一个属性name

Class User < ActionRecord::Base
    validates_uniqueness_of :name
    validates :password, format: { with:/\A[a-zA-Z]+\z/ }
end

user = User.new
user.name = "lvsong"
user.password = "123asfd"

user.save

if user.errors.present?
  puts user.errors.messages
end
```

如上，user对象在save的时候会触发validates_uniqueness_of :name和passord的格式校验，当校验不通过时，会将默认的出错信息加入user对象中(Ruby是一个动态语言，它可以在运行态改变对象的属性和方法)。切记，name校验是需要和后台数据库交互来判断name是否是唯一的，但是password的校验是不需要和数据库交互的;现在假设passport校验先执行，并且passord没有校验通过，这就不会继续执行后续的操作，这就保证每次对数据库的操作时，对象尽量是正确的。

下面对比常用的几项校验模式:

1. 数据库约束

在数据库层面定义约束，例如字段非null，唯一性约束;
缺点：不容易测试和维护;
优点：如果多个应用公用一个数据库的话，可以定义数据库层面的约束；个人感觉无论什么情况下都有必要建立数据库层面的约束，
这是最后一道防线，如果感觉可能对数据库性能造成影响，可以配合其他的几种校验模式使用.

`约定在数据库中禁止建外键`

2. 客户端校验

前端JS校验。
缺点：能够被用户绕过，从而从服务器造成伤害

3. Controller级别的校验

提倡在Controlelr级别对输入对象增加校验，但是仍然不能影响到数据库层级；而且Controller的业务设计应尽量的简单，不要混合过多的业务。
实际上，这一层的校验一般和前端校验差不多，仅仅是实现者不同而已。

4. Model级别的校验

保证插入到数据库中的数据是合法的.

### Spring检验器

Spring的校验器validation主要由Validator和DataBinder类组成；
在介绍这两个组件之前，需要先了解BeanWrapper；BeanWrapper和DataBinder需要用PropertyEditor来解析和格式化类的属性值，
spring3之后，提供了core.convert组件format更加便利的格式化UI用户输入数据.

MessageCodeResolver:错误信息解析

BeanWrapper包裹Bean


