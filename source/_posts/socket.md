---
title: 网络通信
date: 2021-9-3
tags:
	socket
---
网络通信
<!--more-->

# 1. 概述
电路交换
报文交换
分组交换
网络：使用某种媒介将不同地点的计算机连接起来，
      以达到信息交换、资源共享的目的
互联网：internet 网络互联  网网相联
因特网：Internet
协议：
要想让两台计算机进行通信，必须使它们采用相同的信息交换规则
计算机网络中用于规定信息的格式以及如何发送和接收信息的一套规则称为网络协议（network protocol）或通信协议（communication protocol），简称协议（Protocol）
其实就是用来说明网络中传输的二进制数据具体含义和作用的上下文
PDU：Protocol Data Unit 协议数据单元
	即协议对应的具体二进制数据
两种协议数据组织形式：
1. 文本
2. 二进制	
局域网：
以太网
因特网：TCP/IP协议族：TCP、IP、UDP、FTP、HTTP、RARP、SCTP
	TCP/IP栈        
OSI
TCP/IP五层模型：
应用层：直接为用户的应用进程提供服务
传输层：负责两个主机中进程之间的通信
网络层：负责不同主机之间的通信
数据链路层：负责相邻设备间的通信
物理层：负责传输比特流
MAC地址：网卡的每个网口有一个身份标识
	物理地址       
IP地址：Internet网上每台主机的身份标识
三种表示形式：
1. 4字节无符号整型
2. 点分十进制字符串:用.间隔四个0~255间的整型数所对应的字符串
   "192.168.1.66"
3. 域名
   www.sina.com
   www.baidu.com
域名---->4字节无符号整型
struct hostent *gethostbyname(const char *name)
功能：将域名发送给域名服务器，域名服务器发回IP地址等信息
      因此该函数要求电脑联网
返回值：返回的是调用进程数据区一块空间的首地址
	（因此无需释放）
	（后一次调用会覆盖前一次调用产生的结果）
```jsx
struct hostent
{
	char *h_name;
	char **h_aliases;//二次指向空间存放着域名别名
	int  h_addrtype;//始终为AF_INET宏常量
	int  h_length;  //始终为4
	char **h_addr_list;//二次指向空间存放着4字节整型IP地址
};
h_addr_list在程序中先强制转换成struct in_addr **二级指针来使用
struct in_addr
{
	in_addr_t s_addr;//in_addr_t是unsigned int类型的别名
};
char *inet_ntoa(struct in_addr in);
功能：将4字节整型IP地址转换成点分十进制字符串式IP地址
返回值：返回保存字符串的空间首地址
        返回的是调用进程数据区一块空间的首地址
       （因此无需释放）
       （后一次调用会覆盖前一次调用产生的结果）
       
int  inet_aton(const char *cp,struct in_addr *inp);
功能：将点分十进制字符串式IP地址转换成4字节整型IP地址
参数：
	cp：点分十进制字符串式IP地址
	inp：其指向空间用于存放转换结果
返回值：成功为非0，失败为0
```
网络号+主机号
主机号全0,该IP地址为对应网络的IP地址
主机号全1,为对应网络的广播地址
5类IP地址
127.0.0.1 本地环回
端口号：表示不同操作系统上不同进程身份的数字
16位整数
熟知端口，也称知名端口。范围从0到1023，如21ftp，80http
动态端口，范围从1024到65535，这些端口号一般不固定分配给某个服务，
	也就是说许多服务都可以使用这些端口
		  
# 2. UDP与TCP
TCP:面向连接的可靠的流式传输
	面向连接：收发数据前通讯双方需要建立好连接
	可靠：确保发送的成功或失败
	流式：确保数据次序
	服务器：提供服务的进程
	客户端：使用服务的进程	
UDP：非面向连接的不可靠的数据报式传输
TCP:
socket：套接字
        英文原意：插头、插排、插孔的统称       
        五属性：
        1. 协议族  AF_INET （Internet的TCP/IP协议族）
        2. 本地的IP地址
        3. 本地的端口号
        4. 对端的IP地址（插排没有）
        5. 对端的端口号（插排没有）       
        IP地址+端口号 ---->socket地址               
