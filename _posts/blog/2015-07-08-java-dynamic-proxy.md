---
layout: post
title: 代理模式
description: 作为AOP不可或缺的技术，本文介绍下java原生的和CGLib提供的动态代理使用方法
category: blog
---

## why

有时候需要对实际对象进行控制，我们把zheceng访问控制封装成一个新的代理对象来代替实际对象，
交由客户直接访问，由此引入代理模式。

为什么不直接使用对象

* 对象不能直接使用，比如说该对象实际是一个远程的服务对象
* 增强对象的功能; 如果在原有的对象上修改，则会破坏对象职责的单一性，比如给对象附加日志操作

## what

以最简单的静态代理为例：

一般UML视图如下：
![Simple Factory](/images/design_pattern/proxy.png)

本图需要注意几个问题

1. client对象直接操作Proxy对象
2. Proxy和ServiceImpl类对外提供共同的服务接口
3. Proxy内部依赖于ServiceImpl


```
interface Service{
  public void do();
}

class ConcreteService implements Service{
  public void do() {
    System.out.println("concrete service do");
  }
}

class ProxyService implements Service {
  private User user;
  private Service service = new ConcreteService();

  public ProxyService(User user) {
    this.user = user;
  }

  public void do() {
    boolean isValid = AuthenticationService.auth(this.user);

    if(isValid) {
      this.service.do();
    } else {
      System.out.println("You are invalid!");
    }
  }
}
```

客户端可能采用如下的方式调用服务:

```
User user = new User();
Service proxyService = new ProxyService(user);
proxyService.do();
```

优点:

* client像使用真实的服务对象一样使用proxy；
* 代理类可以增加额外的功能逻辑

使用场景：

* 远程服务调用
* 缓存对象
* 权限控制

## How

### 装饰模式

一般UML视图如下：
![Simple Factory](/images/design_pattern/decorator.png)

本图需要注意几个问题

1. client对象直接操作Proxy对象
2. Proxy和ServiceImpl类对外提供共同的服务接口
3. Proxy聚合ServiceImpl

考虑一个订单服务，他有一个计算价格的接口，实现如下:

```
interface OrderServiceInterf {
  public float caculatePrice();
}

class DefaultOrder implements OrderServiceInterf {
  private Product product;

  public DefaultOrder(Product product) {
    this.product = product;
  }

  public float caculatePrice(){
    return 3.0F;
  }
}
```

ok，那么现在增加了新的需求，下单可以使用优惠券，而优惠券订单的价格，是在原始价格的基础上扣掉
优惠券所抵金额。

```
class CouponOrder implements OrderServiceInterf {
  private Coupon coupon;
  private OrderServiceInterf orderService;

  public CouponOrder(OrderServiceInterf orderService) {
    this.orderService = orderService;
  }

  public void setCoupon(Coupon coupon) {
    this.coupon = coupon;
  }

  public float caculatePrice() {
    return this.orderService.caculatePrice - CouponService.getDiscountFromCoupon(this.coupon);
  }
}

```

客户端使用方式:

```
Product product = new Product();
OrderService orderService = new CouponOrder(new DefaultOrder(product));
orderService.setCoupon(new Coupon(10));
float price = orderService.caculatePrice();
```

装饰模式VS静态代理

* 代理模式不实现和目标对象类似的业务功能，而是一些和目标对象功能联系不太紧密的职责，
也就是说二者处理的问题域不同
* 装饰模式处理的问题域和目标对象相同，目标都是增强目标对象某方面的能力
* 装饰器可以动态的为目标对象增加或者减少功能，而且支持嵌套使用产生叠加效应

比如Java IO的InputStream和BufferInputStream

### 动态代理

所谓动态代理，即对象在运行时创建代理类
Java本身提供了动态代理的实现机制，Proxy类和InvocationHandler接口:

* Proxy类提供了一个静态生成代理类的方法newProxy(ClassLoader loader, Class<?>[] interfaces, InvocationHandler handler)
* InvocationHandler对象集中控制所有的代理方法,实现这个接口的时候只需要实现invoke方法,invoke(Object proxy, Method method, Object[] args)
* 当调用Proxy对象的方法时，它会将所有的请求分发到invoke方法中，由invoke负责具体的调用逻辑，可以直接调用method.invoke(proxy, args)，也可以在前后混入其他的逻辑

