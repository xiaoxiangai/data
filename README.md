# 小象数据行为分析开源项目
#### 微信群
![输入图片说明](https://images.gitee.com/uploads/images/2021/0506/190130_320f09a4_5325125.png "屏幕截图.png")

#### 介绍
小象电商系统上线后，需要收集用户行为数据，通过大数据实时分析实现电商业务数字化运营。小象智慧基于此强需求开发小象行为日志产品并开源，产品兼容神策开源的埋点SDK完成终端行为上报，采用Nginx+Flume+kafka实现日志收集，采用Flink+ClickHouse架构实现OLAP的实时分析，同时数据会备份写入HDFS。

本开源项目内容包括nginx环境配置、Flume解密和日志格式处理、将明文数据存放到kafka的Topic下、Flink消费后将埋点数据存入HDFS的关键4步操作。为方便前期埋点的校验调优，在kafka环节，增加了埋点解析数据JSON格式存入MySQL。后续计划增加友盟和其他SDK厂商的埋点处理，以及业务系统日志的采集入库。

#### 项目主要内容
- 日志采集（Flume+kafka）
- 日志入库（Flink+HDFS）

#### 工作流程
完成数据采集技术构建和业务设计，在App、小程序的系统供应商配合下完成用户行为数据采集埋点，并基于埋点的数据构建线上用户行为标签和画像。
 ![输入图片说明](https://images.gitee.com/uploads/images/2021/0504/115612_bbd97789_5325125.png "屏幕截图.png")

#### 架构设计思路
所谓“埋点”，是数据采集领域（尤其是用户行为数据采集领域）的术语，指的是针对特定用户行为或事件进行捕获、处理和发送的相关技术及其实施过程。比如用户某个icon点击次数、观看某个视频的时长等等。
行为埋点的技术实质，是先监听软件应用运行过程中的事件，当需要关注的事件发生时进行判断和捕获。
![输入图片说明](https://images.gitee.com/uploads/images/2021/0504/115703_4013e7fa_5325125.png "屏幕截图.png") 

#### 业务设计思路
埋点业务设计，首先需要根据业务分析明确采集的目标行为，进一步搞清楚应该在哪些地方埋什么样的点。过程中建议使用“事件模型（ Event 模型）”来描述用户的各种行为，事件模型包括事件（ Event ）和用户（ User ）两个核心实体。
基于4W1H模型描述用户行为可将整个行为描述清楚，要点包括：是谁、什么时间、什么地点、以什么方式、干了什么。通过这两个实体结合在一起就可以清晰地描述清楚用户行为。
![输入图片说明](https://images.gitee.com/uploads/images/2021/0504/115753_e474aeec_5325125.png "屏幕截图.png")
 
#### 技术架构
SDK埋点采集行为数据来源终端包括iOS、安卓、Web、H5、微信小程序等。不同终端SDK采用对应平台和主流语言的SDK，埋点采集到的数据通过JSON数据以HTTP POST方式提交到服务端API。
服务端API由数据接入系统组成，采用Nginx来接收通过 API 发送的数据，并且将之写到日志文件上。使用Nginx实现高可靠性与高可扩展性。
对于Nginx打印到文件的日志，会由Flume的 Source 模块来实时读取Nginx日志，并由Channel模块进行数据处理，最终通过Sink模块将处理结果发布到 Kafka中。
Kafka是一个广泛使用的高可用的分布式消息队列，作为数据接入与数据处理两个流程之间的缓冲，同时也作为近期数据的一个备份。通过对外提供访问 API，数澜中台可以直接从 Kafka中将数据引走，进入数仓构建指标。
![输入图片说明](https://images.gitee.com/uploads/images/2021/0504/120743_443b0e95_5325125.png "屏幕截图.png")