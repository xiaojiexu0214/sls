# Logtail简介 {#concept_ppd_yx5_vdb .concept}

Logtail接入服务是日志服务提供的日志采集Agent，通过控制台方式帮助您实时采集阿里云ECS、自建IDC、其他云厂商等服务器上的日志。

![](images/2650_zh-CN.png "Logtail采集功能")

## 功能优势 {#section_ets_snw_w1b .section}

-   基于日志文件、无侵入式的收集日志。用户无需修改应用程序代码，且日志收集不会影响用户应用程序的运行逻辑。
-   除支持文本日志采集外，还支持binlog、http、容器stdout等采集方式。
-   对于容器支持友好，支持标准容器、swarm集群、Kubernetes集群等容器集群的数据采集。
-   能够稳定地处理日志收集过程中各种异常。当遇到网络异常、服务端异常等问题时会采用主动重试、本地缓存数据等措施保障数据安全。
-   基于服务端的集中管理能力。用户在安装Logtail后（参见 [安装Logtail（Windows系统）](intl.zh-CN/用户指南/Logtail采集/安装/安装Logtail（Windows系统）.md) 和 [安装Logtail（Linux系统）](intl.zh-CN/用户指南/Logtail采集/安装/安装Logtail（Linux系统）.md)），只需要在服务端集中配置需要收集的机器、收集方式等信息即可，无需逐个登录服务器进行配置。
-   完善的自我保护机制。为保证运行在客户机器上的收集Agent不会明显影响用户自身服务的性能，Logtail客户端在CPU、内存及网络使用方面都做了严格的限制和保护机制。

## 处理能力与限制 {#section_rbv_ydq_pdb .section}

参见[Logtail限制说明](intl.zh-CN/用户指南/Logtail采集/Logtail限制说明.md)。

## 配置流程 {#section_crw_bht_qy .section}

![](images/2652_zh-CN.png "配置流程")

通过Logtail采集服务器日志可以通过以下步骤完成：

1.  安装Logtail。在需要采集日志的源服务器上安装Logtail操作请参见[安装Logtail（Windows系统）](intl.zh-CN/用户指南/Logtail采集/安装/安装Logtail（Windows系统）.md) 和 [安装Logtail（Linux系统）](intl.zh-CN/用户指南/Logtail采集/安装/安装Logtail（Linux系统）.md)。
2.  [创建用户自定义标识机器组](intl.zh-CN/用户指南/Logtail采集/机器组/创建用户自定义标识机器组.md)。从阿里云ECS采集日志不需要执行此步骤。
3.  [创建IP地址机器组](intl.zh-CN/用户指南/Logtail采集/机器组/创建IP地址机器组.md)。日志服务通过机器组的方式管理所有需要通过Logtail客户端采集日志的服务器。日志服务支持通过IP或者自定义标识的方式定义机器组。您也可以在应用Logtail配置到机器组时，根据提示创建机器组。
4.  创建Logtail采集配置，并应用到机器组。您可以通过**数据接入向导**创建Logtail配置以 [采集文本日志](intl.zh-CN/用户指南/Logtail采集/文本日志/采集文本日志.md)、[Syslog](intl.zh-CN/用户指南/隐藏文件夹/Syslog.md)等，并将该Logtail配置应用到机器组。

在完成如上流程后，您的ECS服务器上需要收集的新增日志会被主动收集、发送到对应Logstore中，历史数据不会被收集。您可以通过日志服务控制台或者SDK及API查询到这些日志。您还可以通过日志服务查询到所有ECS服务器上的Logtail收集日志状态，例如是否在正常收集，是否有错误等。

Logtail接入服务在日志服务控制台上的完整操作请参考[Logtail 收集日志](intl.zh-CN/用户指南/Logtail采集/文本日志/采集文本日志.md) 。

## 容器 {#section_vgn_vfw_w1b .section}