考虑下面的使用场景，一个下单有三个操作：选择商品、提交订单、支付

```
interface OrderServiceInterf {
  public void selectProduct(Product product);
  public void submit();
  public void pay();
}

class DefaultOrder implements OrderServiceInterf {
  private Product product;

  public void selectProduct(Product product){}
  public void submit(){};
  public void pay(){};
}

class Customer {
  public void createOrder() {
    Product product = new Pen();
    OrderServiceInterf order = new DefaultOrder();
    order.selectProduct(pen);
    order.submit();
    order.pay();
    }
}
```

考虑使用静态代理的方式为每个操作增加日志：

```
class LogOrderService implements OrderServiceInterf {
  private OrderServiceInterf defaultOrder = new DefaultOrder();

  public void selectProduct(Product product){
    log("select product");
    defaultOrder.selectProduct();
  }
  public void submit(){
    log("submit");
    defaultOrder.submit();
  }
  public void pay(){
    log("pay");
    defaultOrder.pay():
  }
}

class Customer {
  public void createOrder() {
    Product product = new Pen();
    OrderServiceInterf order = new LogOrderService();
    order.selectProduct(pen);
    order.submit();
    order.pay();
    }
}
```

不足之处：

* 这里假设日志内容大体逻辑都是一样的，即记录方法名称
* 日志功能和具体业务紧耦合，不能做到可插拔

下面用动态代理的方式重构之前的代码

```
public class OrderServiceProxy implements InvocationHandler {
    private OrderServiceInterf service;

    public OrderServiceProxy(OrderServiceInterf service) {
        this.service = service;
    }

    public static OrderServiceInterf proxyService(OrderService service) {
        return (OrderServiceInterf) Proxy.newProxyInstance(
                service.getClass().getClassLoader(),
                service.getClass().getInterfaces(),
                new OrderServiceProxy(service)
        );
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("log: " + method.getName());
        return method.invoke(service, args);
    }
}

public void createProxyOrder(Product product) {
    OrderServiceInterf orderService = OrderServiceProxy.proxyService(new DefaultOrderService());

    orderService.selectProduct(product);
    orderService.submit();
    orderService.pay();
}
```

这里对比customer的使用方式的转变:

```
public void createOrder(Product product) {
    OrderServiceInterf orderService = new LogOrderService();

    orderService.selectProduct(product);
    orderService.submit();
    orderService.pay();
}

public void createProxyOrder(Product product) {
    OrderServiceInterf orderService = OrderServiceProxy.proxyService(new DefaultOrderService());

    orderService.selectProduct(product);
    orderService.submit();
    orderService.pay();
}
```

这里动态代理可以为所有的方法做拦截

#### 缺陷

* 这种代理方式只能代理接口，即OrderService必须是interface声明，否则Proxy.newInstance时不能类型转换为OrderService。
* 即使代理的类继承之某个接口也不行
* 如果出现同类之间方法的嵌套调用，而且这两个方法都做了拦截，但是最终只拦截了一次

若想针对类做动态代理，可以使用CGLib

### CGLib

CGLib的动态代理和Java原生代理方式在使用上没有什么区别

* 代理类需要实现MethodInterceptor接口，并实现其intercept方法
* 代理对象需要使用Enhancer对象生成

```
public static OrderServiceInterf proxyService(OrderServiceInterf service) {
    Enhancer enhancer = new Enhancer();

    enhancer.setSuperclass(service.getClass());
    enhancer.setCallback(new LogOrderServiceCGLib());

    OrderServiceInterf orderService = (OrderServiceInterf) enhancer.create();

    return orderService;
}

public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
    log(method.getName());
    return methodProxy.invokeSuper(o, objects);
}

private void log(String msg) {
    System.out.println("log: " + msg);
}
```

#### CGLib VS Java Proxy

* CGLib不仅可以代理接口，还可以代理类
* Java Proxy动态生成的代理类继承之java.lang.reflect.Proxy，所有的方法指向了
InvocationHandler实现对象的invoke方法，而invoke方法把请求转发给目标对象处理；
* CGLib生成的代理类继承于被代理对象，执行代理对象的方法时，不会转发给目标对象，
而是执行自己的实际方法。
* CLib没有目标对象，只有目标类，在调用proxyService方法是会实例化出来代理对象
* JDK Proxy会先创建一个目标对象，再针对目标对象创建代理对象

