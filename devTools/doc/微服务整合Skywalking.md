# 微服务整合Skywalking

### 1 agent下载

Skywalking通过一系列的探针来采集数据发送给OAP服务，Java程序接入Skywalking需要借助`Java Agent`（Java探针）。  
下载地址：https://www.apache.org/dyn/closer.cgi/skywalking/java-agent/8.12.0/apache-skywalking-java-agent-8.12.0.tgz  

下载之后，解压，放在任意位置。目录结构说明：
- `config`：里面存在一个`agent.config`文件，是`SkyWalking Agent`的唯一配置文件。
- `activations`：当前`SkyWalking Agent`正在使用的功能组件。
- `bootstrap-plugins`：jdk插件，主要提供了Http和多线程相关的功能支持。会在启动时被加载。
- `plugins`：包含了`SkyWalking Agent`的提供的全部插件。会在启动时被加载。
- `optional-plugins`：存储了一些可选的插件（这些插件可能会影响整个系统的性能或是有版权问题），如果需要使用这些插件，需将相应`jar`包移动到`plugins`目录下。
- `optional-reporter-plugins`：可选插件，提供数据报告相关功能。如果需要使用这些插件，需将相应`jar`包移动到`plugins`目录下。

* * *

### 2 agent应用

##### 2.1 修改agent配置

我们需要用到`agent`，此时需要将`agent.config`配置文件拷贝到每个需要集成Skywalking工程的resource目录下，然后修改部分配置。

> `agent.config`是一个 KV 结构的配置文件，类似于`properties`文件，value 部分使用 `${}` 包裹，其中使用冒号分为两部分，
> 前半部分是可以覆盖该配置项的系统环境变量名称，后半部分为默认值。  
> 例如这里的`agent.service_name`配置项，如果系统环境变量中指定了`SW_AGENT_NAME`值（注意，全是大写），则优先使用环境变量中指定的值，
> 如果环境变量未指定，则使用`Your_ApplicationName`这个默认值。

> 直接把配置修改好后放到项目的resource目录下(或者其他路径)是最不容易才出错的一种方式，同时我们可以采用其他方式覆盖默认值：

- JVM配置覆盖

例如这里的`agent.service_name`配置项，如果在JVM启动之前，明确指定了下面的JVM配置：

```shell
# "skywalking."是 Skywalking环境变量的默认前缀
-Dskywalking.agent.service_name=order-service
```

- 探针配置覆盖

将Java Agent配置为如下：

```shell
# 默认格式是 -javaagent:agent.jar=[option1]=[value1],[option2]=[value2]
-javaagent:D:\Code\skywalking-java-agent\skywalking-agent.jar=agent.service_name=order-service
```

> 以上方式都能覆盖`agent.config`配置文件中，`agent.service_name`的默认值。但是这些配置都有不同优先级，优先级如下：  
探针配置 > JVM配置 > 系统环境变量配置 > agent.config文件默认值

> 更多配置说明，参考官方文档：https://skywalking.apache.org/docs/skywalking-java/next/en/setup/service-agent/java-agent/configurations/

##### 2.2 IDEA集成使用agent

1、修改`agent.confg`配置文件（可选步骤，这些配置可以通过步骤2的方式去指定，根据优先级原则会覆盖`agent.conf`默认值）

```shell
# 指定oap服务的地址
collector.backend_service=${SW_AGENT_COLLECTOR_BACKEND_SERVICES:127.0.0.1:11800}
# 指定服务名称
agent.service_name=${SW_AGENT_NAME:xxx-service}
# 指定其他相关配置...
```

2、在IDEA中，为每个微服务项目分别配置VM参数：

```shell
# 配置agent
-javaagent:D:\Code\skywalking-java-agent\skywalking-agent.jar
# 指定agent加载的配置文件路径（可以不指定，默认使用agent包下config目录中的配置文件）
-Dskywalking_config=D:\Code\scp\scp-register\src\main\resources\agent.config
# 指定oap服务的地址
-Dskywalking.collector.backend_service=127.0.0.1:11800
# 指定服务名称
-Dskywalking.agent.service_name=scp-register
# 指定实例名称（可以不指定，默认实例名称是由`UUID@hostname`组成，UUID是32位。这样会导致每次服务重启都会生成一个新的实例名称）
-Dskywalking.agent.instance_name=henry-win
# 追踪sql参数（可选参数，如果应用程序存在sql数据源配置，且需要记录sql执行参数，则可以设置该参数为true）
-Dskywalking.plugin.jdbc.trace_sql_parameters=true
# 设置sql参数长度（可选参数，配和trace_sql_parameters使用。如果设置为正数，db.sql.parameters则会被截断到这个长度，否则会被完全保存，这可能会导致性能问题。）
-Dskywalking.plugin.jdbc.sql_parameters_max_length=512
# 设置采样率（可选参数，表示在3秒内对N个TraceSegment进行采样。-1表示全部采样，改为大于0的数即可。默认情况下，负数或零表示关闭采样）
-Dskywalking.agent.sample_n_per_3_secs=-1
# 指定其他相关配置...
```

_![图片](../_media/Snipaste_2022-10-06_17-15-50.png ':size=60%')

此时启动每个微服务，并访问：http://127.0.0.1:8080 效果如下：

_![图片](../_media/Snipaste_2022-10-06_18-13-30.png ':size=100%')

如果你要追踪Gateway的话，你会发现：无法通过Gateway发现路由的服务链路？

原因：`Spring Cloud Gateway`是基于`WebFlux`实现，必须搭配上`apm-spring-cloud-gateway-2.1.x-plugin`和`apm-spring-webflux-5.x-plugin`两个插件。

解决方案：将`agent/optional-plugins`下的两个插件复制到`agent/plugins`目录下。

##### 2.2 生产环境使用agent

生产环境使用，要视我们项目的部署方式来决定，比如是直接通过`java -jar`来运行程序，还是通过docker方式来部署程序。  
不管是通过哪种方式，大致思路和本地IDEA使用是一致的，都是通过配置`javaagent`，并指定一系列的`skywalking agent`参数。