-   阿里云容器服务Swarm：参见[集成日志服务](../../../../intl.zh-CN/用户指南/日志管理/集成日志服务.md)。
-   阿里云容器服务Kubernetes：参见[Kubernetes日志采集流程](intl.zh-CN/用户指南/Logtail采集/容器日志采集/Kubernetes日志采集流程.md)。
-   自建Kubernetes：参见[ZH-CN\_TP\_13059.md\#section\_kdx\_bqr\_zdb](intl.zh-CN/用户指南/Logtail采集/容器日志采集/Kubernetes日志采集流程.md#section_kdx_bqr_zdb)。
-   自建其他Docker集群：参见[标准Docker日志采集流程](intl.zh-CN/用户指南/Logtail采集/容器日志采集/标准Docker日志采集流程.md#)。

## 核心概念 {#section_h1d_jmw_w1b .section}

-   **机器组**：一个机器组包含一或多台需要收集一类日志的机器。通过绑定Logtail配置到机器组，可以让日志服务根据同样的Logtail配置采集一个机器组内所有服务器上的日志。您也可以通过日志服务控制台方便地对机器组进行管理（包括创建、删除机器组，添加、移除机器等）。同一个机器组内不可同时包含Windows和Linux机器，但可以包含不同版本的Windows Server或者不同发行版本的Linux机器。
-   **Logtail客户端**：Logtail是运行在需要收集日志的服务器上执行日志收集工作的Agent。安装步骤请参考 [安装Logtail（Windows系统）](intl.zh-CN/用户指南/Logtail采集/安装/安装Logtail（Windows系统）.md) 和 [安装Logtail（Linux系统）](intl.zh-CN/用户指南/Logtail采集/安装/安装Logtail（Linux系统）.md)。 在服务器上安装Logtail后，需要配置Logtail并应用到机器组。
    -   **Linux** 下，Logtail安装在 `/usr/local/ilogtail` 目录下，并启动两个以 ilogtail 开头的个独立进程，一个为收集进程，另外一个为守护进程，程序运行日志为 `/usr/local/ilogtail/ilogtail.LOG`。
    -   **Windows** 下，Logtail安装在目录 `C:\Program Files\Alibaba\Logtail`（32 位系统）或 `C:\Program Files (x86)\Alibaba\Logtail`（64 位系统）下。您可以通过Windows管理工具\>服务查看到两个Windows Service，LogtailWorker负责收集日志，LogtailDaemon负责守护工作程序。程序运行日志为安装目录下的 `logtail_*.log`。
-   **Logtail配置**：是Logtail收集日志的策略集合。通过为Logtail配置数据源、收集模式等参数，来对机器组内所有服务器进行定制化的收集策略。Logtail配置定义了如何在机器上收集一类日志并解析、发送到日志服务的指定日志库。您可以通过控制台对每个Logstore添加Logtail配置，表示该Logstore接收以此Logtail配置收集的日志。

## 基本功能 {#section_bz3_bnw_w1b .section}

Logtail接入服务提供如下功能：

-   **实时收集日志：**动态监控日志文件，实时地读取、解析增量日志。日志从生成到发往服务端的延迟一般在3秒内。

    **说明：** Logtail接入服务不支持对历史数据的收集。对于一条日志，读取该日志的时刻减去日志产生的时刻，差值超过12小时的会被丢弃。

-   **自动处理日志轮转：**很多应用会按照文件大小或者日期对日志文件进行轮转（rotation），把原日志文件重命名，并新建一个空日志文件等待写入。例如：监控app.LOG，日志轮转会产生 app.LOG.1，app.LOG.2 等。您可以指定收集日志写入的文件，如 app.LOG，Logtail会自动检测到日志轮转过程，保证这个过程中不会出现日志数据丢失。
-   **多种采集输入源：**Logtail除支持文本日志采集外，还支持syslog、http、MySQL binlog等输入源，更多内容参见采集数据源配置章节。
-   **兼容开源采集Agent：**Logtail支持Logstash、Beats等开源软件采集的数据作为输入源，更多内容参见采集数据源配置章节。
-   **自动处理收集异常：**因为服务端错误、网络措施、Quota超限等各种异常导致数据发送失败，Logtail会按场景主动重试。如果重试失败则会将数据写入本地缓存，稍后自动重发。
-   **灵活配置收集策略：**可以通过Logtail配置来非常灵活地指定如何在一台ECS服务器上收集日志。具体来说，您可以根据实际场景选择日志目录、文件，既可精确匹配，也可通过通配符模糊匹配。您可以自定义日志收集提取的方式和各个提取字段的名称，日志服务支持正则表达式方式的日志提取。另外，由于日志服务日志数据模型要求每条日志必须有精确的时间戳信息，Logtail提供了自定义的日志时间格式，方便您从不同格式的日志数据中提取必须要的日志时间戳信息。
-   **自动同步收集配置：**您在日志服务控制台上新建或更新配置，Logtail一般在3分钟时间内即可自动接受并使之生效，更新配置过程中数据收集不丢失。
-   **自动升级客户端：**在您手动安装Logtail到服务器后，日志服务负责Logtail 自动运维升级，此过程无需您参与。在整个Logtail升级过程中日志数据不丢失。
-   **自我监控状态：**为避免Logtail客户端消耗您太多资源而影响您其他服务。Logtail客户端会实时监控自身CPU和内存消耗。如果Logtail客户端在运行过程中，资源使用超出限制将会自动重启，避免影响机器上的其它作业。同时，该客户端也会有主动的网络限流保护措施，防止过度消耗用户带宽。
-   **签名数据发送：**为保证您的数据在发送过程中不会被篡改，Logtail客户端会主动获取用户的阿里云访问秘钥并对所有发送日志的数据包进行数据签名。

    **说明：** Logtail客户端在获取您的阿里云访问秘钥时采用HTTPS通道，保障您的访问秘钥安全性。


## 数据采集可靠性 {#section_gfl_w5s_cfb .section}

Logtail在采集数据时，会定期将采集的点位（CheckPoint）信息保存到本地，若遇到服务器意外关闭、进程崩溃等异常情况时，Logtail重启后会从上一次记录的位置处开始采集数据，尽可能保证数据不丢失。Logtail会根据配置文件中的[资源限制](intl.zh-CN/用户指南/Logtail采集/安装/配置启动参数.md)进行工作，若资源占用超过限定值5分钟以上，则Logtail会强制重启。重启后可能会产生一定的数据重复。

Logtail内部采用了很多机制提升日志采集可靠性，但并不能保证日志绝对不会丢失。以下情况可能造成日志丢失：

-   Logtail未运行且日志轮转多次。
-   日志轮转速度极快，例如1秒轮转1次。
-   日志采集速度长期无法达到日志产生速度。

