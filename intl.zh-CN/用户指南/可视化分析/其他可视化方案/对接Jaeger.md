# 对接Jaeger {#concept_ek5_rnq_zdb .concept}

容器、Serverless 编程方式的诞生极大提升了软件交付与部署的效率。在架构的演化过程中，可以看到以下两个变化：

-   应用架构开始从单体系统逐步转变为微服务，其中的业务逻辑随之而来就会变成微服务之间的调用与请求。
-   资源角度来看，传统服务器这个物理单位也逐渐淡化，变成了看不见摸不到的虚拟资源模式。

![](images/5875_zh-CN.png "架构演变")

从以上两个变化可以看到这种弹性、标准化的架构背后，原先运维与诊断的需求也变得越来越复杂。为了应对这种变化趋势，诞生了一系列面向 DevOps 的诊断与分析系统，包括集中式日志系统（Logging）、集中式度量系统（Metrics）和分布式追踪系统（Tracing）。

除Jaeger外，阿里云还提供支持 OpenTracing 链路追踪产品[XTrace](https://www.aliyun.com/product/xtrace)，欢迎使用。

## Logging，Metrics 和 Tracing {#section_z1s_p3q_12b .section}

Logging，Metrics 和 Tracing 各有特点。

-   **Logging用于记录离散的事件**。

    例如，应用程序的调试信息或错误信息。它是我们诊断问题的依据。

-   **Metrics用于记录可聚合的数据**。

    例如，队列的当前深度可被定义为一个度量值，在元素入队或出队时被更新；HTTP 请求个数可被定义为一个计数器，新请求到来时进行累加。

-   **Tracing用于记录请求范围内的信息**。

    例如，一次远程方法调用的执行过程和耗时。它是我们排查系统性能问题的利器。这三者也有相互重叠的部分，如下图所示。


![](images/5876_zh-CN.png "Logging，Metrics 和 Tracing")

通过上述信息，我们可以对已有系统进行分类。例如，Zipkin 专注于 tracing 领域；Prometheus 开始专注于 metrics，随着时间推移可能会集成更多的 tracing 功能，但不太可能深入 logging 领域； ELK，阿里云日志服务这样的系统开始专注于 logging 领域，但同时也不断地集成其他领域的特性到系统中来，正向上图中的圆心靠近。

关于三者关系的更详细信息可参考 [Metrics, tracing, and logging](http://peter.bourgon.org/blog/2017/02/21/metrics-tracing-and-logging.html?spm=a2c4e.11153959.blogcont514488.18.463f30c2m1sv0g)。下面为您重点介绍Tracing。

## Tracing技术背景 {#section_ewb_s3q_12b .section}

Tracing 是在90年代就已出现的技术。但真正让该领域流行起来的还是源于 Google 的一篇论文《Dapper, a Large-Scale Distributed Systems Tracing Infrastructure》，而另一篇论文《Uncertainty in Aggregate Estimates from Sampled Distributed Traces》中则包含关于采样的更详细分析。论文发表后一批优秀的 Tracing 软件孕育而生。

其中，比较流行的Tracing 软件有：

-   Dapper（Google）：各 tracer 的基础
-   StackDriver Trace（Google）
-   Zipkin（Twitter）
-   Appdash（golang）
-   鹰眼（Taobao）
-   谛听（盘古，阿里云云产品使用的Trace系统）
-   云图（蚂蚁Trace系统）
-   sTrace（神马）
-   X-ray（AWS）

分布式追踪系统发展很快，种类繁多，但核心步骤一般有三个：代码埋点，数据存储、查询展示。

下图是一个分布式调用的例子，客户端发起请求，请求首先到达负载均衡器，接着经过认证服务，计费服务，然后请求资源，最后返回结果。

![](images/5877_zh-CN.png "分布式调用示例")

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13157/15592902925883_zh-CN.png)

数据被采集存储后，分布式追踪系统一般会选择使用包含时间轴的时序图来呈现这个 Trace。但在数据采集过程中，由于需要侵入用户代码，并且不同系统的 API 并不兼容，这就导致了如果用户希望切换追踪系统，往往会带来较大改动。

## OpenTracing {#section_pxr_53q_12b .section}

为了解决不同的分布式追踪系统 API 不兼容的问题，诞生了 [OpenTracing](http://opentracing.io/?spm=a2c4e.11153959.blogcont514488.21.463f30c2m1sv0g) 规范。OpenTracing 是一个轻量级的标准化层，它位于应用程序/类库和追踪或日志分析程序之间。

![](images/5878_zh-CN.png "OpenTracing")

 **优势** 

-   OpenTracing 已进入 CNCF，正在为全球的分布式追踪，提供统一的概念和数据标准。
-   OpenTracing 通过提供平台无关、厂商无关的 API，使得开发人员能够方便的添加（或更换）追踪系统的实现。

 **数据模型** 

OpenTracing 中的 Trace（调用链）通过归属于此调用链的 Span 来隐性的定义。特别说明，一条 Trace（调用链）可以被认为是一个由多个 Span 组成的有向无环图（DAG图），Span 与 Span 的关系被命名为 References。

例如：下面的示例 Trace 就是由8个 Span 组成：

``` {#codeblock_2zi_v9w_9ay}
单个 Trace 中，span 间的因果关系
        [Span A]  ←←←(the root span)
            |
     +------+------+
     |             |
 [Span B]      [Span C] ←←←(Span C 是 Span A 的孩子节点, ChildOf)
     |             |
 [Span D]      +---+-------+
               |           |
           [Span E]    [Span F] >>> [Span G] >>> [Span H]
                                       ↑
                                       ↑
                                       ↑
                         (Span G 在 Span F 后被调用, FollowsFrom)
```

有些时候，使用下面这种，基于时间轴的时序图可以更好的展现 Trace（调用链）：

``` {#codeblock_hd5_kky_cxm}
单个 Trace 中，span 间的时间关系
––|–––––––|–––––––|–––––––|–––––––|–––––––|–––––––|–––––––|–> time
 [Span A···················································]
   [Span B··············································]
      [Span D··········································]
    [Span C········································]
         [Span E·······]        [Span F··] [Span G··] [Span H··]
```

每个 Span 包含以下状态：

-   An operation name，操作名称。
-   A start timestamp，起始时间。
-   A finish timestamp，结束时间。
-   Span Tag，一组键值对构成的 Span 标签集合。键值对中，键必须为 string，值可以是字符串、布尔、或者数字类型。
-   Span Log，一组 span 的日志集合。每次 log 操作包含一个键值对，以及一个时间戳。

键值对中，键必须为 string，值可以是任意类型。但是需要注意，不是所有的支持 OpenTracing 的 Tracer，都需要支持所有的值类型。

-   SpanContext，Span 上下文对象。
-   References（Span间关系），相关的零个或者多个 Span（Span 间通过 SpanContext 建立这种关系）。

每一个 SpanContext 包含以下状态：

-   任何一个 OpenTracing 的实现，都需要将当前调用链的状态（例如：trace 和 span 的 id），依赖一个独特的 Span 去跨进程边界传输。
-   Baggage Items，Trace 的随行数据，是一个键值对集合，它存在于 trace 中，也需要跨进程边界传输。

更多关于 OpenTracing 数据模型的知识，请参考 OpenTracing语义标准。

## 功能实现 {#section_fyr_53q_12b .section}

这篇[文档](https://opentracing.io/docs/supported-tracers/)列出了所有 OpenTracing 实现。在这些实现中，比较流行的为 [Jaeger](http://jaeger.readthedocs.io/en/latest/?spm=a2c4e.11153959.blogcont514488.24.463f30c2m1sv0g) 和 [Zipkin](https://zipkin.io/?spm=a2c4e.11153959.blogcont514488.25.463f30c2m1sv0g)。

## Jaeger {#section_gyr_53q_12b .section}

Jaeger 是 Uber 推出的一款开源分布式追踪系统，兼容 OpenTracing API。

## 架构 {#section_hyr_53q_12b .section}

![](images/5879_zh-CN.png "架构")

如上图所示，Jaeger 主要由以下几部分组成。

-   Jaeger Client：为不同语言实现了符合 OpenTracing 标准的 SDK。应用程序通过 API 写入数据，client library 把 trace 信息按照应用程序指定的采样策略传递给 jaeger-agent。
-   Agent：它是一个监听在 UDP 端口上接收 span 数据的网络守护进程，它会将数据批量发送给 collector。它被设计成一个基础组件，部署到所有的宿主机上。Agent 将 client library 和 collector 解耦，为 client library 屏蔽了路由和发现 collector 的细节。
-   Collector：接收 jaeger-agent 发送来的数据，然后将数据写入后端存储。Collector 被设计成无状态的组件，因此您可以同时运行任意数量的 jaeger-collector。
-   Data Store：后端存储被设计成一个可插拔的组件，支持将数据写入 cassandra、elastic search。
-   Query：接收查询请求，然后从后端存储系统中检索 trace 并通过 UI 进行展示。Query 是无状态的，您可以启动多个实例，把它们部署在 nginx 这样的负载均衡器后面。

## Jaeger on Alibaba Cloud Log Service {#section_qqh_gjq_12b .section}

[Jaeger on Alibaba Cloud Log Service](https://github.com/aliyun/aliyun-log-jaeger?spm=a2c4e.11153959.blogcont514488.27.463f30c2rnfGmn) 是基于 Jeager 开发的分布式追踪系统，支持将采集到的追踪数据持久化到日志服务中，并通过 Jaeger 的原生接口进行查询和展示。

![](images/5880_zh-CN.png "Jaeger")

## 优势 {#section_zpt_hjq_12b .section}

-   原生 Jaeger 仅支持将数据持久化到 cassandra 和 elasticsearch 中，用户需要自行维护后端存储系统的稳定性，调节存储容量。Jaeger on Alibaba Cloud Log Service 借助阿里云日志服务的海量数据处理能力，让您享受 Jaeger 在分布式追踪领域给您带来便捷的同时无需过多关注后端存储系统的问题。
-   Jaeger UI 部分仅提供查询、展示 trace 的功能，对分析问题、排查问题支持不足。使用 Jaeger on Alibaba Cloud Log Service，您可以借助日志服务强大的查询分析能力，助您更快分析出系统中存在的问题。
-   相对于 Jaeger 使用 elasticsearch 作为后端存储，使用日志服务的好处是支持按量付费，成本仅为 elasticsearch 的13%。参阅自建ELK vs 日志服务（SLS）全方位对比。

## 配置步骤 {#section_tqh_gjq_12b .section}

详细步骤请请参考[GitHub](https://github.com/aliyun/jaeger/blob/master/README_CN.md)。

## 配置示例 {#section_uqh_gjq_12b .section}

HotROD 是由多个微服务组成的应用程序，它使用了 OpenTracing API 记录 Trace 信息。

下面通过一段视频向您展示如何使用 Jaeger on Alibaba Cloud Log Service 诊断 HotROD 出现的问题。视频包含以下内容：

1.  配置日志服务
2.  通过 docker-compose 运行 Jaeger。
3.  运行 HotROD。
4.  通过 Jaeger UI 检索特定的 trace。
5.  通过 Jaeger UI 查看 trace 的详细信息。
6.  通过 Jaeger UI 定位应用的性能瓶颈。
7.  通过日志服务管理控制台，定位应用的性能瓶颈。
8.  应用程序调用OpenTracing API。

## 配置教程 {#section_vmq_p3q_12b .section}

[http://cloud.video.taobao.com//play/u/2143829456/p/1/e/6/t/1/50081772711.mp4](http://cloud.video.taobao.com//play/u/2143829456/p/1/e/6/t/1/50081772711.mp4)

示例中使用的query语句及其含义如下：

-   以分钟为单位统计 frontend 服务的 HTTP GET /dispatch 操作的平均延迟以及请求个数。

    ``` {#codeblock_v3x_c6f_hw0}
    process.serviceName: "frontend" and operationName: "HTTP GET /dispatch" |
    select from_unixtime( __time__ - __time__ % 60) as time,
    truncate(avg(duration)/1000/1000) as avg_duration_ms,
    count(1) as count
    group by __time__ - __time__ % 60 order by time desc limit 60
    ```

-   比较两条 trace 各个操作的耗时。

    ``` {#codeblock_56z_m5a_tdi}
    traceID: "trace1" or traceID: "trace2" |
    select operationName,
    (max(duration)-min(duration))/1000/1000 as duration_diff_ms
    group by operationName
    order by duration_diff_ms desc
    ```

-   统计延迟大于 1.5s 的 trace 的 IP 情况。

    ``` {#codeblock_qej_ng3_18e}
    process.serviceName: "frontend" and operationName: "HTTP GET /dispatch" and duration > 1500000000 |
    select "process.tags.ip" as IP,
    truncate(avg(duration)/1000/1000) as avg_duration_ms,
    count(1) as count
    group by "process.tags.ip"
    ```


