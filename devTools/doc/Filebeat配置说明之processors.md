# Filebeat配置说明之processors

## 一、前言

处理器用于过滤或者增强要发送的event。如果在配置文件中了定义了多个processor，filebeat会按照定义的顺序依次执行。

```
event -> processor 1 -> event1 -> processor 2 -> event2 ...
```

我们来看下官方都给我定义了哪些默认的processor。

## 二、processor

### 1、add\_cloud\_metadata

[添加云服务器实例元数据](https://blog.csdn.net/qq_27818541/article/details/108395438)

### 2、add\_cloudfoundry\_metadata

[自动添加cloudfoundry应用程序的相关元数据](https://blog.csdn.net/qq_27818541/article/details/108395545)

### 3、add\_docker\_metadata

[自动添加docker容器相关元数据](http://blog.csdn.net/qq_27818541/article/details/108395618)

### 4、add\_fields

[添加自定义字段信息](http://blog.csdn.net/qq_27818541/article/details/108395957)

### 5、add\_host\_metadata

[自动追加host相关信息](https://blog.csdn.net/qq_27818541/article/details/108396028)

### 6、add\_id

[根据envet生成唯一的ID](http://blog.csdn.net/qq_27818541/article/details/108396139)

### 7、add\_kubernetes\_metadata

该处理器使用相关元数据对每个事件进行注释，基于[kubernetes](https://so.csdn.net/so/search?q=kubernetes&spm=1001.2101.3001.7020) pod生成事件的元数据。在启动时，它会检测到集群内的环境并缓存Kubernetes相关的元数据。只有在检测到有效配置时才会对事件进行注释。如果它不能检测到有效的Kubernetes配置，那么事件不会用Kubernetes相关的元数据进行注释。  
添加的注释有`Pod Name` 、`Pod UID`、`Namespace`、`Labels`

```
processors:
  - add_kubernetes_metadata: ~
```

### 8、add\_labels

[添加自定义标签](http://blog.csdn.net/qq_27818541/article/details/108396287)

### 9、add\_locale

自动添加时区信息，常用语国际业务中。

```
processors:
  - add_locale: ~
```

下面的意思是 使用缩写模式 ， 例如国内输出结果为`CST`

```
processors:
  - add_locale:
      format: abbreviation
```

### 10、add\_observer\_metadata

自动添加观察者机器的相关元数据

```
processors:
  - add_observer_metadata:
      cache.ttl: 5m
```

输出为：

```
{
  "observer" : {
    "hostname" : "avce",
    "type" : "heartbeat",
    "vendor" : "elastic",
    "ip" : [
      "192.168.1.251",
      "fe80::64b2:c3ff:fe5b:b974",
    ],
    "mac" : [
      "dc:c1:02:6f:1b:ed",
    ]
  }
}
```

### 11、add\_tags

添加标签，并可指定标签在哪个字段下。

例如：

```
processors:
  - add_tags:
      tags: [web, production]
      target: "environment"
```

输出为：

```
{
  "environment": ["web", "production"]
}
```

### 12、convert

将event中的某个字段转换为其他类型，例如将字符串转换为整数。

```
processors:
  - convert:
      fields:
        - {from: "src_ip", to: "source.ip", type: "ip"}
        - {from: "src_port", to: "source.port", type: "integer"}
      ignore_missing: true
      fail_on_error: false
```

### 13、copy\_fields

将event中的某个字段的值赋值给另一个字段

```
processors:
  - copy_fields:
      fields:
        - from: message
          to: event.original
      fail_on_error: false
      ignore_missing: true
```

输出：

```
{
  "message": "my-interesting-message",
  "event": {
      "original": "my-interesting-message"
  }
}
```

### 14、decode\_base64\_field

base64解码，从一个字段里获取值，解码后赋值给另一个字段。

```
processors:
  - decode_base64_field:
      field:
        from: "message"
        to: "decode"
      ignore_missing: true
      fail_on_error: false
```

### 15、decode\_cef

如果你的日式符合Common Event Format (CEF) 标准，你可以使用它进行解码：

```
processors:
  - rename:
      fields:
        - {from: "message", to: "event.original"}
  - decode_cef:
      field: event.original
```

[更多关于cef的介绍](https://www.vertica.com/docs/9.2.x/HTML/Content/Authoring/FlexTables/LoadCEFData.htm?TocPath=Using%20Flex%20Tables%7CUsing%20Flex%20Table%20Parsers%7C_____3)

### 16、decode\_csv\_fields

解析CSF格式的日志到指定字段，内容为数组。

```
processors:
  - decode_csv_fields:
     # 解析到哪里
      fields:
        message: decoded.csv
      separator: ","
      ignore_missing: false
      overwrite_keys: true
      trim_leading_space: false
      fail_on_error: true
```

### 17、decode\_json\_fields

处理对包含JSON字符串的字段进行解码，并将字符串替换为有效的JSON对象。默认是在当前字段替换。

```
processors:
  - decode_json_fields:
      fields: ["field1", "field2", ...]
     # process_array: false
      # max_depth: 1
     # overwrite_keys: false
     # add_error_key: true
```

### 18、decompress\_gzip\_field

gzip解压缩，从一个字段值中加压缩到另一个字段中 。 使用场景少。

```
processors:
  - decompress_gzip_field:
      field:
        from: "field1"
        to: "field2"
      ignore_missing: false
      fail_on_error: true
```

感兴趣的可以看下 [gzip压缩初探](https://www.cnblogs.com/wancheng7/p/9979411.html)

### 19、dissect

从某个字段里（默认message）取值，按照`tokenizer`定义的格式 拆分（切割）数据，并输出到`target_prefix` 字段里，默认是dissect

```
processors:
  - dissect:
      tokenizer: "%{key1} %{key2}"
      # field: "message"
     # target_prefix: "dissect"
```

如果是这样的log`"App01 - WebServer is up and running"` ，输出为

```
"service": {
  "name": "App01",
  "status": "WebServer is up and running"
},
```

### 20、dns

反向解析 根据id查找hostname 。每个processor有自己查询结果的缓存。该processor使用自己的域名服务器，不会查找`/etc/hosts` ，默认使用`/etc/resolv.conf`里面的域名服务器。

```
processors:
  - dns:
      type: reverse
      fields:
        source.ip: source.hostname
        destination.ip: destination.hostname
```

`fields`字段里`source.ip`是指ip所在的字段，`source.hostname`是指新值存入的字段。

### 21、drop\_event

丢弃该event，一般要设置特定条件 。

例如 http响应码为200 就不上报event：

```
processors:
  - drop_event:
      when:
        equals:
          http.code: 200
```

### 22、drop\_fields

丢弃（过滤掉）特定字段， 一般防止敏感信息上报，例如ip 、mac地址…

```
processors:
  - drop_fields:
      fields: ["cpu.user", "cpu.system"]
```

### 23、extract\_array

[从数组中取值并填充（映射到）指定字段](http://blog.csdn.net/qq_27818541/article/details/108454142)

### 24、fingerprint

根据事件指定字段字段值生成事件的指纹。

```
processors:
  - fingerprint:
      fields: ["field1", "field2", ...]
```

输出结果：  
![](../_media/2020090820205773.png ':size=80%')

### 25、include\_fields

输出的event里 会包含哪些字段值。`@timestamp`和`type`字段会始终出现在event中，即使未在include\_fields列表中定义。

```
processors:
  - include_fields:
      when:
        condition
      fields: ["field1", "field2", ...]
```

### 26、registered\_domain

[域名注册，识别一个长串域名，转化成一个简短域名](http://blog.csdn.net/qq_27818541/article/details/108476864) 。

### 27、rename

[重命名字段](http://blog.csdn.net/qq_27818541/article/details/108476933)

### 28、script

[使用`Javascript` 处理event ，可以更加灵活的预处理业务](http://blog.csdn.net/qq_27818541/article/details/108500499)

### 29、timestamp

[从字段解析时间戳并将解析结果写入`@timestamp`（日志采集时间）字段](http://blog.csdn.net/qq_27818541/article/details/108501113)

### 30、translate\_sid

将Windows安全标识符（SID）转换为帐户名，使用仅限`Windows`系统。

```
processors:
  - translate_sid:
      field: winlog.event_data.MemberSid
      account_name_target: user.name
      domain_target: user.domain
      ignore_missing: true
      ignore_failure: true
```

### 31、translate\_sid

按照指定大小截取字段值

```
processors:
  - truncate_fields:
    fields:
    - message
    max_characters: 5
    #max_bytes: 128
    fail_on_error: false
    ignore_missing: true
```

### 32、urldecode

url 解码

```
processors:
  - urldecode:
      fields:
        - from: "field1"
          to: "field2"
      #  - from: "fieldx"
       #  to: "fieldy"
      ignore_missing: false
      fail_on_error: true

```

---

本文转自 [https://blog.csdn.net/qq_27818541/article/details/108229185#5add_host_metadata_80](https://blog.csdn.net/qq_27818541/article/details/108229185#5add_host_metadata_80)，如有侵权，请联系删除。
