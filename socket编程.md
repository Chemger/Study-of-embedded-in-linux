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
​		
​		

