# 1.socket套接字
​	是一个编程接口（网络编程接口）
​	是一种特殊的文件描述符
​	

	socket是独立于具体的协议的编程接口，这个接口位于
	TCP/IP协议四层模型 的 应用层与传输层之间
	
	socket类型
	(1)流式套接字(SOCK_STREAM)
		面向字节流，针对传输层协议为 TCP的应用程序
		
	(2)数据报套接字(SOCK_DGRAM)
		针对传输层协议为 UDP的应用程序
		
	(3)原始套接字 （SOCK_RAW）
		一般用于本地 域协议
		直接跳过传输层

# 2.基本的TCP套接字编程流程
​	任何网络应用都有通信双方:
​	client
​	server
​		

	前面讲到任何网络应用都由 IP + 传输层协议 +端口 唯一确定
	-》
	“网络地址”	
		任何网络应用都需要有一个 “网络地址” IP+端口
	
	基本流程
		server的基本流程
		socket ：创建一个套接字
		bind:把一个套接字和网络地址 绑定在一起
		listen:让套接字进入“监听模式”
		accept:接收客户端的连接
		
		建立连接之后就可以进行通信了
		write/send/sendto		->发送数据
		read/recv/recvfrom		->接收数据
		
		close/shutdown


​		client的基本流程
​		socket
​		bind(可要可不要)
​		connect:主动连接服务端
​		
​		建立连接之后就可以进行通信了
​		write/send/sendto		->发送数据
​		read/recv/recvfrom		->接收数据
​		

		close/shutdown

# 3.socket具体的API函数解析
## (1) socket ：创建一个套接字
​		

```c
#include <sys/types.h>       
#include <sys/socket.h>
 
   int socket(int domain, int type, int protocol);
		domain：指定域，协议族。
			socket接口不仅仅局限于TCP/IP，它还可以用本地
			通信，蓝牙。。。每一种通信模式下面都有许多
			自己的协议，归到一类：协议族
			AF_UNIX, AF_LOCAL   unix域协议族
			AF_INET             IPv4协议族
		type：套接字类型
			SOCK_STREAM   流式套接字 用于TCP

			SOCK_DGRAM 	 数据报套接字 用于 UDP
			
			SOCK_RAW	原始套接字 用于 域协议

		protocol：指定具体的应用层协议
				一般为0，代表不知名的私有协议
				
	返回值：
		成功返回一个套接字描述符(>0 特殊的文件描述符)
		失败返回-1
	如：
		int sockfd = socket(AF_INET,SOCK_STREAM,0);
		创建一个用于 TCP通信的套接字
```

## (2)网络地址结构体


		socket接口不仅用于IPV4协议族，也可以用于蓝牙，域协议..
			不同的协议族，它的网络地址是不一样的
		
			socket编程接口，用一个通用的网络地址结构体
			struct sockaddr {
	           					sa_family_t sa_family;	//指定协议族
	          					 char        sa_data[14];
	     				          }
	
			//ipv4协议族的网络地址结构体：
			struct sockaddr_in
			{
				sa_family_t sin_family;	//指定协议族
				u_int16	sin_port;//端口号
				struct in_addr sin_addr;//ip地址
				char sin_zero[8];//填充了8个字节，为了和通用
								网络地址结构体一样大
			}
			
			//其中的in_addr结构体
			struct in_addr
			{
				in_addr_t s_addr;
			};
			
			typedef uint32_t in_addr_t
			
		例子：
			
			struct sockaddr_in addr;
			memset(&addr,0,sizeof(addr));
			（1）addr.sin_family = AF_INET;
			（2）addr.sin_port = htons(6666);//6666;
			//inet_aton("192.168.0.158",&(addr.sin_addr));
			（3）addr.sin_addr.s_addr = inet_addr("192.168.0.158");
			
		ip地址之间的转换函数
			"192.168.0.158"		
			点分式 字符串  <-> in_addr_t/struct in_addr
		#include <sys/socket.h>
	    #include <netinet/in.h>
	    #include <arpa/inet.h>
	
	   int inet_aton(const char *cp, struct in_addr *inp);
			用于把点分式的 ip 字符串转换为 struct in_addr
			cp:点分式字符串的首地址
			inp:指向一个struct in_addr结构体，
				用来保存转换后的ip
			返回值：
				成功返回0
				失败返回-1
		
		in_addr_t inet_addr(const char *cp);
		in_addr_t inet_network(const char *cp);
			功能一个，参数也一样，就是把一个点分式字符串
			转换为 in_addr_t 类型
		
		//发过来把ip转化成点分式
		char *inet_ntoa(struct in_addr in);
			把 struct in_addr 形式的ip转换为 点分式，返回
			其首地址


