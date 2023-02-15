# Skywalking动态配置

SkyWalking 支持以下动态配置

| **Config Key** | **Value 描述**                                                                 | **Value 格式示例**                                                                                                                            |
| --- |------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------|
| agent-analyzer.default.slowDBAccessThreshold | 数据库慢语句阀值, 覆盖`applciation.yml`中的`receiver-trace/default/slowDBAccessThreshold` | default:200,mongodb:50 |
| agent-analyzer.default.uninstrumentedGateways | 配置网关，覆盖`gateways.yml`文件 | 参考 [gateways.yml](https://skywalking.apache.org/docs/main/next/en/setup/backend/uninstrumented-gateways/#configuration-format) |
| alarm.default.alarm-settings | 配置告警规则, 覆盖`alarm-settings.yml`文件 | 参考 [alarm-settings.yml](https://skywalking.apache.org/docs/main/next/en/setup/backend/backend-alarm/) |
| core.default.apdexThreshold | `apdex`阈值配置, 覆盖`service-apdex-threshold.yml`文件 | 参考 [service-apdex-threshold.yml](https://skywalking.apache.org/docs/main/next/en/setup/backend/apdex-threshold/) |
| core.default.endpoint-name-grouping | 端点名称分组配置, 覆盖`endpoint-name-grouping.yml`文件 | 参考 [endpoint-name-grouping.yml](https://skywalking.apache.org/docs/main/next/en/setup/backend/endpoint-grouping-rules/) |
| core.default.log4j-xml | log4j日志配置。覆盖`log4j2.xml`文件 | 参考 [log4j2.xml](https://skywalking.apache.org/docs/main/next/en/setup/backend/dynamical-logging/) |
| agent-analyzer.default.traceSamplingPolicy | 服务器端的跟踪采样配置。覆盖`trace-sampling-policy-settings.yml`文件 | 参考 [trace-sampling-policy-settings.yml](https://skywalking.apache.org/docs/main/next/en/setup/backend/trace-sampling/) |                                                                       |                                                                                                                                           |
| configuration-discovery.default.agentConfigurations | | 参考 [configuration-discovery.md](https://github.com/apache/skywalking-java/blob/20fb8c81b3da76ba6628d34c12d23d3d45c973ef/docs/en/setup/service-agent/java-agent/configuration-discovery.md) |

通过debug源码发现，配置模块会启动一个定时任务，默认60s向配置中心拉取一次配置  
`org.apache.skywalking.oap.server.configuration.api.ConfigWatcherRegister#start`  
![](../_media/04639efa538a40e4b3a615217869479a.png ':size=100%')  
默认未覆盖动态配置时，对应 `config key` 的值为空  
`org.apache.skywalking.oap.server.configuration.nacos.NacosConfigWatcherRegister#readConfig`  
![](../_media/8101be4b5b1440518eb832c7f04b3e61.png ':size=100%')

##### 覆盖数据库慢语句阀值

![](../_media/Snipaste_2022-10-07_15-02-56.png ':size=100%')  
再次进行源码 debug 可以发现，对应 `config key` 已经有值了  
![](../_media/f5273cfaba3e497e8ba7b59122519d41.png ':size=100%')
