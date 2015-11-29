---
layout: post
title: Hibernate Bean Validation
category: blog
description: 探究Hibernate如何实现Bean Validation
---

## 关于Bean Validation

当处理业务逻辑的时候，`数据校验`是经常性要考虑和面对的问题；应用程序必须保证输入的
数据从语义来讲是正确的，而且在现实的开发中，业务是分层的、由不同开发者维护的，
对数据校验的业务逻辑在不同的层都可能会遇到，如何统一的维护这些校验逻辑并避免冗余定义就成为一个难题。
推崇的解决方式是，将校验和域模型绑定，保证概念的一致，同时提供统一的校验执行策略.

在编程中，最常使用的是Bean Validation，它就是按照上述的原则为Bean提供校验机制的。
Bean Validation为Java Bean验证定义了相应的元数据(Annotation)模型和API, 例如@NotNull、
@Max、@Min等，在应用中用户也可以自定义约束@Constraint.

JSR统一定义了Validatoin的规范，参考[Bean Validation Specification](http://beanvalidation.org/1.1/spec/),
Hibernate Validation是Bean Validation的参考实现，Spring并不实现Bean Validation，反而是
需要集成其它的Bean Validation组件.

## Hibernate Validation

`参考官方文档[Hibernate Validation](http://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single)`

### 简单使用

`普通Student类`

```
public class Student {
    @NotNull
    private String name;

    @Email
    private String email;

    @Min(2)
    @Max(30)
    private int age;

    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public String getEmail() {
        return email;
    }
    public void setEmail(String email) {
      this.email = email;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
}
```

`测试代码`

```
public class StudentValidatorTest {
    private Validator validator;

    @Before
    public void setUp() {
        ValidatorFactory validatorFactory = Validation.buildDefaultValidatorFactory();
        validator = validatorFactory.getValidator();
    }

    @Test
    public void nameNotNullTest() {
        Student student = new Student();
        Set<ConstraintViolation<Student>> violations = validator.validate(student);
        System.out.println("出错个数: " + violations.size());
        for(ConstraintViolation<Student> violation : violations) {
            System.out.println(violation.getMessage());
        }
    }
}
```

上述为最简单的Bean Validation使用方式，下面具体介绍Validation的各个方面.

### Validation的声明方式

声明方式主要分几种：

* 属性声明

直接在属性定义时声明，如

```
    @Email
    private String email;
```

* Bean的属性值获取方法，例如get方法或者返回值是boolean的is开头方法

```
@NotNull
public String getManufacturer() {
    return manufacturer;
      
}
@AssertTrue
public boolean isRegistered() {
    return isRegistered;
      
}
```

* 类型参数约束，如集合元素、map元素、optional

```
@Valid
private List<@ValidPart String> parts = new ArrayList<>();
@Valid
private EnumMap<FuelConsumption, @MaxAllowedFuelConsumption Integer> fuelConsumption = new EnumMap<>( FuelConsumption.class  );
private Optional<@MinTowingCapacity(1000) Integer> towingCapacity = Optional.empty();
```

* 还有特殊的泛型

```
private GearBox<@MinTorque(100) Gear> gearBox;

public class GearBox<T extends Gear> {

  private final T gear;

  public GearBox(T gear) {
      this.gear = gear;
        
  }

  public Gear getGear() {
      return this.gear;
        
  }

}

public class Gear {
  private final Integer torque;

  public Gear(Integer torque) {
      this.torque = torque;
        
  }

  public Integer getTorque() {
      return torque;
        
  }

  public static class AcmeGear extends Gear {
    public AcmeGear() {
          super( 100  );
              
    }
      
  }

}
new GearBox<>( new Gear.AcmeGear()  )
```

以上所有的验证都可以使用Set<ConstraintViolation<Car>> constraintViolations = validator.validate( car  )
的方式来触发验证逻辑.

* 类级别

```
@ValidPassengerCount
public class Car {
  private int seatCount;
}
```

* Constraint继承

在父类或者接口中定义的约束，在子类或者实现类中都可以被继承，哪怕子类重写了父类的
方法。

* 传递验证

例如：

```
public class Car {
    @NotNull
    @Valid
    private Person driver;
}
public class Person {
    @NotNull
    private String name;
}
```

通过@Valid标志需要传递约束验证，当Car的driver验证不为null后，继续验证person的name
不能为空；因为这个验证是传递性的，因此需要避免出现死循环.

传递验证不光作用在1-1的对象关系，1-N的对象关系也可以。如果属性是个集合：Array、
Colleciton、List、Set、Map，都会递归的校验所有的对象.

### Validation验证方式

* validator校验器对象的获取

需要先获取ValidatorFactory，从ValidatorFactory中生成validator，最简单的是Validation.buildDefaultValidatorFactory();
它会从默认配置中实例化出一个validator对象。

* 校验方法

主要由三种，每种方法都会返回一个Set<ConstraintViolation>，保存有失败校验集合.

1. validate()方法，校验所有定义的约束
2. validateProperty()方法，可以指定校验某个对象的指定属性
3. validateValue()方法，验证给指定类的某个属性如果指定某个值，约束是否可以生效

`@Valid传递验证，在validateProperty和validateValue中都不能起作用`

### 简介ConstraintViolation的方法

如果Student内部有个home属性为Address对象，该对象的内容验证未通过

| Method                  |                                     Usage | Example                                    |
| ----------------------- |------------------------------------------ | ------------------------------------------ |
| getMessage              |                            添加的错误信息 | may not be null                            |
| getMessageTemplate      |                          非添加的错误信息 | {javax.validation.constraints.Min.message} |
| getRootBean             | 获取被验证的根对象,相对于递归传递验证来说 | student                                    |
| getRootBeanClass        |                        被验证的根的类对象 | Student                                    |
| getLeafBean             |      出错时的对象，递归验证时，出错的对象 | Address对象                                |
| getPropertyPath         |                                  属性路径 | home.name                                  |
| getInvalidValue         |                                  属性的值 | null                                       |
| getConstraintDescriptor |                                约束描述符 | ..                                         |


了解它的具体信息，为之后的定制化错误信息服务

## 普通方法的Validaton

上面讨论了有关Bean类的验证方式，有属性级别的、属性值获取方法级别的、类型参数级别的、
类级别的等。在真正的使用时，还会遇到普通的方法级别的、构造方法级别的两种，下面来
讨论如何做普通方法和构造方法的验证。

方法级别的验证无非是前向验证和后向验证两种，即方法调用前参数必须满足什么样的条件，
方法调用后返回值必须满足什么样的条件。

示例:

```
public class RentalStation {
  public RentalStation(@NotNull String name) {
      //...
  }
  public void rentCar(
        @NotNull Customer customer,
        @NotNull @Future Date startDate,
        @Min(1) int durationInDays
      ) {
          //...
  }
}
```