​	



## (3)bind :把一个套接字描述符和一个 网络地址 绑定起来



    #include <sys/types.h> 
    #include <sys/socket.h>
    
        int bind(int sockfd, const struct sockaddr *addr,
                socklen_t addrlen);
            sockfd：要绑定的套接字描述符
            addr:要绑定的网络地址 的指针
            addrlen：指定第二个参数指向的网络地址结构体的长度
        返回值：
            成功返回0
            失败返回-1		

​			

## (4)让套接字进入监听listen

```c
#include<sys/types.h>
#include<sys/socket.h>

int listen(int s, int backlog);

```

## (5)等待客户端的请求accept

处理客户端的连接请求

    #include <sys/types.h>         
    #include <sys/socket.h>   
       int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
    		//sockfd:套接字描述符
    		//addr: 通用的网络地址结构体变量的指针，
    				该结构体用来保存向我请求连接的客户端的
    				网络地址
    		
    		//addrlen：网络地址结构体长度类型的指针, 事先要保存第二个参数指向的结构体的长度
    					//目的是为了防止越界
    					//如果函数成功了, addrlen指向的空间会更新为真正的结构体网络地址的长度
    					int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen){
    						...
    						获取到了客户端的网络地址
    						clientaddr = ...
    						...
    						
    						memcpy(addr,&clientaddr,*addrlen);
    						8addrlen = sizeof(clientaddr);
    					}
    		//返回值: 成功返回一个新的套接字描述符, 这个描述符就是后续用来与客户端进行通信的
    					//思考: 为什么需要新的描述符, 而不直接用socket函数返回的那个套接字描述符来进行通信??
    					//因为服务端可以与多个客户端进行通信, 所以socket返回的套接字描述符是用来监听客户端和连接客户端的
    					//连接成功后返回的这个描述符才是真正用来与客户端进行通信的


​		

## (6)connet 用于tcp client去连接 tcp server

```c
#include <sys/types.h>
#include <sys/socket.h>

int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
//sockfd: 套接字描述符
//addr: 服务端的网络地址
//addrlen: 第二个参数的长度
//返回值: 成功返回0, 失败返回-1, 同时errno被设置
```

**==为什么服务端必须要bind, 而客户端不是必须的??==**

> bind还是不bind, 底层都会给套接字绑定一个网络地址(ip可能会自动获取, 但是端口号是任意值), 如果服务端不绑定, 意味着连**服务端自己都不确定自己的网络地址**, 就没办法公布给客户端
>
> 那客户端调用connect时, 怎么确定服务端的网络地址呢??
>
> 所以服务端必须要bind
>
> 而客户端只需要服务端的网络地址, 即可建立连接, 不知道自己的网络地址, 影响不大
>
> 所以: 客户端的bind不是必须的



​	作业：
​		1，设置好电脑，虚拟机及开发板的ip
​		2，了解 TCP 传输层头 的结构
​		3，了解三次握手和四次挥手
​		

## (7)还有几个函数用于网络通信

send/sendto

```c
#include<sys/types.h>
#include<sys/stocket.h>

ssize_t send(int sockfd, const void *buf, size_t len, int flags);
//前三个参数和write一样
//flags一般为0

ssize_t sendto(int sockfd, const void *buf, size_t len, int flags, const struct sockaddr *dest_addr, socklen_t addrlen);
//前四个参数和send一样
//dest_addr:指针, 指向一个网络地址结构体, 该网络地址保存了目的网络地址
//addrlen: 是第五个参数的指向的网络地址结构体的长度
```

为什么要比send多两个参数呢, 因为sendto一般是用于UDP通信, (UDP是无连接的, 所以必须得指定目的网络地址)

recv/recvfrom

