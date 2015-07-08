---
layout: post
title: 代理模式
description: 作为AOP不可或缺的技术，本文探究下java本身提供的动态代理技术
category: blog
---

## why

有时候需要对实际对象进行控制，我们把zheceng访问控制封装成一个新的代理对象来代替实际对象，交由客户直接访问，由此引入代理模式。

## what

一般UML视图如下：
![Simple Factory](/images/design_pattern/proxy.png)

本图需要注意几个问题

1. client对象直接操作Proxy对象
2. Proxy和ServiceImpl类对外提供共同的服务接口
3. Proxy聚合ServiceImpl

客户端可能采用如下的方式调用服务:

```
client.invoke(new Proxy(new ServcieImpl))
```

这种方式的优点是，在没有更改既有服务的前提下，为了满足一个特定的场景，对服务做了增强。

## How

### 静态代理

考虑简单的下单服务：改服务定义了选择商品，下单，支付三个操作，实现如下:

```
interface Order {
  public void selectProduct(Product product);
  public void submit();
  public void pay();
}

class DefaultOrder implements Order {
  private Product product;

  public void selectProduct(Product product){}
  public void submit(){};
  public void pay(){};
}

class Customer {
  public void createOrder() {
    Product product = new Pen();
    Order order = new DefaultOrder();
    order.selectProduct(pen);
    order.submit();
    order.pay();
  }
}
```

ok，那么现在增加了新的需求，需要记录每个用户生成订单过程中做的所有操作. 在不破坏既有业务的前提下，使用代理模式重构代码.

```
class Log implements Order {
  private Product product;
  private Order order;

  public Log(Order order) {
    this.order = order;
  }

  public void selectProduct(Product product){
    System.out.println("select product");
  }
  public void submit(){
    System.out.println("submit order");
  }
  public void pay(){
    System.out.println("pay");
  }
}

class Customer {
  public void createOrder() {
    Product product = new Pen();
    Order order = new DefaultOrder();
    Order logOrder = new Log(order);
    logOrder.selectProduct(pen);
    logOrder.submit();
    logOrder.pay();
  }
}

```

以上为一个简答的静态代理实现，但是明显上面的日志下单有不足的地方，即如果需要为所有的下单操作做日志，如果新增了其他的操作，则需要同时修改代理类；

### 动态代理

所谓动态代理，即对象在运行时才决定
Java本身提供了动态代理的实现机制，Proxy类和InvocationHandler接口:

* Proxy类提供了一个静态生成代理类的方法newProxy(ClassLoader loader, Class<?>[] interfaces, InvocationHandler handler)
* InvocationHandler对象集中控制所有的代理方法,实现这个接口的时候只需要实现invoke方法,invoke(Object proxy, Method method, Object[] args)
* 当调用Proxy对象的方法时，它会将所有的请求分发到invoke方法中，由invoke负责具体的调用逻辑，可以直接调用method.invoke(proxy, args)，也可以在前后混入其他的逻辑

下面用动态代理的方式重构之前的代码

* 定义代理类
* customer直接操作代理类

```
public class OrderServiceProxy implements InvocationHandler {
    private OrderService service;

    public OrderServiceProxy(OrderService service) {
        this.service = service;
    }

    public static OrderService proxyService(OrderService service) {
        return (OrderService) Proxy.newProxyInstance(
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
    OrderService orderService = OrderServiceProxy.proxyService(new DefaultOrderService());

    orderService.selectProduct(product);
    orderService.submit();
    orderService.pay();
}
```

这里对比customer的使用方式的转变:

```
public void createOrder(Product product) {
    OrderService orderService = new DefaultOrderService();

    orderService.selectProduct(product);
    orderService.submit();
    orderService.pay();
}

public void createProxyOrder(Product product) {
    OrderService orderService = OrderServiceProxy.proxyService(new DefaultOrderService());

    orderService.selectProduct(product);
    orderService.submit();
    orderService.pay();
}
```

这里动态代理可以为所有的方法做拦截

## Conclusion

代理模式与其说是为了增强服务的功能，还不如说是拦截服务的功能。
