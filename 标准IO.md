# 标准IO

## 1. 标准IO？

每个操作系统下面，对文件的管理和提供的接口不一样

**linux系统**： open/read/....
**windows系统**： WinOpen....
同一个文件，在不同的操作系统下面。我们操作文件的代码都不一样！！

C语言标准委员会，就觉得应该由它制定一个统一的标准，用来统一操作文件的接口

**C语言 标准IO**

在标准IO库中，用结构体 FILE来表示一个文件，这个结构体中，有两个缓冲区，一个读缓冲区，一个写缓冲区

```c
	FILE
	{
		char * in;//指向读的缓冲区
		char * out;//指向写的缓冲区
		......
	};
```

还提供了对文件的 操作的接口

`fopen/fclose/fread/......`

标准IO带缓冲的IO，IO流，它的效率要比系统IO高，因为带缓冲

> **系统IO:**
> 			**read** 1字节		从磁盘中读一个字节出来
>
> **标准IO:**
> 			**fread** 1字节		从磁盘中读一块（如512字节）出来，
> 			放在 标准IO的缓冲区中

缓冲区开多大呢？	标准Io缓冲区类型：

1. **行缓冲**：缓冲区数据到达一行，同步到硬件中去

   假设一行最多是100个字节，那么这个缓冲区的数据如果到达100个字节 或者 遇到'\n',它就会把缓冲区的数据同步到硬件上去。

   **printf -> 行缓冲**

   printf 的作用就是**把字符串数据输出到标准输出设备**（打印出来）

2. **全缓冲**：

   数据要全部填满缓冲区，才会同步到硬件中去

3. **无缓冲**：

   缓冲区有几个字节就马上同步几个字节
   **perror**

```c
extern struct _IO_FILE *stdin;		/* Standard input stream.  */
extern struct _IO_FILE *stdout;		/* Standard output stream.  */
extern struct _IO_FILE *stderr;		/* Standard error output 
```

标准IO库，会为每个进程自动打开三个标准IO流（文件）

**标准输入**： FILE * stdin;

​	是定义在 stdio.h 中的一个全局变量,它指向标准输入设备

**标准输出**：FILE *stdout;

​	它指向标准输出设备（一般是终端）

**标准出错**：FILE *stderr;

​	它指向标准输出设备（一般是终端）

***

## 2. 标准IO的函数

### fopen() 打开一个文件

```c
FILE *fopen(const char *pathname, const char *mode);
			pathname：文件的路径名
			mode：打开方式
				"r" :只读打开。
					打开后，光标在开头，如果文件不存在则报错
				"r+":读写打开。
					打开后，光标在开头，如果文件不存在则报错
				"w"：只写打开。
					打开后，文件内容被截短（清空），如果文件不存
					在，则创建
				"w+":读写打开。
					打开后，文件内容被截短（清空）
					，如果文件不存在，则创建
		
				"a" :append 追加打开，文件不存在，则创建。
					打开后光标在末尾
					
				"a+" 追加打开，文件不存在，则创建。
					打开后，原始读的光标在开头，原始写的光标在末尾
		返回值：
			成功返回打开的文件的 FILE指针，后续标准IO库的函数
				都需要利用这个指针来操作该文件
			失败返回NULL，同时errno被设置
```

### 读文件流

#### 每次读一个字节

```c
fgetc/getc/getchar
    
#include <stdio.h>
int fgetc(FILE *stream);
stream：FILE * ，指定要从哪个文件中读取数据
    成功返回 读到的数据的值
    失败返回 -1

int getc(FILE *stream);
与fgetc一样，只不过他们的内部实现原理不一样
getc可能是一个宏
int getchar(void);
没有参数，那么从哪个文件读？？？？
    读取标准输入设备的一个字节
```

#### 每次读一行

```c
gets/fgets
char *fgets(char *s, int size, FILE *stream);
	s:指向的内存用来保存读取到的数据
	size:你想读取多少个字节的数据
		fgets读取结束有两种情况
		(1) 遇到 '\n'或者文件结束
			如果在达到字节数目最大之前读完一行
			他将在字符串的'\0'之前加一个换行符表示一行结束
		(2) 已经读了size-1个字节(最后一个字节 '\0')
			就算这一行没读完，也结束
			
			
思考：
	fgets(buf,20,stdin);//获取标准输入设备的20个字符
	->
	gets(buf);//不建议使用
```

#### 直接读

您想读多少个字节就读多少个字节

```c
	fread
	#include <stdio.h>
size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream);
		ptr:指向一段空间，这段空间用来保存读取到的数据
		size：表示单个数据占多少字节
		nmemb：表示你要读取多少个数据
			-》要读取的总字节数目： size*nmemb
		
		stream：文件FILE指针，表示从哪个文件读
	返回值：
		成功返回读取到的 数据个数
		失败返回-1，同时errno被设置
```

### 写文件流

#### 每次写入一个字节

```c
putc/fputc/putchar
#include <stdio.h>
int fputc(int c, FILE *stream);
int putc(int c, FILE *stream);
	fputc和putc的功能和用法完全一致，只不过putc可能是用宏实现的
	c:事先保存好需要写入的字符
	stream：指定要写入到哪个文件流中去
	返回值：
		成功返回写入的字符的ASCII码
		失败返回-1，同时errno被设置
int putchar(int c);
	少了一个参数，没有指定要写入到哪里。
	是因为这个函数默认把字符写入到标准输出设备
	
	putchar('A'); <-> putc('A',stdout);
```

#### 每次写一行