```c
#include<sys/types.h>
#include<sys/stocket.h>

ssize_t recv(int sockfd, void *buf, size_t len, int flags);
//前三个参数和read一样
//flags一般为0

ssize_t recvfrom(int sockfd, const void *buf, size_t len, int flags, struct sockaddr *src_addr, socklen_t *addrlen);
//前四个参数和recv一样
//src_addr: 指针, 指向一个网络地址结构体, 该结构体保存了原网络地址
//addrlen: 指针, 指向的空间事先保存了原网络地址的长度, 防止越界. 函数成功之后, 保存真正的网络地址长度
```

同样recvfrom用于UDP通信, 所以必须指定源网络地址

# 4. TCP通信编程

[全图解被问千百遍的TCP三次握手和四次挥手面试题](https://mp.weixin.qq.com/s?__biz=MzU4ODI1MjA3NQ==&mid=2247485480&idx=2&sn=e0467536dcb58bd80e0f5c8dd6d466e6&chksm=fddedeeccaa957fa9756abe39e4404eb23e338db29d908bc4ccb746fe56eb59b17e3debaecfd&scene=0&xtrack=1&key=e49d2a8f87d27bc182a6cf9efc6603272e19a332960f5a4a0c965584fd3aa67871a13914037e913f3b44b61fa5616339d931016df1c3fa72c2692faf26fbd58550c5d5031226bb5e4116e041b5135eb1&ascene=14&uin=NDExMjM0MjQw&devicetype=Windows+10&version=62080085&lang=zh_CN&exportkey=AUdQQYMJzSYSXt%2B5%2FBX641I%3D&pass_ticket=BvLLLj%2B8R%2Bry2LpDh92hmtTnSWiZj3%2FubpVqK51x434ZgYNYD7ZdnDDKzQcs4FGI&winzoom=1#)

## tcp报文头部格式说明

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeo9xBVAyPJ8iaWCC6sYS843ZPb6tFLvCVuXEn98khfs7y2KRvOV0ia5icVByzIK3aAKRURuVZKagsKw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="TCP头部格式" style="zoom:50%;" />

### **序列号: Seq**

在建立连接时由计算机生成的随机数作为其初始值, 通过SYN包传给接收端主机, 每发送一次数据, 就累加一次该[数据字节数]的大小, 用来解决乱序问题

假设发送1000个字节, 分10个包发送, 为了不失序:

[1,100], [101,200], [201,300], ...

| 第一个包的Seq为x | 第一个包的数据字节数目为100 |
| ---------------- | --------------------------- |
| 第二个包的Seq为x+100 | 第二个包的数据字节数目为100 |

不仅能解决乱序问题, 

还能检测数据是否丢失

### 确认应答号: Ack

指下一次期望收到的数据序列号, 发送端接收到了这个确认应答后, 可以认为在这个序列号之前的所有数据都收到了, 用来解决丢包问题

### SYN

该标志位为1时, 表示希望建立连接, 并且在其Seq字段进行Seq初始值的设定, 出现在三次握手时

### FIN 

该标志位为1时, 表示今后不会再有数据发送, 希望断开连接

出现在四次挥手时

### ACK

该标志位为1时, 确认应答号: Ack才有效

TCP规定, 除了最初建立连接时的SYN包, 



---

## 三次握手

<img src="https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeo9xBVAyPJ8iaWCC6sYS843fFol7gd3035Kibg3gPMSAZQLVibf9nwEblOUaX80hoOaRLVpaYCAI44w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 zoom" style="zoom:60%;" />

为什么要"握手"??

因为TCP是面向连接的, 必须要确保建立好连接之后才能通信

"握手" 就是为了确保连接已经建立好了

## 四次挥手

1. 客户数据发送完成, 向服务器发送断开连接请求

2. 服务器响应客户的断开请求,

   服务器数据发送完成之后

3. 服务器向客户发送断开连接请求

4. 客户响应服务器的断开请求, 并且等待2MSL才关闭, 服务器接收到响应就关闭, 如果在2MSL内又接收到断开请求, 就会再次响应

# 5. UDP通信编程

```c
int shutdown(int sockfd, int how);
//sockfd: 套接字
//how: 关闭方式
	//SHUT_RD: 关闭读
	//SHUT_WR: 关闭写
	//SHUT_RDWR: 关闭读写, 等价于close()
```

