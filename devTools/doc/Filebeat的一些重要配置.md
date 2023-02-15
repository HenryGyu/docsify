# Filebeat的一些重要配置

最近和一些客户交流，发现他们在使用filebeat进行文件采集的时候，主要的场景并不是以行为单位进行采集，而是以文件为单位进行采集。比如，一些实验数据是以文件的形式生成的，即filebeat的监控目录中会在实验结束后，添加数个实验结果的文件，这些文件有以下特点:

-   文件内容很大，从十万行到千万行级别不等
-   文件是一次性的变动，即直接移动到监控目录当中，之后不会再有改动
-   文件可能会有重命名或者删除操作

因为filebeat的默认监控对象是日志型文件，即数据会持续以行为单位输出到文件当中。因此filebeat的默认设置都是按照持续扫描，监控rotate的方式来完成[数据采集](https://cloud.tencent.com/solution/data-collect-and-label-service?from=10680)的。因而，针对以上提到的结果型文件的**一次性**特点，我们需要对filebeat的配置进行一些特定的修改。

## 如何提高文件采集效率

对于结果型文件，大多数时候，这些文件都是很大的，动辄几十M，多辄几百M，文件由十万行到千万行级别不等。

举个例子，有一个172247行的文件，文件大小在11M左右。使用filebeat的默认配置，我们会发现这个文件的采集大概需要花费5~10分钟。如果一个文件有上千万行，那么这个文件的采集可以达到1000分钟，这对我们来说是不可接受的。

这里，是什么限制了filebeat的采集速率？

### bulk\_max\_size

```
  # The maximum number of events to bulk in a single Elasticsearch bulk API index request.
  # The default is 50.
  bulk_max_size: 50
```

这个是`output.elasticsearch`的属性，控制发送给Elasticsearch的bulk API中，每批数据能包含多少条event，默认情况下，我们是每行数据一个document(或者说是event)，因此，每次filebeat默认只会发送50行数据，因此，当我们添加进来的数据由几十万行的时候，可以简单推算，我们需要推送多少次bulk request才能完成这个文件的数据录入

因此，我们可以综合考虑并发的环境来修改此参数。比如，我们所有的数据集中在几台机器上，只有几个filebeat实例在负责数据的录入时，我们可以把这个数据适当调大到500~1000的级别。需根据ES的吞吐，可以参考我们的[benchmark](https://elasticsearch-benchmarks.elastic.co/index.html#):

![](../_media/Snipaste_2023-02-15_19-11-52.png ':size=80%')

在这里插入图片描述

如何读懂这些指标可以参考我的另一篇博文：[如何解读Elasticsearch benchmark上的各种指标](https://lex-lee.blog.csdn.net/article/details/107926873)

可以看到，使用SSD的情况下，3节点可以达到10万+的吞吐。比如说，我们现在只有3个filebeat进行专门的文件录入（通常这种情况下，存储文件的[服务器](https://cloud.tencent.com/product/cvm?from=10680)并没有其他的服务需要运行），我们甚至可以让filebeat的在一个bulk request发送数千条event，当然，这个需要结合单条event的size，比如，一条event只有几十个byte，那么一次bulk request即便包含1000条event也只有几十K的大小，即我们可以再调大这个参数。

### worker

```
  # Number of workers per Elasticsearch host.
  worker: 1
```

这个也是`output.elasticsearch`的属性，我们可以指定filebeat使用多高的并发来往Elastic发送数据，我们也可以适当的增加这个值，比如我们的ES集群有3个data节点 `hosts: ["10.0.07:9200","10.0.08:9200","10.0.09:9200"]`，我们可以把这个worker设为 `3`。

结合上一个设置`bulk_max_size: 1000`，则我们可以达到更高的吞吐

### harvester\_buffer\_size

```
  # Defines the buffer size every harvester uses when fetching the file
  harvester_buffer_size: 16384
```

这个是`Log input`的属性，这个属性限定了单个文件采集器`harvester`每次读取文件的大小，默认的大小是16K。如果我们要增加某些文件的读取吞吐，可以调整这个值的大小。可以通过定义多个input，每个input单独指定的方式来确定不同文件的吞吐大小，比如下面的配置，两个文件的吞吐就不一样:

```
filebeat.inputs:
- type: log
  paths:
  - '/Users/lex.li/Downloads/2020_04_11/平台指标/db_oracle_11g.csv'
  exclude_lines: ['^"?itemid"?,"?name"?,"?bomc_id"?,"?timestamp"?,"?value"?,"?cmdb_id"?']
  harvester_buffer_size: 1638400
- type: log
  paths:
  - '/Users/lex.li/Downloads/2020_04_11/平台指标/dcos_docker.csv'
```

## 如何重新采集文件

在某些情景下，用户希望对同一个文件进行反复的采集。这时，就需要我们有能力告诉filebeat该采集哪些文件。

我们通过log input中的几个参数去定义这个能力，比如：

```
filebeat.inputs:
- type: log
  paths:
  - '/Users/lex.li/Downloads/2020_04_20/平台指标/db_oracle_11g.csv'
  exclude_lines: ['^"?itemid"?,"?name"?,"?bomc_id"?,"?timestamp"?,"?value"?,"?cmdb_id"?']
  fields:
    index_name: "db_oracle_11g"
  harvester_buffer_size: 1638400
- type: log
  paths:
  - '/Users/lex.li/Downloads/2020_04_20/业务指标/esb.csv'
  fields:
    index_name: "esb"
- type: log
  paths:
  - '/Users/lex.li/Downloads/2020_04_20/调用链指标/trace_osb.csv'
  fields:
    index_name: "trace_osb"
    
output.elasticsearch:
  hosts: ["http://localhost:9200"]
  username: "elastic"
  password: "changeme"
  index: "%{[fields.index_name]}"
  ssl.verification_mode: "none"
  bulk_max_size: 1500
  worker: 3

setup:
  template.enabled: false
  ilm.enabled: false
```

这里，我们通过 input中的`fields`和output中的`%{[ ]}`

```
filebeat.inputs:
- type: log
...
  fields:
    index_name: "db_oracle_11g"
...
output.elasticsearch:
...
  index: "%{[fields.index_name]}"
```

将不同的文件内容映射到不同的索引当中。

### registry

然后，filebeat通过registry文件来进行被监控文件的管理，在registry目录下，（比如，在我的mac上是安装目录下的`data->registry->filebeat`)

![](../_media/Snipaste_2023-02-15_19-10-12.png ':size=80%')

在这里插入图片描述

我们可以看到如下的json数组，其中包含三个已经记录的文件对象，其中包含：

-   source：被监控文件的地址
-   offset：文件采集器（harvester）目前对于被监控文件的采集偏移（字节偏移，即已经采集到文件的哪个位置）
-   FileStateOS：文件的标识

```
[
  {
    "source": "/Users/lex.li/Downloads/2020_04_20/平台指标/db_oracle_11g.csv",
    "offset": 11769307,
    "timestamp": "2021-02-17T12:01:36.64399+08:00",
    "ttl": -1,
    "type": "log",
    "meta": null,
    "FileStateOS": {
      "inode": 124801232,
      "device": 16777220
    }
  },
  {
    "source": "/Users/lex.li/Downloads/2020_04_20/业务指标/esb.csv",
    "offset": 15146,
    "timestamp": "2021-02-17T12:01:12.839628+08:00",
    "ttl": -1,
    "type": "log",
    "meta": null,
    "FileStateOS": {
      "inode": 124801237,
      "device": 16777220
    }
  },
  {
    "source": "/Users/lex.li/Downloads/2020_04_20/调用链指标/trace_osb.csv",
    "offset": 16842764,
    "timestamp": "2021-02-17T12:01:37.760077+08:00",
    "ttl": -1,
    "type": "log",
    "meta": null,
    "FileStateOS": {
      "inode": 124801240,
      "device": 16777220
    }
  }
]
```

**因此，如果要让某个文件重新被采集，我们需要在这个`data.json`文件中删除对应的监控文件的对象，或者把`offset`置为0，然后重启filebeat!**

### clean\_inactive & clean\_removed

那除了手动修改注册表，我们还有没有其他办法来让filebeat重新采集某些文件呢？有的！还可以借助`clean_*`配置项：

-   clean\_inactive
-   clean\_removed

这些clean\_\*选项用于清除注册表文件中的状态条目。这些设置有助于减小注册表文件的大小，并可以防止潜在的[inode重用问题](https://www.elastic.co/guide/en/beats/filebeat/current/inode-reuse-issue.html)。而对于我们探讨的这种场景，它也是可用的，比如我们采集的文件是一个周期性产生的同名文件（比如，每2个小时产生一次），它会持续覆盖之前的文件。我们可以将对该文件的采集参数定义为：

```
ignore_older: 1h
clean_inactive: 2h
```

即2个小时后，该文件的状态会从注册表中删除。如果之后，该文件的状态更新，则会重新进行采集，并记录状态

#### clean\_inactive

启用此选项后，经过指定的不活动时间后，Filebeat会删除文件的状态。仅当Filebeat已忽略该文件（文件早于`ignore_older`）时，才能删除状态 。该`clean_inactive`设置必须大于`ignore_older + scan_frequency`，即我们要确保仍在收集的文件不能删除其状态。否则，该设置可能导致Filebeat不断重新发送全部内容，因为 clean\_inactive删除了Filebeat检测到的文件的状态。如果文件已更新或再次出现，则会从头开始读取文件。

该`clean_inactive`配置选项是有用的，以减少注册表文件的大小，特别是如果每天都在产生大量的新文件。

此配置选项对于防止因Linux上的inode重用而导致的Filebeat问题也很有用。有关更多信息，请参见Inode重用导致Filebeat跳过行。

> Tips: 在测试期间，您可能会注意到注册表包含本应根据clean\_inactive设置而被删除的状态条目。发生这种情况是因为**Filebeat直到再次打开注册表以读取其他文件时才删除条目**。如果要测试clean\_inactive设置，请确保Filebeat配置为从多个文件中读取，否则文件状态永远不会从注册表中删除。

#### clean\_removed

启用此选项后，如果在磁盘上找不到以最后一个已知名称命名的文件，则Filebeat将该文件从注册表中清除。这意味着在采集器完成后重命名的文件也将被删除。此选项默认情况下是启用的。

如果共享驱动器在短时间内消失并再次出现，则将从头开始再次读取所有文件，因为状态已从注册表文件中删除。在这种情况下，建议您禁用该 `clean_removed` 选项。

## 文件重命名或删除

另一个用户反馈的问题是，在windows系统上文件被filebeat打开用于采集之后，无法对文件进行重命名操作，提示“该文件正在被其他应用打开”。

对于这个问题，首先要解决的是采集效率的问题，即对于我们提到的这种场景的文件采集，filebeat需要尽快的完成数据的采集。其次，我们需要使用`close_*`参数，来让filebeat尽快关闭文件句柄

### close\_\*

`close_*`配置选项用于指定在达到某一标准或时间后关闭采集器。关闭采集器意味着关闭文件处理程序。如果在采集器关闭后更新文件，则文件将在`scan_frequency`经过后再次被采集。但是，如果在采集器关闭时移动或删除文件，Filebeat将无法再次采集该文件，并且采集器尚未读取的任何数据都将丢失。`close_*`当Filebeat尝试从文件读取时，这些设置将同步应用，这意味着如果Filebeat由于输出阻塞，完整队列或其他问题而处于阻塞状态，则本应关闭的文件保持打开状态，直到Filebeat再次试图从文件中读取数据（阻塞解除后，尝试读取新的数据时，发现达到了`close_*`指定的标准，此时才关闭采集器，释放文件）。

#### close\_inactive

启用该选项后，如果在指定的时间内没有收获文件，Filebeat会关闭文件句柄。所定义期间的计数器从采集器读取最后一行日志时开始。它不是基于文件的修改时间。如果已关闭的文件再次发生变化，则会启动一个新的采集器，并在scan\_frequency过后接收最新的变化。

我们建议你将close\_inactive设置成一个比日志文件最不频繁更新的值。例如，如果你的日志文件每隔几秒钟就会更新一次，你可以安全地将close\_inactive设置为1m。如果有更新率非常不同的日志文件，你可以使用多个不同值的配置。

将close\_inactive设置为一个较低的值意味着文件句柄会更快地被关闭。然而这有一个副作用，如果采集器被关闭，新的日志行不会近乎实时地发送。

关闭文件的时间戳不取决于文件的修改时间。相反，Filebeat使用一个内部时间戳来反映文件最后一次被收割的时间。例如，如果close\_inactive被设置为5分钟，那么这5分钟的倒计时从采集器读取文件的最后一行开始。

你可以使用时间字符串，比如2h（2小时）和5m（5分钟）。默认是5m。

#### close\_rename

启用此选项后，Filebeat 会在文件重命名时关闭文件处理程序。例如，在旋转文件时就会发生这种情况。默认情况下，采集器保持打开并继续读取文件，因为文件处理程序不依赖于文件名。如果close\_renamed选项被启用，并且文件被重命名或移动，以至于它不再与指定的文件模式相匹配，那么文件将不会被再次拾取。Filebeat将不会完成对文件的读取。

当配置了基于路径的file\_identity时，不要使用这个选项。启用该选项是没有意义的，因为Filebeat无法检测使用路径名作为唯一标识符的重命名。

WINDOWS: 如果你的Windows日志旋转系统因为不能旋转文件而显示错误，你应该启用这个选项。

#### close\_removed

当这个选项被启用时，Filebeat会在文件被删除时关闭采集器。通常情况下，一个文件只有在close\_inactive指定的时间内不活动后才会被删除。然而，如果一个文件被提前删除，而你又没有启用close\_removed，Filebeat会保持文件打开，以确保采集器已经完成。如果这个设置导致文件因为太早从磁盘上删除而无法完全读取，请禁用这个选项。

这个选项默认是启用的。如果你禁用这个选项，你也必须禁用clean\_removed。

WINDOWS: 如果你的Windows日志轮转系统因为无法轮转文件而显示错误，请确保启用此选项。

#### close\_eof

只有当你明白数据丢失是一个潜在的副作用时，才使用这个选项。

当启用此选项时，Filebeat会在文件结束时立即关闭文件。当你的文件只写一次，而不是时常更新时，这很有用。例如，当你把每一个日志事件写入一个新文件时，就会发生这种情况。这个选项默认是禁用的。

#### close\_timeout

只有当您了解数据丢失是一个潜在的副作用时才使用此选项。另一个副作用是，在超时之前，多行事件可能无法完全发送。

启用此选项后，Filebeat会给每个采集器一个预定义的寿命。无论阅读器在文件中的位置如何，在close\_timeout期过后，读取将停止。当你只想在文件上花费预定义的时间时，这个选项对旧的日志文件很有用。虽然close\_timeout会在预定义的超时后关闭文件，但如果文件仍在更新，Filebeat会按照定义的scan\_frequency再次启动新的采集器。而这个采集器的close\_timeout将以超时的倒计时重新开始。

这个选项在输出被阻塞的情况下特别有用，这使得Filebeat即使对从磁盘上删除的文件也能保持打开的文件处理程序。将close\_timeout设置为5m，可以确保定期关闭文件，以便操作系统可以释放它们。

如果你把close\_timeout设置为等于ignore\_older，那么如果文件在采集器关闭时被修改，就不会被接收。这种设置组合通常会导致数据丢失，并且不会发送完整的文件。

当你对包含多行事件的日志使用close\_timeout时，采集器可能会在多行事件的中间停止，这意味着只有部分事件会被发送。如果再次启动采集器，并且文件仍然存在，那么只有事件的第二部分会被发送。

这个选项默认设置为0，这意味着其不会停止监控

---

本文转自 [https://cloud.tencent.com/developer/article/1795007](https://cloud.tencent.com/developer/article/1795007)，如有侵权，请联系删除。
