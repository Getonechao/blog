+++
title= "python3:一些库的总结经验"
description= "文章简介"
date= 2023-04-18T11:41:19+08:00
author= "chao"
draft= false
image= "" 
math= true
categories= ["lang"]

tags=  [" "," "]

+++

# 一、OS模块

## 1.目录操作

~~~
os.chdir(path)
os.getcwd()
~~~

打开一个文件，并且设置需要的打开选项

~~~
os.open(file, flags)
~~~

取得指定文件夹下的文件列表

~~~
os.listdir(path)
~~~

创建一个名为 *path* 的目录，应用以数字表示的权限模式 *mode*

~~~
os.mkdir(path, mode=0o777, *, dir_fd=None)
os.remove(path, *, dir_fd=None)
os.rename(src, dst, *, src_dir_fd=None, dst_dir_fd=None)
~~~

递归目录创建函数。与 mkdir()类似，但会自动创建到达最后一级目录所需要的中间目录。

~~~
os.makedirs(name, mode=0o777, exist_ok=False)
os.removedirs(name)
os.renames(old, new)
~~~

创建一个名为 *path* 的 FIFO（命名管道，一种先进先出队列），具有以数字表示的权限状态 *mode*。

~~~
os.mkfifo(path, mode=0o666, *, dir_fd=None)
~~~

创建一个名为 *path* 的文件系统节点（文件，设备专用文件或命名管道）

~~~
os.mknod(path, mode=0o600, device=0, *, dir_fd=None)
~~~

提取主设备号，提取自原始设备号（通常是 `stat` 中的 `st_dev` 或 `st_rdev` 字段）

~~~
os.major(device, /) 
os.minor(device, /) 提取次设备号，提取自原始设备号（通常是 stat 中的 st_dev 或 st_rdev 字段）。
os.makedev(major, minor, /) 将主设备号和次设备号组合成原始设备号。
~~~



~~~
os.pathconf_names 字典，表示映射关系，为 pathconf() 和 fpathconf() 可接受名称与操作系统为这些名称定义的整数值之间的映射。
os.pathconf(path, name) 返回所给名称的文件有关的系统配置信息。name 指定要查找的配置名称，它可以是字符串，是一个系统已定义的名称，这些名称定义在不同标准（POSIX.1，Unix 95，Unix 98 等）中。一些平台还定义了额外的其他名称。当前操作系统已定义的名称在 pathconf_names 字典中给出。对于未包含在该映射中的配置名称，也可以传递一个整数作为 name。

# 获取文件最大连接数
no = os.fpathconf(fd, 'PC_LINK_MAX')
print "Maximum number of links to the file. :%d" % no

# 获取文件名最大长度
no = os.fpathconf(fd, 'PC_NAME_MAX')
print "Maximum length of a filename :%d" % no
~~~

返回一个字符串，为**符号链接指向的实际路径**。其结果可以是绝对或相对路径。如果是相对路径，则可用 **`os.path.join(os.path.dirname(path), result)`** 转换为绝对路径。

~~~
os.readlink(path, *, dir_fd=None)
~~~





## 2.文件操作

这些函数创建新的 file objects

~~~
os.fdopen(fd, *args, **kwargs)
~~~



## 3.进程参数



## 4. 文件描述符操作

打开文件 *path*，根据 *flags* 设置各种标志位，并根据 *mode* 设置其权限状态

~~~
os.open(path, flags, mode=0o777, , dir_fd=None)

os.O_RDONLY
os.O_WRONLY
os.O_RDWR
os.O_APPEND
os.O_CREAT
os.O_EXCL
os.O_TRUNC
上述常量在 Unix 和 Windows 上均可用。

os.O_DSYNC
os.O_RSYNC
os.O_SYNC
os.O_NDELAY
os.O_NONBLOCK
os.O_NOCTTY
os.O_CLOEXEC
这个常数仅在 Unix 系统中可用。

~~~

打开一对新的伪终端，返回一对文件描述符 `（主，从）`，分别为 pty 和 tty。

~~~
os.openpty()
~~~

创建一个管道，返回一对分别用于读取和写入的文件描述符 `(r, w)`

~~~
os.pipe()
~~~

读写

~~~
os.read(fd, n, /)
os.readv(fd, buffers, /)
os.write(fd, str, /)
os.writev(fd, buffers, /)
os.sendfile(out_fd, in_fd, offset, count)
os.sendfile(out_fd, in_fd, offset, count, headers=(), trailers=(), flags=0)
~~~

关闭文件描述符 *fd*。

~~~
os.close(fd)
**************
os.closerange(fd_low, fd_high, /) 关闭从 fd_low （包括）到 fd_high （排除）间的文件描述符，并忽略错误。
**************
os.copy_file_range(src, dst, count, offset_src=None, offset_dst=None) 从文件描述符 src 复制 count 字节，从偏移量 offset_src 开始读取，到文件描述符 dst，从偏移量 offset_dst 开始写入


备注：该功能适用于低级 I/O 操作，必须用于 os.open() 或 pipe() 返回的文件描述符。若要关闭由内建函数 open()、popen() 或 fdopen() 返回的 "文件对象"，则应使用其相应的 close() 方法。
~~~

获取文件描述符 *fd* 的状态

~~~
os.fstat(fd)
~~~

如果文件描述符 *fd* 打开且已连接至 tty 设备（或类 tty 设备），返回 `True`，否则返回 `False`