int socket(int domain,int type,int protocol)
domain:指定协议族，因特网的TCP/IP协议族用AF_INET
type：传输方式,SOCK_STREAM:流式传输
	           SOCK_DGRAM:数据报传输
protocol：指定传输层协议，填0时,socket函数会根据type参数来指定默认协议
          type是SOCK_STREAM时默认协议为TCP
          type是SOCK_DGRAM时默认协议为UDP            
字节序：整型数在其内存空间中的存放次序
1. 数的低位放在低地址：小(低)端字节序 little-endian
2. 数的高位放在低地址：大(高)端字节序 big-endian
网络字节序：大端字节序
struct sockaddr_in：因特网专用socket地址结构
struct sockaddr :通用socke地址结构
TCP客户端通过三路握手与服务器建立好连接
一个可以收发数据的socket必须五属性具备才能收发数据：
客户端socket什么时候指定五属性：
1. socket函数指定协议族
2. connect函数先指定对端（服务器）的IP地址、端口号
3. connect函数再指定本地（客户端）的IP地址、端口号
服务器端使用accept函数返回socket(插孔)与客户端进行数据交互：
1. 协议族由accept函数指定为与第一个参数（建立连接用的socket）的协议族相同的协议族
2. accept函数先指定对端（客户端）的IP地址、端口号
   该客户端就是已经与服务器建立好连接的客户端
3. accept函数再指定本地（服务器）的IP地址、端口号
   实际就是第一个参数（建立连接用的socket）的IP地址、端口号
   
