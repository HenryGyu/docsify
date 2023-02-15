# Skywalking-UI和指标

概念
-----------------------------------------

*   Service 服务，具体的服务，比如用户服务
*   Instance 实例，服务的实例，比如xxx节点上的服务，id@ip
*   Endpoint 端点，服务对外提供的接口，一个接口就是一个端点
*   Service Apdex 用户满意程度
  *   最大值就是 1， 是一个不断优化的方向
  *   分3个指标，T 值代表着用户对服务性能满意的响应时间阈值，假如T是0.5秒
    *   满意：响应时间少于 T 秒钟； 0.5秒内
    *   容忍：响应时间 T～4T 秒； 0.5~2秒内
    *   失望：响应时间超过 4T 秒； 多于2秒
  *   Apdex = （正常请求数 + 可容忍请求数 / 2）/ 请求总数。示例如下
  > 服务A定义T=200ms，在100个采样中，有20个请求小于200ms，有60个请求在200ms到800ms之间，有20个请求大于800ms。  
  > 计算apdex = (20 + 60/2)/100 = 0.5。
*   SLA
  *   服务等级协议，全称：service level agreement，为保障服务的性能和可用性
  *   9越多代表全年服务可用时间越长服务更可靠，停机时间越短
      <table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br></pre></td><td class="code"><pre><span class="line">1年 = 365天 = 8760小时</span><br><span class="line"></span><br><span class="line">99.9 = 8760 * 0.1% = 8760 * 0.001 = 8.76小时</span><br><span class="line">99.99 = 8760 * 0.0001 = 0.876小时 = 0.876 * 60 = 52.6分钟</span><br><span class="line">99.999 = 8760 * 0.00001 = 0.0876小时 = 0.0876 * 60 = 5.26分钟</span><br><span class="line"></span><br><span class="line">全年停机5.26分钟才能99.999%，即5个9</span><br></pre></td></tr></tbody></table>
*   CPM 全称 call per minutes，是吞吐量(Throughput)指标，每分钟请求调用的次数
*   RT Response Time 表示请求响应时间
*   Percent Response 百分位数统计
  *   表示采集样本中某些值的占比，SkyWalking 有 “p50、p75、p90、p95、p99” 一些列值， “p99:360” 表示 99% 请求的响应时间在360ms以内

SkyWalking UI（普通服务 -> 服务）
----------------------------------------------------------

*   顶部控制栏

    ![](../_media/Snipaste_2022-10-07_09-16-17.png ':size=100%')

  *   Service（服务列表）：正在通过agent收集指标的服务列表
  *   Topology（拓扑图）：以拓扑图的方式展现服务的关系
  *   Trace（追踪）：以接口的列表方式展现
  *   Log（日志）：可查看服务日志

* Service：服务列表

  *   Service Groups：服务分组
  *   Service Names：服务名称
  *   Load(calls / min): 服务每分钟请求数
  *   Success Rate(%)：请求成功率
  *   Latency(ms)：平均响应延时，单位ms
  *   Apdex：当前服务的评分
  
*   Service Overview：服务概览

    ![](../_media/Snipaste_2022-10-07_09-30-35.png ':size=100%')

  *   Service Apdex（数字）:当前服务的评分
  *   Success Rate（百分比）:请求成功率
  *   Service Load（数字）：每分钟请求数
  *   Service Avg Response Time（折线图）：每个时间段的平均响应延时，单位ms
  *   Service Apdex（折线图）:每个时间段的Apdex评分
  *   Service Response Time Percentile（折线图）:：百分比响应延时
  *   Servce Load（折线图）：每个时间段的每分钟请求数
  *   Success Rate（折线图）：每个时间段的请求成功率
  *   Message Queue Consuming Count（折线图）：消息队列消费计数（已消费的消息数量）
  *   Message Queue Avg Consuming Latency（折线图）：消息队列平均消费延时（消息接收的时间戳 - 消息投递的时间戳）
  *   Servce Instances Load（列表）：每个服务实例的每分钟请求数
  *   Slow Service Instance（列表）：慢服务实例排行榜
  *   Service Instance Success Rate（列表）：每个服务实例的请求成功率
  *   Endpoint Load in Current Service（列表）：当前服务中的端点负载（即接口每分钟请求数）
  *   Slow Endpoints in Current Service（列表）：当前服务中的端点，慢响应排行榜（即慢接口排行榜）
  *   Success Rate in Current Service（列表）：当前服务中的端点请求成功率排行榜

*   Service Instance：服务实例列表

    ![](../_media/Snipaste_2022-10-07_09-48-58.png ':size=100%')

  *   Service Instances：服务实例名称
  *   Load(calls / min): 每分钟请求数
  *   Success Rate(%)：请求成功率
  *   Latency(ms)：平均响应延时，单位ms
  *   Attributes：服务实例属性（包含操作系统，主机名，JVM参数等相关信息）

