# WireShark笔记

【参考资料汇总】

* [Wiki](https://gitlab.com/wireshark/wireshark/-/wikis/home)
* [OSChina](https://tool.oschina.net/)



## 0825

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