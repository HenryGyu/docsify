# Skywalking日志上报

**1、Maven添加依赖**

```xml
<dependency>
    <groupId>org.apache.skywalking</groupId>
    <artifactId>apm-toolkit-logback-1.x</artifactId>
    <!-- 版本和agent版本保持一致 -->
    <version>8.12.0</version>
</dependency>
```

**2、配置logback-spring.xml文件**

这里采用grpc方式配置日志上传方式。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="false" scan="false">
    <!-- 省略其他配置... -->
  
    <!-- skywalking日志配置 -->
    <appender name="grpc-log" class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.log.GRPCLogClientAppender">
      <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
        <layout class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.mdc.TraceIdMDCPatternLogbackLayout">
          <Pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%X{tid}] [%thread] %-5level %logger{36} -%msg%n</Pattern>
        </layout>
      </encoder>
    </appender>

    <root level="INFO">
      <!-- 引用上面定义的appender -->
      <appender-ref ref="grpc-log"/>
    </root>
</configuration>
```

**3、在Skywalking-UI页面查看上传的日志**

![](../_media/Snipaste_2022-10-07_16-33-11.png ':size=100%')

![](../_media/Snipaste_2022-10-07_16-33-39.png ':size=100%')

**4、更多配置**

如果要实现在控制台打印`TraceId`，并使用`logback`的`AsyncAppender`，或者其他更高级的用法。
参考官方文档：https://skywalking.apache.org/docs/skywalking-java/next/en/setup/service-agent/java-agent/application-toolkit-logback-1.x/
