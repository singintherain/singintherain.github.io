---
layout: post
title: Java注解
description: 了解Java注解的基本知识，便于理解beanValidation和spring等框架对Annotation的使用
category: blog
---

## Why

`简化配置`

如在Spring的使用中，需要定义繁杂的xml配置，使用配置组织对象之间的关系，但是这样
显得过于复杂，而且配置文件和源代码分离，经常遇到代码不同步问题; 而使用注解，在
源代码中直接描述对象之间的关系，显得更自然和易懂。

## What

`官方注解`: 注解（也被称为元数据）为我们在代码中添加信息提供了一种形式化的方法,
使我们可以在之后某个时刻非常方便的使用这些数据

## How

### 基本语法

`示例`

```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface UseCaseAnnotation {
    public int id();
    public String descrption() default "no descripiton";

}
```

Target声明注解的使用位置，method、class、field等.
Retention声明注解被解释的时间，例如source、class、runtime，不懂，只用过runtime
@interface声明定义一个注解
内部定义元素，如id()、description(); 可以理解为属性的简化定义，属性的set方法为直接
=赋值，属性的get方法为自定义方法返回值，如id()；可以指定默认值

### 简单使用

```
public class PasswordUtil {
    @UseCaseAnnotation(id = 1, descrption = "验证密码是否正确")
    public boolean validatePassword() {
            System.out.println("验证密码是否正确");
                    return false;
    }
    @UseCaseAnnotation(id = 2, descrption = "加密密码")
    public void encryPassword() {
            System.out.println("加密密码");
    }
    @UseCaseAnnotation(id = 3, descrption = "验证是否是新密码")
    public boolean checkForNewPassword() {
            System.out.println("验证是否是新密码");
                    return false;
    }
}
```

可以通过反射技术获取一个类里面定义的Method，然后查询Method定义的Annotation, 拿到
Annotation后就可以调用上述定义的方法.

### 高级使用之嵌套定义

注解是不支持继承的，但是可以使用组合嵌套的方式来组织不同的注解关系。如：定义一个
数据库表有关的注解

```
@Target(ElementType.TYPE) //只能作用于类
@Retention(RetentionPolicy.RUNTIME)
  public @interface DBTable {
      public String name() default ""; // 数据库表名

  }
```

数据库字段注解

```
数据库字段约束声明
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
  public @interface Constraints {
  boolean primaryKey() default false;
  boolean allowNull() default true;
  boolean unique() default false;

  }

数据库字段是String类型声明
  @Target(ElementType.FIELD)
  @Retention(RetentionPolicy.RUNTIME)
  public @interface SQLString {
  int value() default 0; // 代表字段长度
  String name() default "";
  Constraints constraints() default @Constraints;

  }

数据库字段是整型
  @Target(ElementType.FIELD)
  @Retention(RetentionPolicy.RUNTIME)
  public @interface SQLInteger {
  String name() default "";
  Constraints constraints() default @Constraints;

  }
```

使用示例：

```
@DBTable(name = "members")
public class Member {
    @SQLString(30)
    private String firstName;
    @SQLString(50)
    private String lastName;
    @SQLInteger
    private int age;

    @SQLString(value = 30, constraints = @Constraints(primaryKey = true))
        String handle;

    public String getFirstName() {
            return firstName;
    }

    public String getLastName() {
            return lastName;
    }

    public int getAge() {
            return age;
    }

    public String getHandle() {
            return handle;
    }
}
```

定义注解处理器

```
public class TableCreator {
  public void create(String classFileName) throws Exception {
    try {
       Class<?> clazz = Class.forName(classFileName);

           DBTable dbTable = clazz.getAnnotation(DBTable.class);

           if(dbTable == null) {
                           System.out.println("No DBTable defined in class " + classFileName);
           }

           String tableName = dbTable.name();

           if(tableName.length() < 1) {
                           tableName = clazz.getName().toUpperCase();
           }

           List<String> columns = new ArrayList<String>();
           for(Field field : clazz.getDeclaredFields()) {
                String columnName = null;
                Annotation[] annotations = field.getDeclaredAnnotations();
                if(annotations.length < 1) {
                                    continue; // not a table column
                }

                if(annotations[0] instanceof SQLInteger) {
                       SQLInteger sqlInteger = (SQLInteger)annotations[0];

                       if(sqlInteger.name().length() < 1) {
                            columnName = field.getName().toUpperCase();
                       } else {
                            columnName = sqlInteger.name();
                        }

                    columns.add(columnName + " INT" + getConstraints(sqlInteger.constraints()));
                }

                if(annotations[0] instanceof SQLString) {
                   SQLString sqlString = (SQLString) annotations[0];
                   if(sqlString.name().length() < 1) {
                       columnName = field.getName().toUpperCase();
                   } else {
                       columnName = sqlString.name();
                   }
                       columns.add(columnName + " VARCHAR(" + sqlString.value() + ")" +
                           getConstraints(sqlString.constraints()));
               }
  }

              StringBuilder createCommand = new StringBuilder("CREATE TABLE " + tableName + "(");
              for(String column : columns) {
                  createCommand.append("\n " + column + ",");
              }
               //Remove trailing comma
              String tableCreate = createCommand.substring(
                                  0, createCommand.length() - 1
              ) + ");";
              System.out.println("Table Create SQL for " +
              classFileName + " is : \n" + tableCreate);
    } catch (ClassNotFoundException e) {
                System.out.println(e.getMessage());
                throw e;
    }
  }
  private String getConstraints(Constraints constraints){
          String cons = "";

          if(!constraints.allowNull()) {
                      cons += " NOT NULL";
          }
          if(constraints.primaryKey()) {
                      cons += " PRIMARY KEY";
          }
          if(constraints.unique()) {
                      cons += " UNIQUE";
          }

          return cons;
  }
}
```

使用实例

```
String classFileName = "lvsong.club.Member";

TableCreator tableCreator = new TableCreator();

tableCreator.create(classFileName);
```

结果

```
Table Create SQL for lvsong.club.Member is :
CREATE TABLE members(
    FIRSTNAME VARCHAR(30),
    LASTNAME VARCHAR(50),
    AGE INT,
    HANDLE VARCHAR(30) PRIMARY KEY
    );
```

## 总结

以上为Java Annotatoin的基本概念和使用的介绍，在现实使用中接触最多的是在有关JavaBean
的数据完整性校验工作中注解的使用，以及Spring、Hibernate框架中注解的使用，这些后期
文章会一一深入研究。
