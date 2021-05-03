# unix域协议

## 1. unix域协议

进程间通信(IPC)方式之一

socket接口进行本地进程间通信

有自己的协议族

> unix域协议: AF_UNIX	/	AF_LOCAL

但是编程的流程和tcp/udp一样, 只不过"**网络地址**"不一样

unix域协议可以采取tcp, 也可以采取udp的方式, 但是一般建议采取udp



unix域协议网络地址

```c
#include<sys/un.h>
struct sockaddr_un
{
    sa_family_t sun_family;	//指定协议族
    char sun_path[104]; //unix域协议地址, 是一个本地文件系统中的一个绝对路径名, 如"/home/china/xxx"
}
```

练习:

完成unix域协议的基本功能函数 