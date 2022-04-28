+++
title= "Tcp_server"
description= "利用linux epoll实现一个tcp server服务器"
date= 2022-04-27T09:17:43+08:00
author= "chao"
draft= false
image= "" 
math= true
categories= [
    "lang"
]

tags=  [
    " c","linux ","project","net"
]

+++

# TCP server

## epoll

### 头文件

~~~c
#include <sys/epoll.h>
~~~

### 结构体

~~~c
typedef union epoll_data
{
  void *ptr;
  int fd;
  uint32_t u32;
  uint64_t u64;
} epoll_data_t;

struct epoll_event
{
  uint32_t events;	/* Epoll events 即enum EPOLL_EVENTS*/
  epoll_data_t data;	/* User data variable */
} __EPOLL_PACKED;
~~~

### 宏

~~~c
enum EPOLL_EVENTS
  {
    EPOLLIN = 0x001,//表示对应的文件描述符可以读
#define EPOLLIN EPOLLIN
    EPOLLPRI = 0x002,//表示对应的文件描述符有紧急的数据可读
#define EPOLLPRI EPOLLPRI
    EPOLLOUT = 0x004,//表示对应的文件描述符可以写
#define EPOLLOUT EPOLLOUT
    EPOLLRDNORM = 0x040,
#define EPOLLRDNORM EPOLLRDNORM
    EPOLLRDBAND = 0x080,
#define EPOLLRDBAND EPOLLRDBAND
    EPOLLWRNORM = 0x100,
#define EPOLLWRNORM EPOLLWRNORM
    EPOLLWRBAND = 0x200,
#define EPOLLWRBAND EPOLLWRBAND
    EPOLLMSG = 0x400,
#define EPOLLMSG EPOLLMSG
    EPOLLERR = 0x008,//表示对应的文件描述符发生错误
#define EPOLLERR EPOLLERR
    EPOLLHUP = 0x010,//表示对应的文件描述符被挂断
#define EPOLLHUP EPOLLHUP
    EPOLLRDHUP = 0x2000,
#define EPOLLRDHUP EPOLLRDHUP
    EPOLLEXCLUSIVE = 1u << 28,
#define EPOLLEXCLUSIVE EPOLLEXCLUSIVE
    EPOLLWAKEUP = 1u << 29,
#define EPOLLWAKEUP EPOLLWAKEUP
    EPOLLONESHOT = 1u << 30,//只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个socket的话，需要再次把这个socket加入到EPOLL队列里。
#define EPOLLONESHOT EPOLLONESHOT
    EPOLLET = 1u << 31//将EPOLL设为边缘触发(Edge Triggered)模式
#define EPOLLET EPOLLET
  };


/* Valid opcodes ( "op" parameter ) to issue to epoll_ctl().  */
#define EPOLL_CTL_ADD 1	/* Add a file descriptor to the interface.  */
#define EPOLL_CTL_DEL 2	/* Remove a file descriptor from the interface.  */
#define EPOLL_CTL_MOD 3	/* Change file descriptor epoll_event structure.  */
~~~



### API

###### epoll_create()函数

功能：创建一个epoll实例。

返回值：如果成功，返回一个epoll 的句柄fd（note：epoll占用一个fd，发生错误时，返回-1，

注意：从Linux 2.6.8开始，size参数被忽略，但必须大于零； 

~~~c
#include <sys/epoll.h>
int epoll_create (int __size)
~~~

&emsp;&emsp;这个文件描述符用于所有后续调用epoll的所有接口。 当不再需要文件描述符时，使用close关闭。 当所有指向这个epoll实例的文件描述符都关闭时，内核销毁实例并释放关联的重用资源

###### epoll_create1()函数

功能：创建一个epoll实例。 如果flags为0，epoll_create1（）和删除了过时size参数的epoll_create（）相同。

~~~c
#include <sys/epoll.h>
int epoll_create1 (int __flags)
~~~

###### epoll_ctl()函数

功能：该系统调用对文件描述符epfd引用的epoll实例执行控制操作。

~~~c
#include <sys/epoll.h>
int epoll_ctl (int __epfd, int __op, int __fd,struct epoll_event *__event) 
~~~

op: EPOLL_CTL_ADD、EPOLL_CTL_DEL、EPOLL_CTL_MOD

###### epoll_wait()函数

~~~c
#include <sys/epoll.h>
int epoll_wait (int __epfd, struct epoll_event *__events,int __maxevents, int __timeout)
~~~

###### epoll_pwait()函数

~~~c
#include <sys/epoll.h> 
int epoll_pwait (int __epfd, struct epoll_event *__events,int __maxevents, int __timeout,
			const __sigset_t *__ss);
~~~



