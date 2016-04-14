---
layout: post
title: Spring AOP 切点
description: 详解Spring AOP 切点的概念和使用
category: blog
---

Spring AOP的本质可以理解为在目标对象的哪个位置做何种类型增强，所谓的增强类型包括
前置增强、后置增强、环绕增强、异常抛出增强、引介增强，而在目标对象的哪个位置做增强
就是切点的概念，把切点和增强类型用类似配置的方式组合起来，就是切面的概念。

本章介绍的切点和切面的概念以及具体的用法.

## 切点

切点是指在哪个类的哪个连接点上做代理增强，Spring AOP只能做方法增强, 这里的连接点
就是指类的方法。

Pointcut切点的接口定义如下：

![aop_pointcut](/images/aop_pointcut.png)

Spring支持两种方法匹配器：静态方法匹配器和动态方法匹配器。

所谓静态方法匹配器是指只对方法名匹配（方法名、入参顺序和类型）, 静态匹配只会判断一次。
动态方法匹配器是指在方法调用的时候检查传入的对象的值, 因为每次方法调用的时候入参
可能不一样，所以每次方法调用的时候必须判断, 对性能影响较大，不常用。

### 切点类型

切点类图如下：

![pointcut_class_structure](/images/pointcut_class_struct.png)

- 静态方法切点

aop.support.StaticMethodMatcherPointcut抽象基类, 默认匹配所有的类。
两个子类NameMatchMethodPointcut和AbstractRegexpMethodPointcut, 一个提供简单字符串
匹配方法名，一个提供正则表达式匹配方法名。

- 动态方法切点

aop.support.DynamicMethodMatcherPointcut抽象基类，默认匹配所有的类。

- 注解切点

支持注解标签定义的切点

- 表达式切点

为了支持AspectJ切点表达式语法而定义的接口

- 流程切点

- 复合切点

为创建多个切点而提供的方便操作类, 组合多个ClassFilter和MethodFilter匹配规则

## 切面

所谓的切面，就是将切点和具体的增强功能使用配置的方式连接起来

切面分为三类：一般切面、切点切面和引介切面

* Advisor: 一般切面，仅包含一个Advice，切点隐含，代表所有类的所有方法
* PointcutAdvisor: 有自定义切点的切面
* IntroductionAdvisor: 引介切面，对应于引介增强，作用于类层面

### PointcutAdvisor

PointcutAdvisor主要有6个具体的实现类

具体的类图如下:

![advisor](/images/advisor.png)

* DefaultPointcutAdvisor: 最常用，通过任意Pointcut和Advice定义一个切面，不支持引介的切面类型
* NameMatchMethodPointcutAdvisor: 按方法名进行切点定义的切面
* RegexpMethodPointcutAdvisor: 正则表达式匹配方法名进行切点定义的骑马
* StaticMethodMathcerPointcutAdvisor: 静态方法匹配器切点定义的切面
* AspectJExpressioinPointcutAdvisor
* AspectJPointcutAdvisor

- 静态普通方法匹配切面

需要单独定义一个类，继承StaticMethodMathcerPointcutAdvisor，并实现matches方法，
自定义方法匹配规则。
ClassFilter默认匹配所有的类, 可以通过重写getClassFilter方法来自定义类匹配规则

- 静态正则表达式匹配切面

不需要自定义一个类，直接定义RegexpMethodPointcutAdvisor类型的Bean就行，需要给出
符合正则匹配格式的patterns规则，该规则是一个list，多个规则之间是or的关系

- 动态方法匹配器切面

可以通过继承DynamicMethodMathcerPointcut实现, 必须实现matches(Method, Class, Object[])
方法。我们可以重写它的静态检查函数，打印必要的信息。DynamicMethodMathcerPointcut默认
的静态检查函数都返回真。

动态切面并不是只执行动态检查匹配，在给代理对象生成织入切面时，会对目标类中所有的方法
进行静态切点检查；在第一次调用代理类的每个方法都会进行一次静态检查；
对于静态匹配成功的方法，在后续的调用中才进行动态切点检查。

如果不覆盖DynamicMethodMathcerPointcut的getClassFilter()和matches(Method, Class)
方法，每次调用方法时都会进行动态检查，比较耗费性能。

### 流程切面

流程切面和动态切面在某种程度上算是一类切面，两者都需要在运行期判断动态的环境。
对于流程切面来说，代理对象在每次调用目标类方法时，都需要判断方法调用堆栈是否满足
流程切点的要求。

