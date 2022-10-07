# Skywalking告警配置

Apache SkyWalking告警是由一组规则驱动，这些规则定义在OAP服务下的config/alarm-settings.yml文件中。

#### 告警规则

告警规则有两种类型，单独规则（Individual Rules）和复合规则（Composite Rules），复合规则是单独规则的组合。

* 单独规则（Individual Rules）

  单独规则主要有以下几点：
  * 规则名称：在告警信息中显示的唯一名称，必须以`_rule`结尾。
  * metrics-name：指标名称，也是OAL脚本中的度量名。默认配置中可以用于告警的指标有：服务，实例，端点，服务关系，实例关系，端点关系。它只支持`long`,`double`和`int`类型。
  * op：操作符，支持 `>`, `>=`, `<`, `<=`, `=`。
  * threshold：阈值。
  * period：评估指标的时间长度。这是一个时间窗口。
  * count：在一个周期窗口中，如果按op计算超过阈值的次数达到count，则发送告警。
  * only-as-condition：true或者false，指定规则是否可以发送告警，或者仅作为复合规则的条件。
  * silence-period：静默时间，当发生告警后，多长时间不再告警，默认和period值保持一致，这意味着相同的告警（同一个度量名称拥有相同的Id）在同一个周期内只会触发一次。
  * message：该规则触发时，发送的通知消息。
  * include-names：包含在此规则之内的实体名称列表。
  * exclude-names：排除在此规则以外的实体名称列表。
  * include-names-regex：提供一个正则表达式来包含实体名称。如果同时设置包含名称列表和包含名称的正则表达式，则两个规则都将生效。
  * exclude-names-regex：提供一个正则表达式来排除实体名称。如果同时设置排除名称列表和排除名称的正则表达式，则两个规则都将生效。
  * include-labels：包含在此规则之内的标签。
  * exclude-labels：排除在此规则以外的标签。
  * include-labels-regex：提供一个正则表达式来包含标签。如果同时设置包含标签列表和包含标签的正则表达式，则两个规则都将生效。
  * exclude-labels-regex：提供一个正则表达式来排除标签。如果同时设置排除标签列表和排除标签的正则表达式，则两个规则都将生效。

单独规则举例：

```yaml
rules:
  service_resp_time_rule: # 规则名称, 必须以`_rule`结尾.
    metrics-name: service_resp_time # 指标名称
    op: ">"
    threshold: 1000
    period: 10
    count: 3
    silence-period: 5
    message: 服务:【{name}】\n响应时间在过去10分钟的3分钟内超过1000毫秒.
  service_sla_rule:
    metrics-name: service_sla
    op: "<"
    threshold: 8000
    period: 10
    count: 2
    silence-period: 3
    message: 在过去10分钟的2分钟内，服务:【{name}】的成功率低于80%.
  service_resp_time_percentile_rule:
    metrics-name: service_percentile
    op: ">"
    threshold: 1000,1000,1000,1000,1000
    period: 10
    count: 3
    silence-period: 5
    message: 在过去10分钟的3分钟内，服务:【{name}】的百分比响应时间告警, 由于p50>1000、p75>1000、p290>1000、ps95>1000、ph99>1000的多种情况.
  service_instance_resp_time_rule:
    metrics-name: service_instance_resp_time
    op: ">"
    threshold: 1000
    period: 10
    count: 2
    silence-period: 5
    message: 服务实例:【{name}】\n响应时间在过去10分钟的2分钟内超过1000毫秒
  database_access_resp_time_rule:
    metrics-name: database_access_resp_time
    threshold: 1000
    op: ">"
    period: 10
    count: 2
    message: 数据库访问:【{name}】\n响应时间在过去10分钟的2分钟内超过1000毫秒
  endpoint_relation_resp_time_rule:
    metrics-name: endpoint_relation_resp_time
    threshold: 1000
    op: ">"
    period: 10
    count: 2
    message: 端点关系:【{name}】\n响应时间在过去10分钟的2分钟内超过1000毫秒
```

* 复合规则（Composite Rules）

  复合规则仅适用于针对相同实体级别的告警规则，例如都是服务级别的告警规则：service_percent_rule && service_resp_time_percentile_rule。  
  不可以编写不同实体级别的告警规则，例如服务级别的一个告警规则和端点级别的一个规则：service_percent_rule && endpoint_percent_rule。

  复合规则主要有以下几点：
  * 规则名称：在告警信息中显示的唯一名称，必须以`_rule`结尾。
  * expression：指定如何组成规则，支持`&&`, `||`, `()`操作符。
  * message：该规则触发时，发送的通知消息。

复合规则举例：

```yaml
rules:
  service_resp_time_rule:
    metrics-name: service_resp_time
    op: ">"
    threshold: 1000
    period: 10
    count: 2
    silence-period: 10
    message: 服务【{name}】\n平均响应时间在过去10分钟的2分钟内超过1000毫秒
  service_sla_rule:
    metrics-name: service_sla
    op: "<"
    threshold: 8000
    period: 10
    count: 2
    silence-period: 10
    message: 在过去10分钟的2分钟内，服务:【{name}】的成功率低于80%.
composite-rules:
  comp_rule:
    expression: service_resp_time_rule && service_sla_rule
    message: 在过去10分钟的2分钟内，服务【{name}】平均响应时间超过1000毫秒并且成功率低于80％
```

#### 告警WebHooks

* webhooks

  当告警触发时，被调用的服务端点列表。  
  如果您按以下方式配置了`webhooks`，则告警消息将按`Content-Type`为`application/json`通过HTTP的`POST`方式发送。

```yaml
webhooks:
  - http://127.0.0.1/notify/
  - http://127.0.0.1/go-wechat/
```

* wechatHooks

  当告警触发时，被调用的微信接口。  
  只有微信的企业版才支持`webhooks`，如何使用微信的`webhooks`可参见如何配置群机器人。  
  如果您按以下方式配置了微信的`webhooks`，则告警消息将按`Content-Type`为`application/json`通过HTTP的`POST`方式发送。

```yaml
wechatHooks:
  textTemplate: |-
    {
      "msgtype": "text",
      "text": {
        "content": "Apache SkyWalking 告警: \n %s."
      }
    }
  webhooks:
    - https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=dummy_key
```

* dingtalkHooks

  当告警触发时，被调用的钉钉接口。  
  您需要遵循自定义机器人开放并创建新的`webhooks`。为了安全起见，您可以为`webhooks`网址配置可选的密钥。  
  如果您按以下方式配置了钉钉的`webhooks`，则告警消息将按`Content-Type`为`application/json`通过HTTP的`POST`方式发送。

```yaml
dingtalkHooks:
  textTemplate: |-
    {
      "msgtype": "text",
      "text": {
        "content": "Apache SkyWalking 告警: \n %s."
      }
    }
  webhooks:
    - url: https://oapi.dingtalk.com/robot/send?access_token=dummy_token
      secret: dummysecret
```
