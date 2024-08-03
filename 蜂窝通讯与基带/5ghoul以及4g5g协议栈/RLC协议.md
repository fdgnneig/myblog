# https://blog.csdn.net/HiWangWenBing/article/details/114624632
- [4G&5G专题-57]：L2 RLC层-详解RLC架构、数据封装、三种模式：透明TM、非确认模式UM、确认模式AM_lte系统中rlc的头部信息-CSDN博客 (2023_12_26 23_55_24).html
- L2（数据链路层），又称无线网络层，实现终端与基站之间通过无线信道（逻辑信道）传递分组数据。
  - （1）PDCP（Packet Data Convergence Control，分组数据汇聚控制层）
  - （2）RLC（Radio Link Control，无线链路控制层）
- 无线链路控制协议（RLC）是为了保证数据传输业务可靠服务质量（QoS）而制定的协议。
- RLC协议是在数据链路控制（DLC）层中引进了多个新的自动重发请求（ARQ）机制，以此来解决对服务质量的要求。

SDU（Service Data Unit）服务数据单元，对应于某个子层中没有被处理的数据。对于某个子层而言，表示由上一层传递到本层还未被处理的数据。
PDU（Protocol Data Unit）协议数据单元，对应于被该子层处理形成特定格式的数据。对于某个子层而言，表示将本层SDU经过特定格式处理后将传递到下一层的数据。

SUD和PDU是相对的概念

上层（N+1层）的PDU，对本层（N层）而言，就是N层的SDU,  N层的SDU+ N层的头信息，就变成了N层的PDU。
本层（N层）的PDU，就下层（N-1层）而言，就是N-1层的SDU。

RLC在空口协议栈中处于L2层，位于L2 PDCP和MAC之间，其向上提供三种无线链路层数据传输服务：

透明传输TM(类RAW Socket模式)、非确认模式UM(类UDP模式)、确认模式AM(类TCP模式)。

有点类似与TCP/IP协议栈提供的三种数据传输服务。

信令无线承载”（SRB）定义为仅仅用于RRC和NAS消息传输的无线承载（RB）。更具体地讲，定义如下三种SRB：
-  SRB0用于RRC 消息，使用CCCH逻辑信道；也就是SRB0在RLC实体的传输类型是TM模式。
-  SRB1 用于RRC 消息, 使用DCCH逻辑信道；同时对于NAS消息，SRB1先于SRB2的建立，RLC实体的传输类型是AM或UM模式
-  SRB2 用于NAS消息，使用DCCH逻辑信道。SRB2要后于 SRB1建立，并且总是由E-UTRAN在安全激活后进行配置。

# https://www.mscbsc.com/viewnews-101731.html
- 【LTE基础知识】RLC子层协议概述 - 移动通信网 (2023_12_26 23_54_58).html