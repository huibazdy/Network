# 一、套接字基础

### 创建套接字

#### 1. 创建socket

```C
#include<sys/socket.h>
int socket(int domain,int type,int protocol);
```



#### 2. 给socket分配IP与端口号

```C
#include<sys/socket.h>
int bind(int sockfd,struct sockaddr *myaddr,socklen_t addrlen);
```



#### 3. 将socket转为监听状态

```C
#include<sys/socket.h>
int listen(int sockfd,int backlog);
```



#### 4. 受理连接请求

```C
#include<sys/socket.h>
int accept(int sockfd,struct sockaddr *addr,socklen_t *addrlen);
```



### 文件描述符（File Descriptor）

> **使用Linux提供的文件I/O函数，首先需要理解文件描述符**

文件描述符是操作系统分配给文件或者套接字的**整数**。

文件或套接字一般经过**创建过程后**才会被分配文件描述符。

文件描述符不过是方便称呼操作系统创建的文件或套接字而赋予的**代号**。

文件描述符有时也称为**文件句柄**（Windows中）。



### 文件I/O函数

#### 1.打开文件

要读些文件中的数据，首先要打开文件。为此定义了打开文件的函数，该函数有两个参数：1）目标文件名和路径；2）文件打开模式

```C
#include<sys/type.h>
#include<sys/stat.h>
#include<fcntl.h>

int open(const char *path,int flag); //成功返回文件描述符，失败返回-1
```

上述函数原型中的`path`是文件名的字符串地址，`flag`是文件打开模式信息。

关于打开模式的常量值极其含义见下表：

| 打开模式 | 含义                       |
| -------- | -------------------------- |
| O_CREAT  | 必要时创建文件             |
| O_TRUNC  | 删除全部现有数据           |
| O_APPEND | 维持现有数据，保存到其后面 |
| O_RDONLY | 只读打开                   |
| O_WRONLY | 只写打开                   |
| O_RDWR   | 读写打开                   |



#### 2.关闭文件

C语言学习过程中提到过，**使用文件后必须关闭**。

```C
#include<unistd.h>

int close(int fd);  //成功返回0，失败返回-1
```

此函数不仅**可以关闭文件也可以关闭套接字**，这也再次证明了对于Linux来说“万物皆文件”。



#### 3.写文件

写数据主要使用`write()`函数。Linux中不区分文件和套接字，所以通过socket向其他计算机传输数据时也会用到此函数。

```C
#include<unistd.h>

ssize_t write(int fd,const void* buf,size_t nbytes);  //成功时返回写入字节数，否则-1
```

其中参数`fd`是要写入数据的文件描述符，`buf`是保存要传输的数据缓冲区地址，`nbytes`是需要传输的字节数。`size_t`是typedef声明的unsigned in类型，`ssize_t`是typedef声明的signed int类型。

> `size_t`与`ssize_t`都是元数据类型（primitive），在`<sys/types.h>`中由定义



一个创建新文件并写入数据的例子：

```C
#include<stdio.h>
#include<stdlib.h>
#include<fcntl.h>
#include<unistd.h>
void err_handle(char* message);

int main()
{
    int fd;
    char buf[] = "Let's go!\n";
    
    fd = open("data.txt",O_CREAT|O_WRONLY|O_TRUNC);//创建并打开文件
    if(fd == -1)
        err_handle("open() error!");
    printf("file descriptor: %d\n",fd);
    
    if(write(fd,buf,sizeof(buf)) == -1) //写数据
        error_handle("write() error!");
    
    close(fd);  //关闭文件
    
    return 0;
}
```

需要说明的是文件打开模式是三种模式综合：创建空文件并只能写，若存在data.txt文件则清空文件的全部数据。



#### 4.读文件

```C
#include<unistd.h>

ssize_t read(int fd,void* buf,size_t nbytes); //成功返回接收字节数（遇文件结尾返回0），失败返回-1
```



```C
##include<stdio.h>
#include<stdlib.h>
#include<fcntl.h>
#include<unistd.h>
void err_handle(char* message);

int main()
{
    int fd;
    char buf[BUF_SIZE];
    
    fd = open("data.txt",O_RDONLY);  //打开文件，模式为只读
    if(fd == -1)
        err_handle("open() error!");
    printf("file descriptor: %d \n",fd);
    
    if(read(fd,buf,sizeof(buf)) == -1)  //读取文件数据
        err_handle("read() error!");
    printf("file data: %s",buf);
    
    close(fd);  //关闭文件
    
    return 0;
}
```



# 二、套接字协议

从创建套接字的函数`socket()`开始解析协议族

```C
#include<sys/socket.h>
int socket(int domain,int type,int protocol);
```

* `domain`

    表示socket使用的**协议族**是哪个：

    | 宏定义    | 协议族             |
    | --------- | ------------------ |
    | PF_INET   | IPv4协议族         |
    | PF_INET6  | Ipv6协议族         |
    | PF_LOCAL  | 本地通信Unix协议族 |
    | PF_PACKET | 底层套接字协议族   |
    | PF_IPX    | IPX Novell协议族   |

* `type`

    表示套接字类型，即：**数据传输方式**（决定了协议族不能同时决定数据传输方式）。有两种代表性的数据传输方式：

    | 宏定义      | 协议传输方式     |
    | ----------- | ---------------- |
    | SCOK_STREAM | 面向连接的套接字 |
    | SOCK_DGRAM  | 面向消息的套接字 |

* `protocol`

    该参数表示套接字**最终采用的协议**。

    前两个参数已经可以创建所需套接字了，大多数情况下第三个参数传递0即可。

    但是同一协议族中可能存在多个数据传输方式相同的协议，此时就需要向第三个参数传递指定最终的协议信息。