# 3. 模板
```jsx
TCP客户端代码模板：
int sockfd = -1;
struct sockaddr_in servaddr;
int ret = 0;
sockfd = socket(AF_INET,SOCK_STREAM,0);//创建套接字
/*填写服务器IP地址端口号*/
bzero(&servaddr, sizeof(servaddr));
servaddr.sin_family = AF_INET; //指定协议族
servaddr.sin_port = htons(xxx); //指定服务器端口号
inet_aton(“a.b.c.d”, &servaddr.sin_addr);//指定服务器IP地址
//通过三路握手连接服务器
ret = connect(sockfd,  (struct sockaddr *)&servaddr,  sizeof(struct sockaddr_in));
if(ret < 0)
{
	printf("connect failed\n");
	//出错处理
}	
//……数据传输
close(sockfd);//关闭套接字
int connect(int sockfd, const struct sockaddr *servaddr, socklen_t addrlen);
功能：通过三路握手建立与TCP服务器的连接
返回值：成功为0，失败 < 0
参数说明：
sockfd：socket函数返回的socket描述符
servaddr：指向与协议族相符的地址结构指针
addrlen：实际地址结构的大小
备注：对于AF_INET协议族，其地址组成为IP地址+端口号，
      结构为struct  sockaddr_in
TCP服务器代码模板：
int servfd = -1;int datafd = -1;
struct sockaddr_in  servaddr;
int ret = 0;
servfd = socket(AF_INET, SOCK_STREAM, 0);//创建主动套接字（插头）
/*填写服务端自身IP地址和端口号*/
bzero(&servaddr, sizeof(servaddr));//
servaddr.sin_family = AF_INET;//指定协议族
inet_aton(“a.b.c.d”, &servaddr.sin_addr)//指定服务器IP地址
servaddr.sin_port = htons(xxx);//指定服务器端口号
/*服务器端IP地址和端口号必须告诉客户端知道，
  因此不能有系统自动指定，只能通过bind函数自己来指定，
  必须与客户端connect函数所用的IP地址与端口号一致*/
ret = bind(servfd, (struct sockaddr *) &servaddr, sizeof(servaddr));
/*
	listen函数功能是将主动套接字（插头）变为被动套接字（插排），
	然后通知被动套接字开始监听处理客户端连接请求。
	通知完后就返回了，因此该函数并不阻塞
*/
ret += listen(servfd, 5);
if(ret < 0)
{
	printf("bind or listen failed\n");
	//出错处理		
}
while(1) 
{	
   /*accept函数只有当有客户端与服务器连接建立成功后才返回。
     如果没有客户端发起连接请求或者连接请求还没有处理完毕
     该函数将一直阻塞到有连接建立成功。
     返回的是可以与已经建立好连接的客户端交互数据用的套接字（插孔）*/
   datafd = accept(servfd,NULL,NULL);   
   //……数据传输   
   close(datafd);
}
close(servfd);
int bind(int sockfd,  const struct sockaddr  *  myaddr, socklen_t addrlen);
功能：将本地协议地址赋予指定的socket描述符
返回值：成功为0，失败 < 0
参数：
sockfd： socket函数返回的socket描述符
myaddr：指向与协议族相符的地址结构指针
addrlen：实际地址结构的大小
备注：对于AF_INET协议族，其地址组成为IP地址+端口号，
      结构为struct  sockaddr_in
int  listen(int  sockfd,  int  backlog);
功能：将主动套接字改为被动套接字（用来接受指向该套接字的连接请求），
      并设定套接字排队的最大连接个数。
      该函数不会阻塞。只是通知被动套接字可以开始处理连接请求
返回值：成功0，失败 < 0
参数：
sockfd：用来专门处理客户端连接请求的套接字描述符（即服务器端自己调用socket函数产生的描述符）
backlog：指定管理连接请求的队列长度
系统针对每个被动套接字采用两个队列管理连接请求：
未完成连接队列
已完成连接队列
backlog实际上是一个基数，两个队列的长度由这个基数采用一定算法计算而得。
一般情况下backlog经验值为5-20。
对于大型服务器（如新浪主机）一般指定为系统能支持的最大数。
int accept(int sockfd,  struct sockaddr * cliaddr, socklen_t * addrlen);
功能：从已完成连接队列的对头返回一个已完成连接，队列空则阻塞等已完成的连接
返回值：成功返回已完成连接对应的socket描述符，服务器通过这个描述符与请求连接的客户端进行数据传输。
        失败 < 0
参数：
sockfd：监听套接字（即服务器用来处理连接请求专用的被动套接字）。
cliaddr：返回已连接的对端进程（客户端）的协议地址。
addrlen：值-结果参数，调用前指定cliaddr的地址结构长度，返回后保存
                     对应地址结构的确切字节数。
如果不关心客户端地址cliaddr和addrlen可以置为NULL。
本函数会阻塞。
通用读写函数
ssize_t read(int fd, void *buf, size_t count);
ssize_t write (int fd,const void * buf,size_t count);
网络编程专用读写函数
ssize_t recv(int sockfd, void *buff,size_t nbytes,int flags);
ssize_t send(int sockfd, const void *buff,size_t nbytes,int flags);
当fd为TCP socket描述符时，read和recv函数如果返回为0表示对端已经关闭连接
flags为0时，接收发送阻塞，为MSG_DONTWAIT时接收发送非阻塞

UDP编程：
ssize_t sendto(int sockfd,
               const void * buff,
               size_t nbytes,
               int flags, 
               const struct sockaddr * to,  
               socklen_t   addrlen);
功能：向指定地址发送期望字节数量的数据
返回值：成功返回实际发送的字节数。失败 < 0
参数：
sockfd、buff、nbytes、flags：含义同send函数
to：指向接收者的协议地址结构指针。
addrlen：to所用的地址结构的字节长度。
ssize_t recvfrom(int sockfd,
                 void * buff,
                 size_t nbytes,
                 int flags, 
                 struct sockaddr * from,
                 socklen_t  * addrlen);
功能：从指定的UDP-Socket里接收数据，同时填充发送方的协议地址结构
返回值：成功返回实际接收到的字节数。失败 < 0
参数：
sockfd、buff、nbytes、flags：含义同recv函数
from：指向发送方的协议地址结构指针，类似于accept对应参数的含义，用于保存发送方地址，以便于通过sendto回应对方相应数据。
addrlen：from所用的地址结构的字节长度，类似于accept对应参数的含义，指出实际使用的地址结构长度。
返回0是可接受的正确的结果，并非表示socket关闭，意味着可以发送空的UDP数据报（只有UDP首部而没有数据部分）给远端进程。
from和addrlen可以同时为NULL，表示不关心发送方地址信息

Client----->Server
1. login request
2. private_chat request
3. group_chat request
4. get userlist request
Server----->Client
1. login response
2. private_chat response
3. get userlist response
```