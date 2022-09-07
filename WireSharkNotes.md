# WireShark分析TCP/IP

【参考资料汇总】

* [Wiki](https://gitlab.com/wireshark/wireshark/-/wikis/home)
* [OSChina](https://tool.oschina.net/)
* https://zhangbinalan.gitbooks.io/protocol/content/tcpbao_wen_ge_shi.html
* https://zhuanlan.zhihu.com/p/92993778
* https://developer.aliyun.com/article/906989

## 一、wireshark基本使用

这里举例Windows环境，利用`ipconfig`命令查看本地网络配置信息：

<img src="https://raw.githubusercontent.com/huibazdy/TyporaPicture/main/202208251719978.png" alt="image-20220825171906880" style="zoom: 50%;" />

可以观察到我的电脑有两个网卡在使用中：有线网络（以太网3）与无线网络（WLAN），它们的**IP地址**（IPv4）分别是`10.239.130.215`以及`10.239.154.55`。

使用命令`ipconfig/all`查看电脑的**MAC地址**：

<img src="https://raw.githubusercontent.com/huibazdy/TyporaPicture/main/202208251743916.png" alt="image-20220825174300845" style="zoom:50%;" />

有线网络的MAC地址是：`2C-16-DB-A7-42-0A`，无线网的MAC地址是：`14-85-7F-77-2A-EF`。



以自己抓取并保存的一个时间段的wireshark文件分析为例：

![image-20220825194046069](https://raw.githubusercontent.com/huibazdy/TyporaPicture/main/202208251940281.png)

窗口1所在的部分是抓取到的所有数据包（Frame）的列表；窗口2是窗口1中选定的某个数据包的详细分层（这里是TCP/IP五层模型）说明；窗口3是窗口1中选取的数据包的数据源，左半部分是16进制，右半部分是16进制对应的额ASCII码。



【**传输层**】

在这里传输层使用的是TCP协议，在展开窗口2的传输层详情后，可以在详情清楚看到选定数据包（Frame）的TCP报文详细信息。





【**设置过滤器**】

> 在不设置过滤条件的情况下，会抓取到大量冗余的数据包，不利于进行包分析

设置过滤器主要可以分为两种情况：

* 在开始**抓取前**设置抓取过滤规则
* 在**抓取后**设置显示过滤规则





## 二、IP协议

需要知道的是，每个数据包的某层协议数据信息都会表明上一层是什么协议，例如：

![image-20220907161456438](https://raw.githubusercontent.com/huibazdy/TyporaPicture/main/202209071614487.png)

在这里，打开数据链路层的数据信息，可以看到有一个`Type`字段在最后标明上一层使用的协议是IPv4。



同理，同一个数据包，在网络层数据信息的最后`Protocol`字段也标明了上层使用的协议是：TCP

![image-20220907162027730](https://raw.githubusercontent.com/huibazdy/TyporaPicture/main/202209071620765.png)



## 三、TCP协议

### TCP报文

<img src="https://raw.githubusercontent.com/huibazdy/TyporaPicture/main/202209071624749.png" alt="image-20220907162429702"  />

### 实例分析

#### 三次挥手

![image-20220907163953644](https://raw.githubusercontent.com/huibazdy/TyporaPicture/main/202209071639682.png)

* 第一次挥手（客户端发送TCP请求）

    1. `61658->443`表示客户端请求连接
    2. `seq=0`且`[SYN]`表示SYN置为1

    ![image-20220907165812685](https://raw.githubusercontent.com/huibazdy/TyporaPicture/main/202209071658725.png)

* 第二次挥手（服务端响应TCP请求）

    1. `443->61658`表示服务端口响应客户端口的TCP请求
    2. `seq=0`且`[SYN]`置位为1，返回确认号`[ACK]`且ACK号为客户端发过来的seq号加1，即为1

    ![image-20220907170203794](https://raw.githubusercontent.com/huibazdy/TyporaPicture/main/202209071702836.png)