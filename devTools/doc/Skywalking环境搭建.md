# Skywalking环境搭建

1. [Docker运行skywalking-oap-server](#Docker运行skywalking-oap-server)
2. [Docker运行skywalking-ui](#Docker运行skywalking-ui)
3. [Windows快速运行skywalking](#Windows快速运行skywalking)

### [](#Docker运行skywalking-oap-server "Docker运行skywalking-oap-server")Docker运行skywalking-oap-server

后台运行一个OAP服务容器，使用如下脚本：

```shell
docker run --name skywalking-oap \
-v ${cur_dir}/conf/config:/skywalking/config \
-v ${cur_dir}/conf/ext-libs:/skywalking/ext-libs \
--net=host \
-e TZ=Asia/Shanghai \
-e SW_STORAGE=mysql \
-e SW_JDBC_URL="jdbc:mysql://172.17.0.1:3306/skywalking?rewriteBatchedStatements=true" \
-e SW_DATA_SOURCE_USER=root \
-e SW_DATA_SOURCE_PASSWORD=123456 \
-e SW_CLUSTER=nacos \
-e SW_CLUSTER_NACOS_HOST_PORT=172.19.0.1:8842 \
-e SW_CLUSTER_NACOS_NAMESPACE=public \
-e SW_CLUSTER_NACOS_USERNAME=nacos \
-e SW_CLUSTER_NACOS_PASSWORD=nacos \
-e SW_CONFIGURATION=nacos \
-e SW_CONFIG_NACOS_SERVER_ADDR=172.19.0.1 \
-e SW_CONFIG_NACOS_SERVER_PORT=8842 \
-e SW_CONFIG_NACOS_SERVER_GROUP=skywalking \
-e SW_CONFIG_NACOS_SERVER_NAMESPACE=public \
-e SW_CONFIG_NACOS_PERIOD=60 \
-e SW_CONFIG_NACOS_USERNAME=nacos \
-e SW_CONFIG_NACOS_PASSWORD=nacos \
-e SW_CORE_REST_HOST=10.0.12.3 \
-e SW_CORE_REST_PORT=12801 \
-e SW_CORE_GRPC_HOST=10.0.12.3 \
-e SW_CORE_GRPC_PORT=11801 \
-d apache/skywalking-oap-server:9.2.0
```

> **挂载配置文件目录：`-v ${cur_dir}/conf/config:/skywalking/config`**
>   
> `${cur_dir}`是我系统中指向当前目录的一个变量，需替换成你真实的Linux路径  
> 配置文件需从容器中拷贝出来，再挂载，具体操作如下：  
> 1、快速运行临时容器：`docker run --name oap -d apache/skywalking-oap-server:9.2.0`  
> 2、拷贝配置文件目录到当前目录下：`docker cp oap:/skywalking/config ./conf/config`  
> 3、停止临时容器：`docker stop oap`  
> 4、删除临时容器：`docker rm oap`  
> 
> * * *
>
> **挂载扩展jar包目录：`-v ${cur_dir}/conf/ext-libs:/skywalking/ext-libs`**
> 
> 例如，当Skywalking采用Mysql作为存储层的时候，需要将mysql的连接驱动放在改目录下，否则启动会报错。
>
> * * *
>
> **环境变量说明**
>
> SW_STORAGE: 指定指标数据存储方式（mysql/elasticsearch/...），不指定则会使用内置H2数据源  
> SW_CLUSTER: 指定OAP服务集群方式，默认值为standalone，即单机模式  
> SW_CONFIGURATION: 指定动态配置存储方式，默认值为none，即无配置中心，采用本地配置，也就是上面挂载的配置文件目录  
> SW_CORE_REST_HOST: RESTful服务的绑定IP。默认为：0.0.0.0  
> SW_CORE_REST_PORT: RESTful服务的绑定端口。默认为：12800  
> SW_CORE_GRPC_HOST: gRPC服务的绑定IP。默认为：0.0.0.0  
> SW_CORE_GRPC_PORT: gRPC服务的绑定端口。默认为：11800  
> `elasticsearch作为存储层，需指定的配置：`  
> SW_STORAGE_ES_CLUSTER_NODES: 指定ES集群节点，多个逗号分隔  
> SW_ES_USER: 指定ES集群用户名  
> SW_ES_PASSWORD: 指定ES集群密码  
> `mysql作为存储层，需指定的配置：`  
> SW_JDBC_URL: 指定mysql地址  
> SW_DATA_SOURCE_USER: 指定mysql用户名  
> SW_DATA_SOURCE_PASSWORD: 指定mysql密码  
> `nacos作为集群注册中心，需指定的配置：`  
> SW_CLUSTER_NACOS_HOST_PORT: 指定nacos集群地址  
> SW_CLUSTER_NACOS_NAMESPACE: 指定nacos命名空间  
> SW_CLUSTER_NACOS_USERNAME: 指定nacos集群用户名  
> SW_CLUSTER_NACOS_PASSWORD: 指定nacos集群密码  
> `nacos作为动态配置中心，需指定的配置：`  
> SW_CONFIG_NACOS_SERVER_ADDR: 指定nacos配置中心IP  
> SW_CONFIG_NACOS_SERVER_PORT: 指定nacos配置中心端口  
> SW_CONFIG_NACOS_SERVER_GROUP: 指定nacos配置分组  
> SW_CONFIG_NACOS_SERVER_NAMESPACE: 指定nacos命名空间  
> SW_CONFIG_NACOS_PERIOD: 指定nacos数据同步的周期（以秒为单位）  
> SW_CONFIG_NACOS_USERNAME: 指定nacos用户名  
> SW_CONFIG_NACOS_PASSWORD: 指定nacos密码  
> 
> 更多环境变量配置说明，参考官方文档：https://skywalking.apache.org/docs/main/v9.2.0/en/setup/backend/configuration-vocabulary/
>
> * * *
>
> **查看OAP服务启动日志：`docker logs -f --tail 500 skywalking-oap`**

* * *

### [](#Docker运行skywalking-ui "Docker运行skywalking-ui")Docker运行skywalking-ui

后台运行一个skywalking-ui容器，使用如下脚本：

```shell
docker run --name skywalking-ui \
-p 10800:8080 \
-e TZ=Asia/Shanghai \
-e SW_OAP_ADDRESS=http://10.0.12.3:12801 \
-d apache/skywalking-ui:9.2.0
```

> `SW_OAP_ADDRESS`: 指定OAP服务地址，多个逗号分隔（注意OAP服务绑定了内网IP，所以这里也使用内网IP）  
> 这里指定宿主机的10800端口，映射到容器的8080端口。启动成功后，访问`http://10.0.12.3:10800`，正常显示skywalking控制台即可。

### [](#Windows快速运行skywalking "Windows快速运行skywalking")Windows快速运行skywalking

如果你没有Docker环境，想在本地windows快速运行skywalking，需先下载skywalking的apm包。  
下载地址：https://www.apache.org/dyn/closer.cgi/skywalking/9.2.0/apache-skywalking-apm-9.2.0.tar.gz

下载之后，解压。进入`bin`目录下，会看到对应的启动脚本，windows的脚本后缀都是`.bat`，脚本如下：
- `oapService.bat`: 启动skywalking-oap-server
- `oapServiceInit.bat`: 启动skywalking-oap-server并执行初始化
- `oapServiceNoInit.bat`: 启动skywalking-oap-server，不执行初始化。此时需要依靠其他oap服务优先执行初始化
- `startup.bat`: 启动skywalking-oap-server + skywalking-ui
- `webappService.bat`: 启动skywalking-ui

一般情况下，只需双击`startup.bat`，即可运行一个OAP服务和UI服务。默认访问地址：http://127.0.0.1:8080

> 如果需要像上面Docker启动一样，指定一系列的环境变量，则只需修改`config`目录下的`application.yml`文件即可，将里面的环境变量修改成你想要设置的内容。
