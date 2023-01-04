# Pinpoint

[TOC]

## <u>文档标识</u>

| 文档名称 | Pinpoint |
| -------- | -------- |
| 版本号   | <V1.0.0> |

## <u>文档修订历史</u>

| 版本   | 日期      | 描述   | 文档所有者 |
| ------ | --------- | ------ | ---------- |
| V1.0.0 | 2022.12.6 | create | 杨丝雨     |
|        |           |        |            |
|        |           |        |            |

## <u>端口规划</u>

| 端口 | 协议 | remrks |
| ---- | ---- | ------ |
|      |      |        |
|      |      |        |
|      |      |        |
|      |      |        |

## <u>相关文档参考</u>

[Pinpoint官网]: https://pinpoint-apm.github.io/pinpoint/
[demo演示地址]: http://125.209.240.10:10123/main/ApiGateway@SPRING_BOOT/5m/2022-12-06-11-23-04
[Pinpoint github]: https://gitcode.net/mirrors/pinpoint-apm/pinpoint?utm_source=csdn_github_accelerator




## Pinpoint 简介
### Pinpoint 是什么
Pinpoint是一款全链路分析工具，提供了无侵入式的调用链监控、方法执行详情查看、应用状态信息监控等功能。基于GoogleDapper论文进行的实现，与另一款开源的全链路分析工具Zipkin类似，但相比Zipkin提供了无侵入式、代码维度的监控等更多的特性。 Pinpoint支持的功能比较丰富，可以支持如下几种功能：
			1、服务拓扑图：对整个系统中应用的调用关系进行了可视化的展示，单击某个服务节点，可以显示该节点的详细信息，比如当前节点状态、请求数量等
			2、实时活跃线程图：监控应用内活跃线程的执行情况，对应用的线程执行性能可以有比较直观的了解
			3、请求响应散点图：以时间维度进行请求计数和响应时间的展示，拖过拖动图表可以选择对应的请求查看执行的详细情况
			4、请求调用栈查看：对分布式环境中每个请求提供了代码维度的可见性，可以在页面中查看请求针对到代码维度的执行详情，帮助查找请求的瓶颈和故障原因。
			5、应用状态、机器状态检查：通过这个功能可以查看相关应用程序的其他的一些详细信息，比如CPU使用情况，内存状态、垃圾收集状态，TPS和JVM信息等参数。

### 架构组成

- Pinpoint 主要由 3 个组件外加 Hbase 数据库组成，三个组件分别为：Agent、Collector 和 Web UI。
  - Agent组件：用于收集应用端监控数据，无侵入式，只需要在启动命令中加入部分参数即可
  - Collector组件：数据收集模块，接收Agent发送过来的监控数据，并存储到HBase
  - WebUI：监控展示模块，展示系统调用关系、调用详情、应用状态等，并支持报警等功能

