---
layout: post
title: scala play web framework
description: 通过一个简单play framework来刨析web框架的各个必要元素
category: blog
---

play framework是scala生态圈内部一个简单的、较完善的Web开发框架, 通过对它的学习，
我们希望能够管中窥豹,以解在Web业务开发时的疑惑。

## Play Plugin Or Module

## Injector

## adhoc application

自主app，不用sbt、activator、maven等管理的独立自主的application，需要由开发人员
手动控制配置文件、插件等文件的整合

### Configuration

file ---> Config ---> Configuration

### Router
### ServerConfig

server服务器的配置，配置参数有：

* @param rootDir The root directory of the server. Used to find default locations of
* files, log directories, etc.
* @param port The HTTP port to use.
* @param sslPort The HTTPS port to use.
* @param address The socket address to bind to.
* @param mode The run mode: dev, test or prod.
* @param configuration: The configuration to use for loading the server. This is not
* the same as application configuration. This configuration is usually loaded from a
* server.conf file, whereas the application configuration is usually loaded from an
* application.conf file.

### Environment

app文件系统、rootPath、应用启动环境测试/开发/生产
这个rootPath代表什么

```
@Singleton
public class Environment {
  private final play.api.Environment env;

  @Inject
    public Environment(play.api.Environment environment) {
      this.env = environment;
    }

  public Environment(File rootPath, ClassLoader classLoader, Mode mode) {
    this(new play.api.Environment(rootPath, classLoader, play.api.Mode.apply(mode.ordinal())));

  }

  public Environment(File rootPath, Mode mode) {
    this(rootPath, Environment.class.getClassLoader(), mode);

  }

  public Environment(File rootPath) {
    this(rootPath, Environment.class.getClassLoader(), Mode.TEST);

  }

  public Environment(Mode mode) {
    this(new File("."), Environment.class.getClassLoader(), mode);
  }

}
```

### NettyServer VS DefaultApplication

1. 如何修改current application root path

在ServerConfig中可以定义

2. router如何从配置文件中读取

3. 如何仅仅通过读取application.conf就启动一个play实例

4. play app的启动支持默认的配置、命令行配置、指定配置文件启动方式

对各种插件的引入应该放入application.conf中

```

/**
 * Helper to provide the Play built in components.
 */
trait BuiltInComponents {
  def environment: Environment
    def sourceMapper: Option[SourceMapper]
    def webCommands: WebCommands
    def configuration: Configuration

    def router: Router

    lazy val injector: Injector = new SimpleInjector(NewInstanceInjector) + router + crypto + httpConfiguration

    lazy val httpConfiguration: HttpConfiguration = HttpConfiguration.fromConfiguration(configuration)
    lazy val httpRequestHandler: HttpRequestHandler = new DefaultHttpRequestHandler(router, httpErrorHandler, httpConfiguration)
    lazy val httpErrorHandler: HttpErrorHandler = new DefaultHttpErrorHandler(environment, configuration, sourceMapper,
        Some(router))

    lazy val applicationLifecycle: DefaultApplicationLifecycle = new DefaultApplicationLifecycle
    lazy val application: Application = new DefaultApplication(environment, applicationLifecycle, injector,
        configuration, httpRequestHandler, httpErrorHandler, Plugins.empty)

    lazy val cryptoConfig: CryptoConfig = new CryptoConfigParser(environment, configuration).get
    lazy val crypto: Crypto = new Crypto(cryptoConfig)

}
```

router = Router.load

```
/**
 * Try to load the configured router class.
 *
 * @return The router class if configured or if a default one in the root package was detected.
 */
def load(env: Environment, configuration: Configuration): Option[Class[_ <: Router]] = {
  val className = configuration.getString("application.router")

    try {
      Some(Reflect.getClass[Router](className.getOrElse("Routes"), env.classLoader))

    } catch {
      case e: ClassNotFoundException =>
              // Only throw an exception if a router was explicitly configured, but not found.
              // Otherwise, it just means this application has no router, and that's ok.
              className.map { routerName =>
                throw configuration.reportError("application.router", "Router not found: " + routerName)
              }

    }

}
```

只需要加载正确configuration就可以了, configuration的加载参考以下:
  lazy val configuration: Configuration = Configuration(ConfigFactory.load())



DefaultApplication
GuiceApplicationBuilder

NettyServer.fromApplication时，router是怎么获取的

Configuration保存的是系统级别的配置

play框架是无状态的，才能够进行代码的热加载

a route Maps是映射url到指定的action
a reverse Routing是映射一个action到url, 例如routes.Users.showUser("joe")是/users/joe

Netty的pipeline、channel的概念？

Netty的工作模式

adhoc application为什么不能指定路由配置文件

最终需要搞清楚的是app在启动的时候是怎么加载路由、配置文件的，还有每个action在渲染时是怎么找到view的

无状态框架指的是什么？session信息保存在cookie中?

play如何做到异步执行http响应的?
难倒异步执行逻辑中包含socket连接符，生成数据后直接写到socket中就可以了?