~~~
os.isatty(fd, /)
~~~







## 5. Linux 扩展属性



## 6.进程管理



## 7.调度器接口

**控制操作系统如何为进程分配 CPU 时间**

~~~
***********调度策略常量**************
os.SCHED_OTHER 默认调度策略
os.SCHED_BATCH 用于 CPU 密集型进程的调度策略，它会尽量为计算机中的其余任务保留交互性
os.SCHED_IDLE 用于极低优先级的后台任务的调度策略
os.SCHED_SPORADIC 用于偶发型服务程序的调度策略
os.SCHED_FIFO 先进先出的调度策略
os.SCHED_RR 循环式的调度策略
os.SCHED_RESET_ON_FORK 此旗标可与任何其他调度策略进行 OR 运算。 当带有此旗标的进程设置分叉时，其子进程的调度策略和优先级会被重置为默认值。
**********************************
class os.sched_param(sched_priority) 
os.sched_get_priority_min(policy) 获取 policy 的最低优先级数值。 policy 是以上调度策略常量之一
os.sched_get_priority_max(policy) 获取 policy 的最高优先级数值。 policy 是以上调度策略常量之一

eg:
import os
param = os.sched_param(os.sched_get_priority_max(os.SCHED_FIFO))
os.sched_setscheduler(0, os.SCHED_FIFO, param)
~~~

~~~
os.sched_setscheduler(pid, policy, param) 根据进程的 PID pid 设置其调度策略。pid 为 0 指的是调用本方法的进程；policy 是以上调度策略常量之一；param 是一个 sched_param 实例。

os.sched_getscheduler(pid) 返回 PID 为 pid 的进程的调度策略。pid 为 0 指的是调用本方法的进程。返回的结果是以上调度策略常量之一。 

os.sched_setparam(pid, param) 设置 PID 为 pid 的进程的某个调度参数。pid 为 0 指的是调用本方法的进程。param 是一个 sched_param 实例

os.sched_getparam(pid) 返回 PID 为 pid 的进程的调度参数为一个 sched_param 实例。pid 为 0 指的是调用本方法的进程。

os.sched_rr_get_interval(pid) 返回 PID 为 pid 的进程在时间片轮转调度下的时间片长度（单位为秒）。pid 为 0 指的是调用本方法的进程。

os.sched_yield() 自愿放弃 CPU。

os.sched_setaffinity(pid, mask) 将 PID 为 pid 的进程（为零则为当前进程）限制到一组 CPU 上。mask 是整数的可迭代对象，表示应将进程限制在其中的一组 CPU。

os.sched_getaffinity(pid) 返回 PID 为 pid 的进程（为零则为当前进程）被限制到的那一组 CPU
~~~



## 8.其他系统信息

~~~
os.cpu_count() cpu核数
os.getloadavg() 返回系统运行队列中最近 1、5 和 15 分钟内的平均进程数。
***********************************************************
os.confstr_names 字典，表示映射关系，为 confstr() 可接受名称与操作系统为这些名称定义的整数值之间的映射
os.confstr(name) 
***********************************************************
os.sysconf_names 字典，表示映射关系，为 os.sysconf() 可接受名称与操作系统为这些名称定义的整数值之间的映射。这可用于判断系统已定义了哪些名称。
os.sysconf(name) 
***********************************************************
os.curdir .
os.pardir ..
os.sep    /
os.extsep . "分隔基本文件名与扩展名的字符 eg test.txt"
os.pathsep : 操作系统通常用于分隔搜索路径（如 PATH）中不同部分的字符，如 POSIX 上是 ':'，Windows 上是 ';'
os.devnull  空设备的文件路径。如 POSIX 上为 '/dev/null'，Windows 上为 'nul'
~~~



## 9.随机数

## 10.一些变量

~~~
1. os.environ
一个表示字符串环境的mapping对象，访问方式：os.environ['HOME']

2. 
~~~

## 11.OS.path路径操作

~~~
os.path.abspath(path)	返回绝对路径
os.path.basename(path)	返回文件名
os.path.dirname(path)	返回文件路径
s.path.exists(path)	路径存在则返回True,路径损坏返回False
os.path.getatime(path)	返回最近访问时间（浮点型秒数）
os.path.getmtime(path)	返回最近文件修改时间
os.path.getctime(path)	返回文件 path 创建时间
os.path.isabs(path)	判断是否为绝对路径
os.path.isfile(path)	判断路径是否为文件
os.path.isdir(path)	判断路径是否为目录
os.path.join(path1[, path2[, ...]])	把目录和文件名合成一个路径
os.path.normcase(path)	转换path的大小写和斜杠
os.path.realpath(path)	返回path的真实路径
os.path.samefile(path1, path2)	判断目录或文件是否相同
os.path.sameopenfile(fp1, fp2)	判断fp1和fp2是否指向同一文件
os.path.split(path)	把路径分割成 dirname 和 basename，返回一个元组
os.path.splitext(path)	分割路径中的文件名与拓展名
os.path.walk(path, visit, arg)	遍历path，进入每个目录都调用visit函数，visit函数必须有3个参数(arg, dirname, names)，dirname表示当前目录的目录名，names代表当前目录下的所有文件名，args则为walk的第三个参数
os.path.sep	获取当前系统路径分隔符
~~~





# 二、SYS模块