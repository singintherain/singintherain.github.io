---
layout: post
title: 简单工厂、工厂方法、抽象工厂
description: 工厂方法将对象的使用和对象的实例化相隔离，探究其实现原理和思想
category: blog
---

## 简单工厂

考虑下面的场景：客户需要订购不同风味的匹萨，需要开发一个匹萨预定系统

采用简单工厂的模式设计如下：
![Simple Factory](/images/design_pattern/simple_factory.png)

```
PizzaFacotory.java

class PizzaFactory{
    SimplePizzaFactory pizzaFactory = new SimplePizzaFactory();

    public PizzaStore(SimplePizzaFactory simplePizzaFactory) {
        this.pizzaFactory = pizzaFactory;
    }

    public Pizza orderPizza(String pizzaType) {
        // 简单工厂
        Pizza pizza = new SimplePizzaFactory().createPizza(pizzaType);

        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();

        return pizza;
    }
}
```

使用一个类的方法来实例化所需对象, 这个方法一般是静态方法；
设计为静态方法的缺点是子类不能去修改改静态方法，可扩展性不强；

考虑上面的orderPizza方法，它封装了一个匹萨具体的制作过程

1. 选择需要制作哪个风味的匹萨
2. 材料准备
3. 烘制
4. 切片
5. 打包

这里假定所有匹萨的制作过程都是一样的，即CheesePizza和GreekPizza都遵循了一个统一的接口

## 工厂方法

子类自行决定创建自己的对象。
设计如下：
![Simple Factory](/images/design_pattern/factory_method.png)

和简单工厂相比，发生了以下变化：

* 匹萨的种类和匹萨商店有了依赖关系，不同的商店生成了其特定的匹萨

维持不变的条件：

* 匹萨的制作仍然维持统一的流程

实现上：

* 简单工厂将匹萨对象的实例化抽离出匹萨商店
* 工厂方法将匹萨对象的实例化又集成到匹萨商店，但是只有具体的匹萨商店才会负责该商店的匹萨生成

注意：

* 一般为每个不同的对象种类定义一个工厂，工厂一般只实例化一种对象，不会出现根据参数来创建不同对象的情况
* 工厂方法比简单工厂更有弹性，实际上，上述的简单工厂在实现时违背了开闭原则，对简单工厂的重构方法就是构造工厂方法
* 简单工厂只有在实例化的对象只有一种时才有意义

## 抽象工厂

下面考虑这种情形：

1. 生产匹萨的原料工厂有很多个
2. 每个原料工厂都能产生cheese、dough等原料
3. 每个原料工厂产生的原料都有自己的特色，例如虽然都是cheese，但是纽约的原料工厂产生的cheese和Chicago的原料工厂产生的cheese是不一样的。
4. 每个匹萨店有其自己专属的原料供应商

UML设计图如下：
![Simple Design](/images/design_pattern/abstract_factory.png)

这里需要注意，Pizza类和MaterialFactory的聚合关系，Pizza根据所属PizzaStore的MaterailFactory获取原料

* 误区：
这里的抽象，可以抽象成类，也可以抽象成接口

## Conclusion

### 工厂模式的优点

个人感觉最大的优点是集中管理对象的实例化，可以避免代码的重复，方便以后的维护

### 简单工厂 VS 抽象工厂 VS 工厂方法

* 工厂和商品的对应关系
  * 简单工厂中，一个工厂生成一种商品，但是仅有一个工厂
  * 在工厂方法中，为每一个商品定义了一个工厂，但是可以有多个工厂
  * 在抽象工厂中，每个工厂可以生成多个不同种类的商品，但是会出现多个工厂
* 优缺点
  * 简单工厂不能扩展商品，否则会违背开闭原则
  * 工厂方法可以任意增加商品，但是没有不能生成多个商品
  * 抽象工厂有产品族的概念，即每个抽象工厂的实现类负责生产所属族的多个商品；缺点是其水平扩展产品族很简单，但是如果增加一个新的产品比较困难，这里需要在MaterialFacotry中新增一个createNewObject方法，而顶级接口的改变意味着所有的实现类都必须改变


