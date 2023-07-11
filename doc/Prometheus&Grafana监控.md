课件和软件包地址：https://pan.baidu.com/s/1LFGMfHVYfQ8nda0fuZsTfg?pwd=ndi7

bilibili：https://www.bilibili.com/video/BV1HT4y1Z7vR

## 一、Prometheus入门

　　Prometheus 受启发于 Google 的Brogmon 监控系统（相似的 Kubernetes 是从 Google的 Brog 系统演变而来），从 2012 年开始由前 Google 工程师在 Soundcloud 以开源软件的形式进行研发，并且于 2015 年早期对外发布早期版本。

2016 年 5 月继 Kubernetes 之后成为第二个正式加入 CNCF 基金会的项目，同年 6 月正式发布 1.0 版本。2017 年底发布了基于全新存储层的 2.0 版本，能更好地与容器平台、云平台配合。

Prometheus 作为新一代的云原生监控系统，目前已经有超过 650+位贡献者参与到Prometheus 的研发工作上，并且超过 120+项的第三方集成。

实现流程：

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307112322704.png)

### 1.1、Prometheus 的特点

　　Prometheus 是一个开源的完整监控解决方案，其对传统监控系统的测试和告警模型进行了彻底的颠覆，形成了基于中央化的规则计算、统一分析和告警的新模型。 相比于传统监控系统，Prometheus 具有以下优点：

> 1、易于管理

➢Prometheus核心部分只有一个单独的二进制文件，不存在任何的第三方依赖(数据库，缓存等等)。唯一需要的就是本地磁盘，因此不会有潜在级联故障的风险。
➢Prometheus基于Pull模型的架构方式，可以在任何地方（本地电脑，开发环境，测试环境）搭建我们的监控系统。
➢对于一些复杂的情况，还可以使用Prometheus服务发现(Service Discovery)的能力动态管理监控目标。

> 2、监控服务的内部运行状态

　　Pometheus鼓励用户监控服务的内部状态，基于Prometheus丰富的Client库，用户可以轻松的在应用程序中添加对Prometheus的支持，从而让用户可以获取服务和应用内部真正的运行状态。

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307112322853.png)

> 3、强大的数据模型

所有采集的监控数据均以指标(metric)的形式保存在内置的时间序列数据库当中
(TSDB)。所有的样本除了基本的指标名称以外，还包含一组用于描述该样本特征的标签。

> 4、强大的查询语言PromQL

Prometheus内置了一个强大的数据查询语言PromQL。通过PromQL可以实现对监控数据的查询、聚合。同时PromQL也被应用于数据可视化(如Grafana)以及告警当中。
通过PromQL可以轻松回答类似于以下问题：
➢在过去一段时间中95%应用延迟时间的分布范围？
➢预测在4小时后，磁盘空间占用大致会是什么情况？
➢CPU占用率前5位的服务有哪些？(过滤)

> 5、高效

　　对于监控系统而言，大量的监控任务必然导致有大量的数据产生。而Prometheus可以高效地处理这些数据，对于单一Prometheus Server实例而言它可以处理：
➢数以百万的监控指标
➢每秒处理数十万的数据点

> 6、可扩展

　　可以在每个数据中心、每个团队运行独立的Prometheus Sevrer。Prometheus对于联邦集群的支持，可以让多个Prometheus实例产生一个逻辑集群。

当单实例PrometheusServer处理的任务量过大时，通过使用功能分区(sharding)+联邦集群(federation)可以对其进行扩展。

> 7、易于集成

　　使用Prometheus可以快速搭建监控服务，并且可以非常方便地在应用程序中进行集成。目前支持：Java，JMX，Python，Go，Ruby，.Net，Node.js等等语言的客户端SDK，基于这些SDK可以快速让应用程序纳入到Prometheus的监控当中，或者开发自己的监控数据收集程序。
　　同时这些客户端收集的监控数据，不仅仅支持Prometheus，还能支持Graphite这些其他的监控工具。
　　同时Prometheus还支持与其他的监控系统进行集成：Graphite，Statsd，Collected，Scollector，muini，Nagios等。Prometheus社区还提供了大量第三方实现的监控数据采集支持：JMX，CloudWatch，EC2，MySQL，PostgresSQL，Haskell，Bash，SNMP，Consul，Haproxy，Mesos，Bind，CouchDB，Django，Memcached，RabbitMQ，
Redis，RethinkDB，Rsyslog等等。

