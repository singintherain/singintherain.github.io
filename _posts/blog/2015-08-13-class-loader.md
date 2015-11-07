---
layout: post
title: 类加载器
description: 了解Java类加载器
category: blog
---

## Why

class文件只有被加载到虚拟机中才有作用，无论该class文件是如何生成的，这样需要定义
一个加载器负责从何种位置导入class文件并在虚拟机中生成Class对象。

## What

类加载器是一个定义在JVM之外的组件，它的功能就是根据类的全限定名称来获取描述此类的
二进制字节流;虽然类加载器的功能只用于实现类的加载动作，但是他在虚拟机中的作用远远
不限于类加载阶段。

## How

### 自定义classLoader对象

Java原生定义了一个抽象类ClassLoader，自定义的加载器只需要实现该抽象类即可；较为
简单的classLoader只需要实现其loadClass(String name)，即从文件中获取class文件内容，
根据文件内容生成内存中的Class即可；具体步骤分为：

* 读取文件流
* 将流中的内容读入字节数组
* 通过该字节数组defineClass

`示例`

```
ClassLoader classLoader = new ClassLoader() {
   @Override
   public Class<?> loadClass(String name) throws ClassNotFoundException {
      String fileName = name.substring(name.lastIndexOf(".") + 1) + ".class";
      InputStream inputStream = getClass().getResourceAsStream(fileName);
      if(inputStream == null) {
            return super.loadClass(name);
      }
      try {
            byte[] b = new byte[inputStream.available()];
            inputStream.read(b);
            return defineClass(name, b, 0, b.length);
      } catch (IOException e) {
                          throw new ClassNotFoundException(name);
      }
   }
};
```

`切记`

在虚拟机中需要由类以及它的类加载器一同来确定该类的唯一性，否则即使同一个class文件，
如果由不同的加载器加载到同一个虚拟机中，这两个类也不相等(Class类的equals、isAssignableFrom、
isInstance方法返回的结果)。借用上面自定义的classLoader对象，校验类是否相同。

```
Object object = classLoader.loadClass("lvsong.club.StudentBean").newInstance();
System.out.println(object.getClass());
Assert.assertFalse(object instanceof lvsong.club.StudentBean);
```

输出为: class lvsong.club.StudentBean，而且object对象非lvsong.club.StudentBean的实例，
这是因为虚拟机中存在了两个lvsong.club.StudentBean类，一个是系统应用程序加载器加载的，
还有一个是自定义加载器加载的, 虽然来源都一样，但是是两个独立的类.

### 系统定义加载器

系统默认定义了三种加载器，我们的应用程序就是在这三种加载器和自定义加载器互相配合
进行加载的。

* BootstrapClassLoader类启动加载器

负责将JAVA_HOME/lib目录中的，或者被-Xbootclasspath参数指定的路径的，并且能不虚拟机
识别的类库加载到虚拟机中去。启动类加载器不能直接被Java程序使用

* ExtensionClassLoader扩展类加载器

这个加载器由sun.misc.Launcher$ExtClassLoader实现，它负责加载JAVA_HOME/lib/ext目录
中的，后者被java.ext.dirs系统变量所指定的路径中的所有类库，开发者可以直接使用该
加载器。

* ApplicationClassLoader应用程序加载器

这个加载器由sun.misc.Launcher$AppClassLoader实现。通过getSystemClassLoader()可以获取它，
因此又称为系统类加载器，它负责加载用户类路径ClassPath上指定的类库。开发者可以直接使用
该类加载器，一般情况下如果应用程序没有自定义过加载器，这个就是默认加载器。

### 双亲委派机制

下面来看看上述的几种加载器是如何配合运作的，先看下如下的图

![双亲委派模型](/images/java/classLoader.png)

上图为Java推荐的一种类加载实现模型，即双亲委派模型；虽然看着是有着亲子关系，但
并不是依靠继承而是靠组合来维护彼此的关系；它的工作过程如下：

<当一个类加载器收到类加载的请求后，它首先不会主动的加载这个类，而是把这个请求委派
给父加载器去做，每一个层级的加载器都是如此，因此最终会传递到顶层的BootstrapClassLoader，
只有上一层的加载器无法加载类时，才会自己主动加载。

### 重定义类加载器

了解了双亲委派机制后，发现开始自定义的加载器并不满足这种策略，因此需要重写；

正确的步骤应该是:

* 查询类是否已经被加载，findLoadedClass(String)
* 如果父加载器存在，委托给父加载器去加载
* 如果父加载器不存在，在查询使用BootstrapClass去加载
* 如果此时仍然没有加载成功，则findClass(String)

翻开ClassLoader类中定义的方法，发现loadClass方法本身已经定义好了双亲委派策略，自定义
加载器需要做的仅仅是重定义findClass就可以了(推荐方法)。

`loadClass的原生定义`

```
protected Class<?> loadClass(String name, boolean resolve)
  throws ClassNotFoundException
{
  synchronized (getClassLoadingLock(name)) {
    // First, check if the class has already been loaded
    Class<?> c = findLoadedClass(name);
    if (c == null) {
      long t0 = System.nanoTime();
      try {
        if (parent != null) {
          c = parent.loadClass(name, false);

        } else {
          c = findBootstrapClassOrNull(name);

        }

      } catch (ClassNotFoundException e) {
        // ClassNotFoundException thrown if class not found
        // from the non-null parent class loader

      }

      if (c == null) {
        // If still not found, then invoke findClass in order
        // to find the class.
        long t1 = System.nanoTime();
        c = findClass(name);

        // this is the defining class loader; record the stats
        sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
        sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
        sun.misc.PerfCounter.getFindClasses().increment();

      }

    }
    if (resolve) {
      resolveClass(c);

    }
    return c;

  }

}

```

## 总结

作为一个初学者，系统的学习了Java中的类加载机制，对虚拟机中Class对象的产生有了清晰的认识。

