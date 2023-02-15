# Skywalking忽略自定义异常

在某些代码中，异常被用作控制业务流程的一种方式。Skywalking 提供了 2 种方法来容忍在span中跟踪的异常。

  1. 在agent配置中设置异常类的名称
  2. 在代码中使用`@IgnoredException`注解

假设我们自定义的业务异常为：`com.dct.scp.common.core.exception.CustomException`,  
当我们没有做任何设置的情况下，访问某个接口抛出该异常时，skywalking会将其标记为异常状态，并显示异常信息。

![](../_media/Snipaste_2022-10-07_17-13-07.png ':size=100%')

![](../_media/Snipaste_2022-10-07_17-31-23.png ':size=100%')

**1. 在agent配置中设置异常类的名称**

`statuscheck.ignored_exceptions`属性用于在`agent`配置文件中设置类名。如果在`agent`中检测到此处列出的异常，`agent`核心会将相关`span`标记为错误状态。

```shell
# 在IDEA中，或java -jar启动命令中，配置VM相关参数
# 配置agent
-javaagent:D:\Code\skywalking-java-agent\skywalking-agent.jar
# 指定oap服务的地址
-Dskywalking.collector.backend_service=127.0.0.1:11800
# ...此处省略一系列配置
# 配置忽略跟踪的异常，此时这些异常就不会被视为错误
-Dskywalking.statuscheck.ignored_exceptions=com.dct.scp.common.core.exception.CustomException
```

此时，再去看skywalking中的链路追踪，会发现这次调用没有被标记为错误了，但异常信息仍会记录

![](../_media/Snipaste_2022-10-07_17-30-32.png ':size=100%')

**2. 在代码中使用`@IgnoredException`注解**

如果异常类上带有`@IgnoredException`注解，则在跟踪时不会将异常标记为错误状态。因为注解支持继承，也会影响子类。  
这种方式实现起来也很简单，但对代码具有侵入性，这里就不做演示了。  
感兴趣可以参考官方文档：https://skywalking.apache.org/docs/skywalking-java/next/en/setup/service-agent/java-agent/how-to-tolerate-exceptions/