> 8、可视化

➢Prometheus Server中自带的Prometheus UI，可以方便地直接对数据进行查询，并且支持直接以图形化的形式展示数据。同时Prometheus还提供了一个独立的基于Ruby On Rails的Dashboard解决方案Promdash。

➢最新的Grafana可视化工具也已经提供了完整的Prometheus支持，基于Grafana可以创建更加精美的监控图标。

➢基于Prometheus提供的API还可以实现自己的监控可视化UI。

> 9、开放性

　　通常来说当我们需要监控一个应用程序时，一般需要该应用程序提供对相应监控系统协议的支持，因此应用程序会与所选择的监控系统进行绑定。

为了减少这种绑定所带来的限制，对于决策者而言要么你就直接在应用中集成该监控系统的支持，要么就在外部创建单独的服务来适配不同的监控系统。
　　而对于Prometheus来说，使用Prometheus的client library的输出格式不止支持Prometheus的格式化数据，也可以输出支持其它监控系统的格式化数据，比如Graphite。

因此你甚至可以在不使用Prometheus的情况下，采用Prometheus的client library来让你的应用程序支持监控数据采集。

### 1.2、Prometheus 的架构

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307112322006.png)

### 1.3、Prometheus生态圈组件

➢Prometheus Server：主服务器，负责收集和存储时间序列数据
➢client libraies：应用程序代码插桩，将监控指标嵌入到被监控应用程序中
➢Pushgateway：推送网关，为支持short-lived作业提供一个推送网关
➢exporter：专门为一些应用开发的数据摄取组件—exporter，例如：HAProxy、StatsD、Graphite等等。
➢Alertmanager：专门用于处理alert的组件

### 1.4、架构理解

　　Prometheus既然设计为一个维度存储模型，可以把它理解为一个OLAP系统。

> 1、存储计算层

➢Prometheus Server，里面包含了存储引擎和计算引擎。
➢Retrieval组件为取数组件，它会主动从Pushgateway或者Exporter拉取指标数据。
➢Service discovery，可以动态发现要监控的目标。
➢TSDB，数据核心存储与查询。
➢HTTP server，对外提供HTTP服务。

> 2、采集层

采集层分为两类，一类是生命周期较短的作业，还有一类是生命周期较长的作业。
➢短作业：直接通过API，在退出时间指标推送给Pushgateway。
➢长作业：Retrieval组件直接从Job或者Exporter拉取数据。

> 3、应用层

应用层主要分为两种，一种是AlertManager，另一种是数据可视化。
➢AlertManager
对接Pagerduty，是一套付费的监控报警系统。可实现短信报警、5分钟无人ack打电话通知、仍然无人ack，通知值班人员Manager...
Emial，发送邮件
...
...
➢数据可视化
Prometheus build-in WebUI
**Grafana**
其他基于API开发的客户端

##  二、Prometheus相关软件安装

官网：https://prometheus.io/
下载地址：https://prometheus.io/download/

Prometheus基于Golang编写，编译后的软件包，不依赖于任何的第三方依赖。只需要下载对应平台的二进制包，解压并且添加基本的配置即可正常启动Prometheus Server。

### 2.1、安装包上传解压

将四个压缩包均上传到：/usr/local/prometheus目录下，然后执行下面的命令进行解压、重命名操作(直接粘贴我下面的即可)

```
cd /usr/local/prometheus/

tar -zxvf prometheus-2.29.1.linux-amd64.tar.gz
tar -zxvf pushgateway-1.4.1.linux-amd64.tar.gz
tar -zxvf node_exporter-1.2.2.linux-amd64.tar.gz
tar -zxvf alertmanager-0.23.0.linux-amd64.tar.gz

mv prometheus-2.29.1.linux-amd64 prometheus-2.29.1
mv pushgateway-1.4.1.linux-amd64 pushgateway-1.4.1
mv node_exporter-1.2.2.linux-amd64 node_exporter-1.2.2
mv alertmanager-0.23.0.linux-amd64 alertmanager-0.23.0
```

### 2.2、Prometheus配置与启动(*)

修改配置文件： cd prometheus-2.29.1/ && vim prometheus.yml 

这里的IP修改，如果是用的云服务器，写服务器IP的公网地址，我这边用的是虚拟机，这里就写虚拟机的IP了；