![架构图](https://www.xmxblog.com/wp-content/uploads/2021/07/123-300x167.png)

### **Google Dapper中分布式事务追踪**

当一个消息从Node1发送到Node2时，分布式追踪系统的作用是：在分布式系统中识别Node1和Node2之间的关系。

![img](https://upload-images.jianshu.io/upload_images/11639765-e6f746dc70fa56ef.png?imageMogr2/auto-orient/strip|imageView2/2/w/416/format/webp)

### **Pinpoint中的数据结构**

在Pinpoint中，核心数据结构由Spans、Traces和TraceIds组成。

1.Span：RPC追踪的基本单元。当一个RPC到达时，Span标示工作已经处理完成并包含追踪数据。为了确保代码级别的可见性，一个Span具有将SpanEvent标记为一个数据结构的子节点。

2.Trace：一个Span的集合。它由有关联的RPCs(Spans)组成。在相同Trace中的Spans，共享相同的TransactionId。

3.TraceId：由TransactionId、SpanId和ParentSpanId组成的一个keys的集合。TransactionId指示消息的ID，而SpanId和ParentSpanId表示RPCs的父-子关系。

4.TransactionId(TxId)：从单个事务跨分布式系统发送/接收的消息的ID。它必须跨整个服务器集群做到全局唯一。

5.SpanId：当收到RPC消息时一个工作被处理的ID。当一个RPC到达一个节点时生成SpanId。

6.ParentSpanId(pSpanId)：发起RPC的父Span的SpanId。如果一个节点是一个事务的起点，将没有父span - 对于这种情况，使用值-1表示这个span是一个事务的根span。

### TraceId如何工作

![img](https://upload-images.jianshu.io/upload_images/11639765-965650f1361e1d1a.png?imageMogr2/auto-orient/strip|imageView2/2/w/643/format/webp)

### Pinpoint优点、功能、支持模块

| 优点                                            | 功能                      | **支持的模块**                                               |
| ----------------------------------------------- | ------------------------- | ------------------------------------------------------------ |
| 1、分布式事务跟踪，跟踪跨分布式应用的消息       | 1、故障快速定位           | JDK 6+                                                       |
| 2、自动检测应用拓扑，帮助你搞清楚应用的架构     | 2、各个调用环节的性能分析 | Tomcat 6/7/8, Jetty 8/9, JBoss EAP 6                         |
| 3、水平扩展以便支持大规模服务器集群             | 3、数据分析等             | Spring, Spring Boot                                          |
| 4、提供代码级别的可见性以便轻松定位失败点和瓶颈 | 4、生成服务调用拓扑图     | Apache HTTP Client 3.x/4.x, JDK HttpConnector, GoogleHttpClient, OkHttpClient, NingAsyncHttpClient |
| 5、使用字节码增强技术，添加新功能而无需修改代码 |                           | Thrift Client, Thrift Service, DUBBO PROVIDER, DUBBO CONSUMER |
|                                                 |                           | MySQL, Oracle, MSSQL, CUBRID, DBCP, POSTGRESQL, MARIA        |
|                                                 |                           | Arcus, Memcached, Redis, CASSANDRA                           |
|                                                 |                           | iBATIS, MyBatis                                              |
|                                                 |                           | gson, Jackson, Json Lib                                      |
|                                                 |                           | log4j, Logback                                               |

### **环境搭建所需要的工具**

| 工具               | Remark                          |
| ------------------ | ------------------------------- |
| JDK                | 1.8以上                         |
| Tomcat 8.5.3       | 发布用                          |
| Pinpoint-Web       | 将收集到的数据显示成WEB网页形式 |
| Pinpoint-Collector | 收集各种性能数据                |
| Pinpoint-Agent     | 和自己运行的应用关联起来的探针  |
| HBase Storage      | 收集到的数据存到HBase中         |
| hbase_scripts      | Pinpoint初始化数据库            |



## PinPoint部署安装

| **HBase**             | **下载地址：http://archive.apache.org/dist/hbase/**          |
| --------------------- | ------------------------------------------------------------ |
| **PinPoint**          | **下载地址：https://github.com/pinpoint-apm/pinpoint/releases** |
| **JDK**               | **下载地址：https://www.oracle.com/java/technologies/downloads/#java8** |
| **Tomcat**            | **下载地址：https://tomcat.apache.org/**                     |
| **Hbase的Pinpoint库** | **下载地址：https://github.com/pinpoint-apm/pinpoint/tree/master/hbase/scripts** |

> ⚠️：本文已经安装完java和tomcat，所以不在复述java和tomcat安装（如有需要请执行java安装脚本和查看tomcat文档）

### 安装HBase

------

1、下载Hbase

```shell
wget https://archive.apache.org/dist/hbase/2.5.2/hbase-2.5.2-bin.tar.gz
```

2、解压安装安装文件

```shell
tar xf hbase-2.5.2-bin.tar.gz
```

3、配置JAVA_HOME环境变量

```shell
echo "export JAVA_HOME=/usr/local/java/1.8.0/" >> /usr/local/hbase-2.5.2/conf/hbase-env.sh
```

4、启动和停止服务

```shell
cd /usr/local/hbase-2.5.2/bin
./start-hbase.sh // 启动服务
./stop-hbase.sh  // 停止服务
```

5、浏览器上访问http://192.168.10.168:16010，查看安装信息

![image-20221207100830212](../../../Library/Application Support/typora-user-images/image-20221207100830212.png)

6、配置hbase的数据保存配置

```
vim /root/hbase-2.5.2/conf/hbase-site.xml
<configuration>
    <property>
        <name>hbase.rootdir</name>
        <value>file:root/hbase-2.5.2//hbase_data</value>
</property>
<property>
  <name>zookeeper.session.timeout</name>
  <value>120000</value>
</property>
<property>
  <name>hbase.zookeeper.property.tickTime</name>
  <value>6000</value>
</property>
</configuration>

```

7、启动hbase

```
# 停止
sh /root/hbase-2.5.2/bin/stop-hbase.sh
# 启动
sh /root/hbase-2.5.2/bin/start-hbase.sh
```

8、初始化Hbase的Pinpoint库

```shell
./hbase-1.2.6.1/bin/hbase shell /var/ftp/pub/hbase-create.hbase
```

![image-20230104145422592](../../../Library/Application Support/typora-user-images/image-20230104145422592.png)

9、从web端验证HBase是否启动成功

```
# 用浏览器访问16010端口可以查看HBase状态
http://本机ip:16010/master-status
```



> 下载地址：https://github.com/pinpoint-apm/pinpoint/releases/tag/v2.1.0

### 安装pinpoint-collector

------

```
wget https://github.com/pinpoint-apm/pinpoint/releases/download/v2.1.0/pinpoint-collector-boot-2.1.0.jar
```

- 以后台运行的方式启动 pinpoint-collector-boot-2.1.0.jar

```
nohup java -jar -Dpinpoint.zookeeper.address=localhost /root/pinpoint-collector-boot-2.1.0.jar > collector.log 2>&1 &
```

- 持续查看日志信息

```
tail –f collector.log
```

### 安装pinpoint-web

------

```
wget https://github.com/pinpoint-apm/pinpoint/releases/download/v2.1.0/pinpoint-web-boot-2.1.0.jar
```

- 以后台的方式运行：pinpoint-web-boot-2.1.0.jar

```
nohup java -jar -Dpinpoint.zookeeper.address=localhost /root/pinpoint-web-boot-2.1.0.jar > web.log 2>&1 &
```

查看是否启动成功

```
# 用jps查看java进程
jps
# 如果终端打印出pinpoint相关的任务,就启动成功了

32145 pinpoint-collector-boot-2.1.0.jar
1569 Jps
1315 Bootstrap
871 pinpoint-web-boot-2.1.0.jar
28477 HMaster
```

验证pinpoint-web是否安装成功

然后可以在浏览器中：`http://服务器ip:8080/` ，此时还没有agent

### 部署pinpoint-agent采集监控数据

- 基本配置部署

> 拷贝pinpoint-agent-2.1.0.tar.gz到web应用服务,这里环境都是一台服务器

```
wget https://github.com/pinpoint-apm/pinpoint/releases/download/v2.1.0/pinpoint-agent-2.1.0.tar.gz
tar zxvf pinpoint-agent-2.1.0.tar.gz
```

- 修改配置文件*\pinpoint-agent\ pinpoint-root.config、profiles/release/ pinpoint.config, 修改pinpoint-collector的地址

```
# 修改pinpoint-root.config配置
vim /root/pinpoint-agent-2.1.0/pinpoint-root.config
# 上面两个配置文件中将pinpoint-collector 修改为你的服务器ip地址,大约第20行

# 修改 pinpoint.config配置
vim /root/pinpoint-agent-2.1.0/profiles/release/pinpoint.config

# 上面这个两个配置文件的第20行都要修改为你的服务器ip地址

```

- 修改/root/pinpoint-agent-2.1.0/profiles/release/ pinpoint.config

```
# 修改 pinpoint.config配置的第60行,修改为1
vim /root/pinpoint-agent-2.1.0/profiles/release/pinpoint.config
	profiler.sampling.rate=1
```

### 监控tomcat

------

修改测试项目下的tomcat启动文件catalina.sh（linux）

```
（1）增加探针配置vi catalina.sh。

（2）添加如下配置# AGENT_PATH：pinpoint-agent的目录# AGENT_ID： agent的ID，这个ID是唯一的,不能重复。# APPLICATION_NAME： 采集项目的名字，这个名字可以随便取，只要各个项目不重复就好了#VERSION： agent 的版本。

```

```
CATALINA_OPTS="$CATALINA_OPTS -javaagent:/root/pinpoint-agent-2.1.0/pinpoint-bootstrap-2.1.0.jar"       #pinpoint包所在路径
CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.agentId=123456"
CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.applicationName=my-test1"
```

重新启动测试用的Tomcat的服务器

在pinpointweb浏览器中查看

![image-20230104150808714](../../../Library/Application Support/typora-user-images/image-20230104150808714.png)

### 监控springboot

如果是jar包部署，直接在启动命令加启动参数：

```
java -javaagent:/root/pinpoint-agent-2.1.0/pinpoint-bootstrap-2.1.0.jar -Dpinpoint.agentId=registry_windows -Dpinpoint.applicationName=woniuticket -jar registry-0.0.1-SNAPSHOT.jar

```

> 【注意】applicationName – 同一个项目建议相同；agentId – 不同的jar包自定义不同的id。
>
> 【说明】java -javaagent:pinpoint agent bootstrap-2.1.0.jar的路径 -Dpinpoint.agentId=agentid -Dpinpoint.applicationName=applicationName