```c
puts/fputs
int puts(const char *s);
	s：指向一段内存空间，该内存中事先保存了要写的数据
	没有指定写入到哪个文件，默认写入到标准输出文件
int fputs(const char *s, FILE *stream);
	s: 指向一段内存空间，该内存中事先保存了要写的数据
	stream：指定要写入到哪个文件流中去
```

#### 直接写入

```c
fwrite
size_t fwrite(const void *ptr, size_t size, size_t nmemb,
         FILE *stream);
	ptr:指向要写入进文件的内容
	size:要写入的内容单个数据的大小
	nmemb:代表你想写入多少个数据
			写入的总字节数目 size*nmemb
	stream:指定要写到哪个文件流中去
	返回值：
		成功返回实际写入的数据个数
		失败返回-1，同时errno被设置
```

### 冲洗文件流 fflush

```c
#include <stdio.h>
int fflush(FILE *stream);
对于输出流，fflush() 把写缓冲区的内容 写/更新 到硬件中去
对于输入流，一般来讲是直接把读缓冲区的数据丢弃，但是很多标准
	中并没有定义该行为
	stream:指定冲洗哪个文件流
```

### 定位文件流 -> 光标

```c
#include <stdio.h>
int fseek(FILE *stream, long offset, int whence);
	stream:指定定位哪个文件流
	offset:偏移量
	whence:定位方式
		offset和whence 和lseek中的含义一致
		
	返回值：
		成功返回0
		失败返回-1
	
long ftell(FILE *stream);
	用来返回当前光标距离文件开头的字节数目
		
		
求文件大小
int size = lseek(fd,0,SEEK_END);
->
fseek(fp,0,SEEK_END);
int size = ftell(fp);	
```

### 文件结束

```c
feof 用来判断 stream指向的文件流是否结束
	int feof(FILE *stream);
		返回值：
			返回0 代表文件流没有结束
			返回非0 代表文件结束了
			
	标准IO库，在读到文件流末尾时，会往读缓冲区填入一个特殊字符 EOF(二进制11111111)
		即使是一个空文件，文件流中也会有一个字符 EOF
```

### 练习

利用标准IO实现文件的复制粘贴

```c
int main()
{
	FILE * fp1 = fopen("file1","r");
	....
	FILE * fp2 = fopen("file2","w");
	....
	
	char buf[1000];
	int r;
	while(!feof(fp1))
	{
		r = fread(buf,1,100,fp1);
		fwrite(buf,1,r,fp2);
		//fgets(buf,1000,fp1);
		//fputs(buf,fp2);
	}
	
	fclose(fp1);
	fclose(fp2);
}
```

### 格式化IO

#### 格式化输入

按照事先指定的格式进行输入，如果不遵守这个格式，就会出错

```c
int a,b;
scanf("%d%d",&a,&b); //100 200
```

```c
#include <stdio.h>
int scanf(const char *format, ...);//参数个数不确定，但是至少要有一个参数
```

分为两类

**第一类**参数就是第一个参数

**format**：指定输入格式  

​		是用来告诉用户怎么输入的，意思是说必须得按照这个字符串指定的格式进行输入，否则出错。format字符串中有三类字符串
​		a,一般字符串->精确的匹配
​			怎么写，用户就得怎么输入
​		b,空白符(空格，tab，回车键...) -》指示用户输入1个或多个空白符
​			（有些特殊情况甚至可以不要）

​		c,转义字符(以%开头)，有特殊含义
​		%d ->输入整数
​		%c ->输入字符
​		....

**第二类**参数就是 某某 地址 列表
	代表把输入的数据依次存到该地址列表中去
			
返回值：
	成功返回 成功匹配的输入项数
	失败返回-1

`int fscanf(FILE *stream, const char *format, ...);`
	`fscanf`和`scanf`的功能类似，只不过`fscanf`的数据来源是 一个指定的文件流
	而`scanf`的数据来源是 标准输入流
	所以 `fscanf`比`scanf`多了一个参数 stream ->用来指定文件流
	
`int sscanf(const char *str, const char *format, ...);`
	`fscanf`和`scanf`的功能类似，只不过`sscanf`的数据来源是 一块 内存空间
	所以 `sscanf`比`scanf`多了一个参数 `str`->用来指定一块内存

#### 格式化输出

`printf("请输入:%d\n",a);`

```c
#include <stdio.h>
int printf(const char *format, ...);
	printf是把字符串数据输出到 标准输出设备
		
int fprintf(FILE *stream, const char *format, ...);
	fprintf和printf的功能，用法类似
	比printf多了一个参数 stream，指定输出到哪个文件流
	
	FILE * fp = fopen("1.txt","w");
	
	fprintf(fp,"请输入:%d\n",a);
int dprintf(int fd, const char *format, ...);
	和 fprintf功能完全一样，只不过fprintf是用 FILE 指针来指定文件
			dprintf是用 文件描述符 fd 来指定文件
int sprintf(char *str, const char *format, ...);
	sprintf和printf的功能，用法类似 ，是把字符串数据输出到一段内存
	比printf多了一个参数 str,用于指定 一段内存空间
	sprintf有一个bug 你懂得
	所以有了改进版 snprintf
	
int snprintf(char *str, size_t size, const char *format, ...);
	加了一个size参数，用来指定 str内存的最大字节数目
```

***

## 作业

用标准IO函数，来实现一个简单的 学生成绩管理系统

 ```c
struct student
 {
	int num;//学号 是唯一的
	char name[100];
	int score;
 };
 ```

实现基本的 增删改查
			
思路：
	创建一个文件，用来代替数据库保存 数据
			