```
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["192.168.46.128:9090"]
  # 添加 PushGateway 监控配置
  - job_name: 'pushgateway'
    static_configs:
    - targets: ['192.168.46.128:9091']
      labels:
        instance: pushgateway

# 添加 Node Exporter 监控配置
  - job_name: 'node exporter'
    static_configs:
    - targets: ['192.168.46.128:9100']
```

> 1、配置说明：

1、global配置块：控制Prometheus服务器的全局配置
　　➢scrape_interval：配置拉取数据的时间间隔，默认为1分钟。
　　➢evaluation_interval：规则验证（生成alert）的时间间隔，默认为1分钟。
2、rule_files配置块：规则配置文件
3、scrape_configs配置块：配置采集目标相关，prometheus监视的目标。Prometheus自身的运行信息可以通过HTTP访问，所以Prometheus可以监控自己的运行数据。
　　➢job_name：监控作业的名称
　　➢static_configs：表示静态目标配置，就是固定从某个target拉取数据
　　➢targets：指定监控的目标，其实就是从哪儿拉取数据。Prometheus会从http://localhost:9090/metrics上拉取数据。

> 2、启动

```
nohup ./prometheus > ./prometheus.log 2>&1 &
```

访问测试：http://192.168.46.128:9090/

> 3、设置开机自启（可选）

1、写入启动脚本

vim start_prometheus.sh

```
#!/bin/bash
 cd /usr/local/prometheus/prometheus-2.29.1
nohup ./prometheus >> ./prometheus.log 2>&1 &
```

2、添加开机自启

```
chmod 777 /usr/local/prometheus/prometheus-2.29.1/start_prometheus.sh
echo "/usr/local/prometheus/prometheus-2.29.1/start_prometheus.sh" >> /etc/rc.d/rc.local
```

### 2.3、Pushgateway配置与启动

　Prometheus在正常情况下是采用拉模式从产生metric的作业或者exporter（比如专门监控主机的NodeExporter）拉取监控数据。

但是我们要监控的是Flink on YARN作业，想要让Prometheus自动发现作业的提交、结束以及自动拉取数据显然是比较困难的。

PushGateway就是一个中转组件，通过配置Flink on YARN作业将metric推到PushGateway，Prometheus再从PushGateway拉取就可以了。

```
nohup  ./pushgateway >> ./pushgateway.log 2>&1 &
```

访问测试：http://192.168.46.128:9091/metrics

> 设置开机自启（可选）

1、写入启动脚本

vim start_pushgateway.sh

```
#!/bin/bash
cd /usr/local/prometheus/pushgateway-1.4.1
nohup ./pushgateway >> ./pushgateway.log 2>&1 &
```

2、添加开机自启

```
chmod 777 /usr/local/prometheus/pushgateway-1.4.1/start_pushgateway.sh
echo "/usr/local/prometheus/pushgateway-1.4.1/start_pushgateway.sh" >> /etc/rc.d/rc.local
```

### 2.4、安装Alertmanager（选择性安装，此处就不进行安装启动）

 

### 2.5、安装NodeExporter(*)

　　在Prometheus的架构设计中，Prometheus Server主要负责数据的收集，存储并且对外提供数据查询支持，而实际的监控样本数据的收集则是由Exporter完成。

因此为了能够监控到某些东西，如主机的CPU使用率，我们需要使用到Exporter。Prometheus周期性的从Exporter暴露的HTTP服务地址（通常是/metrics）拉取监控样本数据。

　　Exporter可以是一个相对开放的概念，其可以是一个独立运行的程序独立于监控目标以外，也可以是直接内置在监控目标中。只要能够向Prometheus提供标准格式的监控样本数据即可。
为了能够采集到主机的运行指标如CPU,内存，磁盘等信息。

　　我们可以使用NodeExporter。Node Exporter同样采用Golang编写，并且不存在任何的第三方依赖，只需要下载，解压即可运行。可以从https://prometheus.io/download/获取最新的nodeexporter版本的二进制包。

```
nohup ./node_exporter >> node_exporter.log 2>&1 &
```

http://192.168.46.128:9100/metrics，可以看到当前node exporter获取到的当前主机的所有监控数据。

访问prometheus，就可以看到节点的状态了

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307112322961.png)

> 设置开机自启（可选）