*   Service Instance Overview：服务实例概览

    ![](../_media/Snipaste_2022-10-07_09-56-46.png ':size=100%')

  *   Service instance load（折线图）：当前服务实例，每个时间段的每分钟请求数
  *   Service Instance Success Rate（折线图）：当前服务实例，每个时间段的请求成功率
  *   Service Instance Latency（折线图）：当前服务实例，每个时间段的响应延时
  *   Database Connection Pool（折线图）：当前服务实例，每个时间段的数据库连接池数量（点击切换，可查看active数量，commit数量，idle数量等相关信息）
  *   Thread Pool（折线图）：当前服务实例，每个时间段的线程池数量

*   Service Instance Trace：服务实例追踪

    ![](../_media/Snipaste_2022-10-07_10-39-03.png ':size=100%')

    ![](../_media/Snipaste_2022-10-07_10-41-43.png ':size=100%')

    ![](../_media/Snipaste_2022-10-07_10-44-19.png ':size=100%')

*   Service Instance JVM：服务实例JVM信息

    ![](../_media/Snipaste_2022-10-07_10-23-20.png ':size=100%')

  *   JVM CPU（折线图）：jvm占用CPU的百分比
  *   JVM Memory（折线图）：JVM内存占用大小，单位mb（点击切换，可查看heap内存，noheap内存等信息）
  *   JVM GC Time（折线图）：JVM垃圾回收时间（点击切换，可查看YGC, OGC, NGC）
  *   JVM GC Count（折线图）：JVM垃圾回收次数（点击切换，可查看YGC, OGC, NGC）
  *   JVM Thread Count（折线图）：JVM线程数（点击切换，可查看live, daemon, peak）
  *   JVM Thread State Count（统计图）：JVM线程状态计数（包含timed_waiting, blocked, waiting, runnable）
  *   JVM Class Count（统计图）：JVM类计数

* Service Endpoint：服务端点列表

  ![](../_media/Snipaste_2022-10-07_10-51-59.png ':size=100%')

  *   Endpoints：端点名称
  *   Load(calls / min): 每分钟请求数
  *   Success Rate(%)：请求成功率
  *   Latency(ms)：平均响应延时，单位ms

* Service Endpoint Overview：服务端点概览

  ![](../_media/Snipaste_2022-10-07_10-56-27.png ':size=100%')

  *   Endpoint Load（折线图）：端点的每分钟请求数
  *   Endpoint Avg Response Time（折线图）：端点每个时间段的平均请求响应时间
  *   Endpoint Response Time Percentile（折线图）：端点每个时间段的响应时间占比
  *   Endpoint Success Rate（折线图）：端点每个时间段的请求成功率
  *   Message Queue Consuming Count（折线图）：消息队列消费计数（已消费的消息数量）
  *   Message Queue Avg Consuming Latency（折线图）：消息队列平均消费延时（消息接收的时间戳 - 消息投递的时间戳）

* Service Endpoint Trace：服务端点追踪

  记录了端点的所有请求记录，具体信息和上面将的 Service Instance Trace：服务实例追踪。一样，此处不再阐述。
  ![](../_media/Snipaste_2022-10-07_11-16-30.png ':size=100%')

* Topology：拓扑图

  记录了服务间的关系和连接情况，红色方块代表不正常的服务，可以从这里点击某个服务，进去对应的Dashboard，查看具体的信息
  ![](../_media/Snipaste_2022-10-07_11-24-48.png ':size=100%')

* 其他相关信息

  ![](../_media/Snipaste_2022-10-07_11-31-27.png ':size=100%')

SkyWalking UI（普通服务 -> 虚拟数据库）
----------------------------------------------------------

*   Virtual Database：虚拟数据库列表

    ![](../_media/Snipaste_2022-10-07_11-34-48.png ':size=100%')

  *   SkyWalking会通过agent推测我们程序连接的数据库信息，并进行监控
  *   Service Names：数据库连接名
  *   Latency(ms)：平均访问延时，单位ms
  *   Success Rate(%)：请求成功率
  *   Traffic(calls / min): 每分钟请求数

*   Virtual-Database-Service：虚拟数据库详情

    ![](../_media/Snipaste_2022-10-07_11-39-52.png ':size=100%')

  *   Database Avg Response Time（折线图）：数据库每个时间段的平均请求响应时间
  *   Database Access Successful Rate（折线图）：数据库每个时间段的请求成功率
  *   Database Traffic（折线图）：数据库每个时间段的负载
  *   Database Access Latency Percentile（折线图）：数据库每个时间段的访问延时占比
  *   Slow Statements（列表）：慢SQL排行榜


