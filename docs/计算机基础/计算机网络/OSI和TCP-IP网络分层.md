## OSI七层模型

![image-20220823171004694](https://picture.lingzero.cn/img/image-20220823171004694.png)

![img](https://picture.lingzero.cn/img/202208231724894.png)



## TCP/IP 四层模型

![TCP/IP 各层协议概览](https://picture.lingzero.cn/img/202208231726150.png)

![image-20220823174538424](https://picture.lingzero.cn/img/202208232158136.png)



### 1.应用层

**应用层位于传输层之上，主要提供两个终端设备上的应用程序之间信息交换的服务，它定义了信息交换的格式，消息会交给下一层传输层来传输。** 

![应用层重要协议](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/cs-basics/network/application-layer-protocol.png)

#### HTTP报文

![image-20220823214045544](https://picture.lingzero.cn/img/202208232158137.png)

![image-20220823215839335](https://picture.lingzero.cn/img/202208232158138.png)

![image-20220823215922098](https://picture.lingzero.cn/img/202208232159237.png)

### 2.传输层

**传输层的主要任务就是负责向两台终端设备进程之间的通信提供通用的数据传输服务**

**运输层主要使用以下两种协议：**

1. **传输控制协议 TCP**（Transmisson Control Protocol）--提供 **面向连接** 的，**可靠的** 数据传输服务。
2. **用户数据协议 UDP**（User Datagram Protocol）--提供 **无连接** 的，尽最大努力的数据传输服务（不保证数据传输的可靠性）。

![传输层重要协议](https://picture.lingzero.cn/img/202208232010372.png)

![image-20220823201249566](https://picture.lingzero.cn/img/202208232158139.png)

#### TCP报文

计算机网络谢希仁 P217

特点

1. **面向连接的运输层协议**
2. **每一条TCP连接只能有两个端点(endpoint),每一条TCP连接只能是点对点的(一 对一)。**
3. **TCP提供可靠交付的服务**
4.  **TCP提供全双工通信**
5.  **面向字节流**![image-20220823201802134](https://picture.lingzero.cn/img/202208232158140.png)

#### UDP报文

**计算机网络谢希仁 P208**

特点

1. **无连接，**
2. **尽最大努力交付，**
3. **面向报文，**
4. **没有拥塞控制，**
5. **支持一对一、一对多、多对一和多对多的交互通信**
6. **首部开销小，只有8个字节，比TCP的20个字节的首部要短。**

![image-20220823201347912](https://picture.lingzero.cn/img/202208232158141.png)

### 3.网络层

网络层属于主机之间的通信，他的目的是向上提供简单灵活的，无连接的，尽最大努力交付的数据报服务，不保证可靠性

![image-20220823194538216](https://picture.lingzero.cn/img/202208232158142.png)

![网络层重要协议](https://picture.lingzero.cn/img/202208231747743.png)



#### IP数据报

计算机网络谢希仁 P128

![image-20220823195119906](https://picture.lingzero.cn/img/202208232158143.png)

#### ICMP报文

计算机网络谢希仁 P147

![image-20220823194936669](https://picture.lingzero.cn/img/202208232158144.png)

![image-20220823195336771](https://picture.lingzero.cn/img/202208232158145.png)

#### RIP报文格式

计算机网络谢希仁 P157

内部网关协议

分布式的**基于距离向量的路由选择协议**

RIP协议使用运输层的用户数据报**UDP**进行传送（使用UDP的端口 520）

![image-20220823195623006](https://picture.lingzero.cn/img/202208232158146.png)



#### OSPF报文

计算机网络谢希仁 P161

内部网关协议

**开放最短路径优先OSPF**(Open Shortest Path First）

OSPF不用**UDP**而是直接用**IP数据报**传送(其IP数据报首部的协议字段值为89)

![image-20220823200026738](https://picture.lingzero.cn/img/202208232158147.png)

#### BGP报文

外部网关协议一**边界网关协议BGP**

![image-20220823200405580](https://picture.lingzero.cn/img/202208232158148.png)

#### IPv6数据报

计算机网络谢希仁 P172

![image-20220823194651262](https://picture.lingzero.cn/img/202208232158149.png)

![image-20220823194625655](https://picture.lingzero.cn/img/202208232158150.png)

### 4.网络接口层（Network interface layer）

1. 数据链路层(data link layer)通常简称为链路层（ 两台主机之间的数据传输，总是在一段一段的链路上传送的）。**数据链路层的作用是将网络层交下来的 IP 数据报组装成帧，在两个相邻节点间的链路上传送帧。每一帧包括数据和必要的控制信息（如同步信息，地址信息，差错控制等）。**
2. **物理层的作用是实现相邻计算机节点之间比特流的透明传送，尽可能屏蔽掉具体传输介质和物理设备的差异**

![网络接口层重要协议](https://picture.lingzero.cn/img/202208232113764.png)