1、写入启动脚本

vim start_node_exporter.sh

```
#!/bin/bash
cd /usr/local/prometheus/node_exporter-1.2.2
nohup ./node_exporter >> node_exporter.log 2>&1 &
```

2、添加开机自启

```
chmod 777 /usr/local/prometheus/node_exporter-1.2.2/start_node_exporter.sh
echo "/usr/local/prometheus/node_exporter-1.2.2/start_node_exporter.sh" >> /etc/rc.d/rc.local
```

## 三、　PromQL介绍

　　Prometheus 通过指标名称（metrics name）以及对应的一组标签（labelset）唯一定义一条时间序列。指标名称反映了监控样本的基本标识，而 label 则在这个基本特征上为采集到的数据提供了多种特征维度。用户可以基于这些特征维度过滤，聚合，统计从而产生新的计算后的一条时间序列。

　　PromQL 是 Prometheus 内置的数据查询语言，其提供对时间序列数据丰富的查询，聚合以及逻辑运算能力的支持。并且被广泛应用在Prometheus的日常应用当中，包括对数据查询、可视化、告警处理当中。

PromQL 是Prometheus 所有应用场景的基础.

访问：[http://localhost:9090:/metrics](http://localhost:9090/metrics) 可以查看所有指标

### 3.1、基本用法

#### 3.1.1 查询时间序列

　　当 Prometheus 通过 Exporter 采集到相应的监控指标样本数据后，我们就可以通过PromQL 对监控样本数据进行查询。

当我们直接使用监控指标名称查询时，可以查询该指标下的所有时间序列。如：

```
prometheus_http_requests_total                  
等同于： 
prometheus_http_requests_total{}     
```

该表达式会返回指标名称为 prometheus_http_requests_total 的所有时间序列：　　

```
prometheus_http_requests_total{
    code="200",
    handler="alerts",
    instance="localhost:9090",
    job="prometheus",
    method="get"
}= (20889@1518096812.326)

prometheus_http_requests_total{
    code="200",
    handler="graph",
    instance="localhost:9090",
    job="prometheus",
    method="get"
}= (21287@1518096812.326)
```

PromQL 还支持用户根据时间序列的标签匹配模式来对时间序列进行过滤，目前主要支持两种匹配模式：完全匹配和正则匹配。

➢ PromQL 支持使用 = 和 != 两种完全匹配模式：

⚫ 通过使用 label=value 可以选择那些标签满足表达式定义的时间序列；

⚫ 反之使用 label!=value 则可以根据标签匹配排除时间序列；

例如，如果我们只需要查询所有 prometheus_http_requests_total 时间序列中满足标签 instance 为 localhost:9090 的时间 序列，则可以使用如下表达式：

```
prometheus_http_requests_total{instance="localhost:9090"}       
```

反之使用 instance!=“localhost:9090” 则可以排除这些时间序列：　　

```
prometheus_http_requests_total{instance!="localhost:9090"}
```

➢ PromQL 还可以支持使用正则表达式作为匹配条件，多个表达式之间使用 | 进行分离：

⚫ 使用 label=~regx 表示选择那些标签符合正则表达式定义的时间序列；

⚫ 反之使用 label!~regx 进行排除；

例如，如果想查询多个环节下的时间序列序列可以使用如下表达式：

```
prometheus_http_requests_total{environment=~"staging|testing|development",method!="GET
"}
排除用法
prometheus_http_requests_total{environment!~"staging|testing|development",method!="GET
"}
```

#### 3.1.2 范围查询

　　直接通过类似于 PromQL 表达式 httprequeststotal 查询时间序列时，返回值中只会包含该时间序列中的最新的一个样本值，这样的返回结果我们称之为瞬时向量。而相应的这样的表达式称之为 瞬时向量表达式。

而如果我们想过去一段时间范围内的样本数据时，我们则需要使用区间向量表达式。区间向量表达式和瞬时向量表达式之间的差异在于在区间向量表达式中我们需要定义时间选择的范围，时间范围通过时间范围选择器 [] 进行定义。 例如，通过以下表达式可以选择

最近 5 分钟内的所有样本数据：

```
prometheus_http_requests_total{}[5m]
```

该表达式将会返回查询到的时间序列中最近 5 分钟的所有样本数据：

#### 3.1.3 时间位移操作

在瞬时向量表达式或者区间向量表达式中，都是以当前时间为基准：

```
prometheus_http_requests_total{} # 瞬时向量表达式，选择当前最新的数据 
prometheus_http_requests_total{}[5m] # 区间向量表达式，选择以当前时间为基准，5 分钟内的数据
```

而如果我们想查询，5 分钟前的瞬时样本数据，或昨天一天的区间内的样本数据呢? 这个时候我们就可以使用位移操作，位移操作的关键字为 offset。 可以使用 offset 时间位移操作：　　

```
prometheus_http_requests_total{} offset 5m
prometheus_http_requests_total{}[1d] offset 1d
```

#### 3.1.4 使用聚合操作

　　一般来说，如果描述样本特征的标签(label)在并非唯一的情况下，通过 PromQL 查询数据，会返回多条满足这些特征维度的时间序列。

而 PromQL 提供的聚合操作可以用来对这些时间序列进行处理，形成一条新的时间序列：

```
# 查询系统所有 http 请求的总量
sum(prometheus_http_requests_total)
# 按照 mode 计算主机 CPU 的平均使用时间
avg(node_cpu_seconds_total) by (mode)
# 按照主机查询各个主机的 CPU 使用率
sum(sum(irate(node_cpu_seconds_total{mode!='idle'}[5m]))     /     sum(irate(node_cpu_seconds_total [5m]))) by (instance)
```

#### 3.1.5 标量和字符串

除了使用瞬时向量表达式和区间向量表达式以外，PromQL 还直接支持用户使用标量(Scalar)和字符串(String)。

➢ 标量（Scalar）：一个浮点型的数字值

标量只有一个数字，没有时序。 例如：10

需要注意的是，当使用表达式 count(prometheus_http_requests_total)，返回的数据类型，依然是瞬时向量。用户可以通过内置函数scalar()将单个瞬时向量转换为标量。

➢ 字符串（String）：一个简单的字符串值

直接使用字符串，作为PromQL 表达式，则会直接返回字符串。

```
"this is a string"
'these are unescaped: \n \\ \t'
`these are not unescaped: \n ' " \t
```

#### 3.1.6 合法的 PromQL 表达式

　　所有的 PromQL 表达式都必须至少包含一个指标名称(例如 http_request_total)，或者一个不会匹配到空字符串的标签过滤器(例如{code=”200”})。

```
因此以下两种方式，均为合法的表达式：
prometheus_http_requests_total # 合法
prometheus_http_requests_total{} # 合法
{method="get"} # 合法

而如下表达式，则不合法：
{job=~".*"} # 不合法

同时，除了使用  {label=value}  的形式以外，我们还可以使用内置的 name 标签来指定监控指标名称：
{  name  =~"prometheus_http_requests_total"} # 合法
{  name  =~"node_disk_bytes_read|node_disk_bytes_written"} # 合法
```

### 3.2 PromQL 操作符

　　使用PromQL 除了能够方便的按照查询和过滤时间序列以外，PromQL 还支持丰富的操作符，用户可以使用这些操作符对进一步的对事件序列进行二次加工。这些操作符包括：数学运算符，逻辑运算符，布尔运算符等等。

#### 3.2.1 数学运算

PromQL 支持的所有数学运算符如下所示：

```
+ (加法)
- (减法)
* (乘法)
/ (除法)
% (求余)
^ (幂运算)
```

#### 3.2.2 布尔运算

➢ Prometheus 支持以下布尔运算符如下

```
== (相等)
!= (不相等)
>(大于)
< (小于)
>= (大于等于)
<= (小于等于)
```

➢ 使用bool 修饰符改变布尔运算符的行为

　　布尔运算符的默认行为是对时序数据进行过滤。而在其它的情况下我们可能需要的是真正的布尔结果。例如，只需要 知道当前模块的 HTTP 请求量是否>=1000，如果大于等于1000 则返回 1（true）否则返回 0（false）。这时可以使 用 bool 修饰符改变布尔运算的默认行为。 例如：

```
prometheus_http_requests_total > bool 1000
```

使用 bool 修改符后，布尔运算不会对时间序列进行过滤，而是直接依次瞬时向量中的各个样本数据与标量的比较结果 0 或者 1。从而形成一条新的时间序列。

同时需要注意的是，如果是在两个标量之间使用布尔运算，则必须使用 bool 修饰符：2 == bool 2 # 结果为 1

#### 3.2.3 使用集合运算符

使用瞬时向量表达式能够获取到一个包含多个时间序列的集合，我们称为瞬时向量。通过集合运算，可以在两个瞬时向量与瞬时向量之间进行相应的集合操作。

目前，Prometheus 支持以下集合运算符：

```
and (并且)
or (或者)
unless (排除)
```

#### 3.2.4 PromQL 聚合操作

Prometheus 还提供了下列内置的聚合操作符，这些操作符作用域瞬时向量。可以将瞬时表达式返回的样本数据进行 聚合，形成一个新的时间序列。

```
sum (求和)
min (最小值)
max (最大值)
avg (平均值)

stddev (标准差)
stdvar (标准差异)
count (计数)
count_values (对 value 进行计数)
bottomk (后 n 条时序)
topk (前n 条时序)
quantile (分布统计)
```

使用聚合操作的语法如下：　　

```
<aggr-op>([parameter,] <vector expression>) [without|by (<label list>)] 
```

其中只有 count_values , quantile , topk , bottomk 支持参数(parameter)。

without 用于从计算结果中移除列举的标签，而保留其它标签。by 则正好相反，结果向量中只保留列出的标签，其余标签则移除。通过 without 和 by 可以按照样本的问题对数据进行聚合。

例如：

```
sum(prometheus_http_requests_total) without (instance)                  等价于
sum(prometheus_http_requests_total) by (code,handler,job,method)       
```

如果只需要计算整个应用的 HTTP 请求总量，可以直接使用表达式：**sum(prometheus_http_requests_total)**　　

## 四、Prometheus整合

### 4.1、Prometheus整合Grafana

　　grafana 是一款采用 Go 语言编写的开源应用，主要用于大规模指标数据的可视化展现，是网络架构和应用分析中最流行的时序数据展示工具，目前已经支持绝大部分常用的时序数据库。

Grafana下载地址：https://grafana.com/grafana/download

Grafana监控面板下载：https://grafana.com/grafana/dashboards/

在前面讲解中，已经安装了软件：**NodeExporter**，负责收集本机数据，这时候Grafana的作用就是将这些数据做一个展示的功能！

> 1、安装

　　将软件包grafana-8.1.2.linux-amd64.tar.gz上传到/usr/local/grafana下，记得新建grafana文件夹

```
tar -zxvf grafana-enterprise-8.1.2.linux-amd64.tar.gz
cd grafana-8.1.2
nohup ./bin/grafana-server web > ./grafana.log 2>&1 &
```

访问：[localhost:3000](localhost:3000)，即可看到Grafana监控界面，默认用户名密码都是admin

> 2、加入开机自启：

```
#!/bin/bash
cd /usr/local/grafana/grafana-8.1.2
nohup ./bin/grafana-server web > ./grafana.log 2>&1 &
```

------

```
echo "/usr/local/grafana/grafana-8.1.2/start_grafana.sh"  >> /etc/rc.d/rc.local
```

>  3、Grafana关联Prometheus数据源

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307112323913.png)

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307112323614.png)

> 4、添加监控面板

导入网盘中给的文件：**node-exporter-for-prometheus-dashboard-cn-v20201010_rev24.json**

点击Import

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307112323316.png)

就可以看到了安装了**NodeExporter**软件的服务器相关资源信息：

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307112323910.png)

### 4.2、Prometheus整合SpringBoot应用

 参考：https://grafana.com/grafana/dashboards/12856-jvm-micrometer/

 本例使用的应用程序是ruoyi的非前后端分离项目进行演示：

> 1、项目添加pom

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
<!-- 可选, 用于进程内存使用图表 -->
<dependency>
    <groupId>io.github.mweirauch</groupId>
    <artifactId>micrometer-jvm-extras</artifactId>
    <version>0.2.0</version>
</dependency>
```

> 2、修改yml配置

```
management:
  endpoints:
    web:
      exposure:
        include: 'prometheus' # 暴露/actuator/prometheus
  metrics:
    tags:
      application: ${spring.application.name} # 暴露的数据中添加application label
```

> 3、代码中放行请求　　

说明，因为我这里使用若依项目进行演示，若依使用的安全框架是Shiro，还需要在Shiro的过滤器中放行/actuator/prometheus

com.ruoyi.framework.config.ShiroConfig.shiroFilterFactoryBean

```
filterChainDefinitionMap.put("/actuator/prometheus**", "anon");
```

配置完成后可以启动项目了

> 4、在Prometheus中添加采集信息

```
global:
  scrape_interval: 15s
scrape_configs:
  - job_name: "spring-demo"
    metrics_path: "/actuator/prometheus"
    static_configs:
    - targets: ["localhost:8080"]
```

下面把我的完整prometheus.yml配置贴出来：

```
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
    
# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["192.168.147.128:9090"]
  # 添加 PushGateway 监控配置
  - job_name: 'pushgateway'
    static_configs:
    - targets: ['192.168.147.128:9091']
      labels:
        instance: pushgateway

# 添加 Node Exporter 监控配置
  - job_name: 'node exporter'
    static_configs:
    - targets: ['192.168.147.128:9100','182.92.209.212:9100']

  - job_name: "ruoyi-admin"
    metrics_path: "/actuator/prometheus"
    static_configs:
    - targets: ["192.168.147.128:8080"]
```

重启Prometheus，就可以看到我们的监控参数了：

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307112323130.png)

> 5、Grafana面板展示　　

选择+ -> Import -> 粘贴url-->load
https://grafana.com/grafana/dashboards/12856

或者可以选择课件中的 JVM监控面板.json 进行导入
完整的效果如下:　

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307112323404.png)

### 4.3、Prometheus整合MySQL

**mysqld_exporter** 是 Prometheus 的 MySQL 指标导出插件。

Github 地址：https://github.com/prometheus/mysqld_exporter。这里来聊聊它的部署。

首先在 https://github.com/prometheus/mysqld_exporter/releases 中找到对应的 mysqld_exporter 版本。

按照官方的说法：也就是你的MySQL版本要>=5.6，才可以被监控到指标数据，各位自行检查自己的MySQL版本。

这里我就下载当前最新的mysqld_exporter版本：[mysqld_exporter-0.14.0.linux-amd64.tar.gz](https://github.com/prometheus/mysqld_exporter/releases/download/v0.14.0/mysqld_exporter-0.14.0.linux-amd64.tar.gz)

>  1、解压

新建/usr/local/mysqld_exporter将安装包上传到此文件夹下

```
tar -zxvf mysqld_exporter-0.14.0.linux-amd64.tar.gz
mv mysqld_exporter-0.14.0.linux-amd64 mysqld_exporter-0.14.0
cd mysqld_exporter-0.14.0
```

> 2、新建配置文件，指定Mysql监控的用户

这里我就直接使用root用户了，下面给出了创建用户以及授权的SQL，各位自行添加用户即可：

```
CREATE USER 'sonar'@'%' IDENTIFIED BY '123456'; -- 设置创建的用户名和密码，%表示该用户可以被远程连接，如果只是在本地使用的话可以使用localhost
GRANT ALL PRIVILEGES ON *.* TO 'sonar'@'%'; -- 授权用户所有权限
FLUSH PRIVILEGES; -- 刷新权限
```

 

新建配置文件，并写入创建的用户名密码：**vim mysqld_exporter.cnf**

```
[client]
user=xxx
password=xxx
```

> 3、启动mysqld_exporter程序

指定启动程序和配置文件地址即可

```
nohup /usr/local/mysqld_exporter/mysqld_exporter-0.14.0/mysqld_exporter --config.my-cnf=/usr/local/mysqld_exporter/mysqld_exporter-0.14.0/mysqld_exporter.cnf >> nohup.out 2>&1 &
```

> 4、测试访问

默认启动端口是9104，浏览器访问：http://192.168.147.128:9104/metrics，即可查看到mysql的监控数据信息了。

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307112323951.png)

> 5、Prometheus添加mysqld_exporter监控

**vim $PROMETHEUS_HOME/prometheus.yml**

```
scrape_configs:
# 整合Mysql
  - job_name: "mysql-panel"
    static_configs:
    - targets: ["192.168.147.128:9104"]
```

重启Prometheus，就可以在Prometheus中看到mysql相关的检测信息了。

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307112323294.png)

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307112323391.png)

> 6、整合Grafana

\+ --> import --> 在 “Import via grafana.com” 下方输入 7362：

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307112323287.png)

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307112323304.png)

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307112323196.png)

 