+++

title= "实验10C_C++预编译头文件配置"
description= "蓝桥网课xmake笔记"
date= 2021-07-15T10:36:46+08:00
author= "chao"
draft= false
image= "" 
math= true
categories= [
    "tools"
]

tags=  [
    "xmake"
]

+++

# C_C++预编译头文件配置

在本节实验中，我们会学习到 C/C++ 预编译头文件的一些基本概念，了解如何在 xmake 中配置开启头文件预编译，以及会做一些对比测试来直观感受预编译头文件对编译性能的影响。

#### 知识点

- C/C++ 预编译头文件的基础概念
- gcc 编译器对预编译的处理细节
- 预编译头文件对编译速度的影响比对
- xmake 中如何配置开启头文件预编译



## 预编译头文件介绍

在编译器编译每个 C++ 代码的时候，都需要额外处理一遍所有被引入的头文件，而 C++ 头文件的引入就会带来大量的类型定义，C++ 模板类定义等，都会导致源文件的编译速度非常的慢。

如果我们使用过 boost、std 等 C++ 库，就应该知道，这些库大部分类定义都在头文件中，而且大量使用了 C++ 模板特性，引入一个头文件就会使得整体编译慢很多，尤其是臃肿的 boost 库。

因此，为了优化编译速度，其实我们不需要编译每个源文件都去重复预处理一遍里面的头文件，我们可以把大部分项目常用的头文件放置在一个统一的头文件中，比如：stdafx.h （如果使用 vs 开发过 C++ 程序，应该经常能看到这个文件），然后让编译器先预先编译掉这个头文件，然后其它所有代码使用的时候，就不再需要额外引入编译了，在链接阶段直接复用就行了。

这样就将之前的 N 次编译，减少为仅仅一次头文件编译，极大地提升了编译效率，减少大量的头文件冗余编译，这就是整个预编译头文件的作用和处理过程。而且目前大部分 C/C++ 主流编译器，例如：gcc、clang、msvc 等，都是完全支持通过预编译头文件来优化 C/C++ 代码的编译速度。

关于预编译头文件处理的大概机制可以参考下图。

![1](实验10C_C++预编译头文件配置.assets/bfe9439264336bfe456c57080718a6de-0)

#### 配置使用 C++ 预编译头文件

在开始实验如何使用 C++ 预编译头文件之前，先来做一些准备工作，通过如下命令创建一个空的 C++ 工程。

```bash
cd Code
xmake create pcxxheader
```

然后进入 pcxxheader 文件夹，创建并编辑 `src/stdafx.h` 头文件，引入一些常用的 C++ 头文件。

```c++
#ifndef HEADER_H
#define HEADER_H

#include <algorithm>
#include <deque>
#include <iostream>
#include <map>
#include <memory>
#include <set>
#include <utility>
#include <vector>
#include <string>
#include <queue>
#include <cstdlib>
#include <utility>
#include <exception>
#include <list>
#include <stack>
#include <complex>
#include <fstream>
#include <cstdio>
#include <iomanip>

#endif
```

预编译头文件的名字并不一定非得是 stdafx.h，可以取任意名字，这里仅仅只是沿用了 vs 项目里面的命名而已，另外我们看到上面的代码中，引入了很多的头文件，这主要是为了之后便于对比开启预编译后的编译速度。

创建好预编译头文件后，我们可以创建一些引用了这个头文件的 C++ 源文件，比如创建并编辑 `src/test1.cpp` 文件，添加如下代码。

```c++
#include "stdafx.h"
```

当然，仅仅创建一个，后面测试的时候，也许效果不会太明显，我们可以再多创建一些同样的 C++ 文件，可以执行下面的命令。

```bash
cp src/test1.cpp src/test2.cpp
cp src/test1.cpp src/test3.cpp
cp src/test1.cpp src/test4.cpp
cp src/test1.cpp src/test5.cpp
cp src/test1.cpp src/test6.cpp
cp src/test1.cpp src/test7.cpp
cp src/test1.cpp src/test8.cpp
cp src/test1.cpp src/test9.cpp
cp src/test1.cpp src/test10.cpp
cp src/test1.cpp src/test11.cpp
cp src/test1.cpp src/test12.cpp
cp src/test1.cpp src/test13.cpp
cp src/test1.cpp src/test14.cpp
cp src/test1.cpp src/test15.cpp
cp src/test1.cpp src/test16.cpp
```

然后就可以直接编译我们的项目了，因为自动创建的工程在 `xmake.lua` 里面配置了 `add_files("src/*.cpp")` 会自动匹配引入所有创建的 C++ 代码，所以并不需要做什么配置改动，就可以编译通过。

不过现在我们还没有开启预编译配置，仅仅只是普通的头文件引用和编译。执行如下命令，并且获取正常编译情况下的总编译耗时。

```bash
time xmake -r -j1
```

这里，我们设置 `-j1` 开启单任务编译，主要是为了避免 xmake 默认的并行编译优化的干扰。

下图是正常编译时候的耗时情况，从红框中可以看出，大概编译耗时 4.672s。

![2](实验10C_C++预编译头文件配置.assets/be03ae4308e625557c8829492be2bb7c-0)

然后我们编辑 `xmake.lua` 文件，添加 `set_pcxxheader("src/stdafx.h")` 的预编译设置，告诉编译器将 stdafx.h 作为预编译头文件，开启预编译模式。

```lua
target("pcxxheader")
    set_kind("binary")
    add_files("src/*.cpp")
    set_pcxxheader("src/stdafx.h")
```

然后重新执行刚刚的编译命令，统计下编译耗时。

```bash
time xmake -r -j1
```

这次的编译，我们会明显感觉变快了很多，从下图的红框中看到，整体编译耗时降到了 1.593s，编译速度快了将近 3 倍，如果参与编译的源文件更多的话，效果会更加明显。

![3](实验10C_C++预编译头文件配置.assets/b854a1e212b974f68a1baead2052aee4-0)



## 不同编译器对预编译的处理

从刚刚的对比实验中，我们发现采用预编译模式，整个项目确实会快不少。现在我们从编译器层面简单了解下，gcc 等编译器到底是怎么处理预编译文件的。

我们可以通过 `xmake -rv` 命令，查看完整的编译输出来分析传入编译器的参数选项，输出内容如下图。

![4](实验10C_C++预编译头文件配置.assets/a66f2c9fb2f0e20591684d0ab8fbd73e-0)

从上图可知，xmake 会优先调用 gcc 去编译 stdafx.h 头文件，生成 stdafx.pch 命名的预编译头文件。然后，后续的其它源文件编译通过 `-include stdafx.h` 将其引入进来。

大致的流程如下。

```bash
gcc -c -x c++-header -o src/stdafx.pch src/stdafx.h
gcc -c -include src/stdafx.h -o build/test.o src/test1.cpp
```

我们也可以手动执行上面的命令，尝试直接调用 gcc 来处理头文件预编译，这里额外传递了 `-x c++-header` 编译选项，是为了告诉 gcc 编译器，`src/stdafx.h` 是 C++ 头文件，应该使用 C++ 编译器来编译，而不是使用 C 编译器。

如果执行过程中没有报任何错误，那就说明执行成功了。

记住，第一条 stdafx.pch 的编译命令只需要执行一遍，之后就可以通过 `-include src/stdafx.h` 来查找对应的 pch 文件，直接使用预编译后的结果了。

gcc 的预编译头处理方式其实跟 clang 的基本类似，唯一的区别就是：它不支持 `-include-pch` 参数，因此不能直接指定使用的 stdafx.pch 文件路径，但是会有一些搜索规则来查找对应的 pch 文件。

1. 从 stdafx.h 所在目录中，查找 stdafx.pch 文件是否存在。
2. 从 -I 指定的头文件搜索路径中查找 stdafx.pch。

#### 配置使用 C 预编译头文件

其实，除了 C++ 项目可以支持头文件预编译，C 项目也是同样支持的，只不过 C 代码没有 C++ 模板这种复杂的特性，编译原本就很快，因此是否开启预编译，其优化效果没有 C++ 代码那么明显。

不过，xmake 还是对 C 项目的预编译同时做了支持，相关的预编译配置接口只需要从 `set_pcxxheader` 改成 `set_pcheader` 就可以了，其他配置上没啥变化。

下面，我们重新创建一个基于 C 的空工程来测试验证 C 代码的头文件预编译效果。

```bash
cd Code
xmake create -l c pcheader
```

然后进入 pcheader 文件夹，创建并编辑 `src/stdafx.h` 文件，引入一些常用系统头文件。

```c
#ifndef HEADER_H
#define HEADER_H

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdint.h>
#include <pthread.h>

#endif
```

接下来，我们继续创建一些使用了这个 stdafx.h 头文件的 C 代码，比如 `src/test1.c`，内容跟之前 C++ 中的类似。

```c++
#include "stdafx.h"
```

我们可以使用下面的命令多创建些代码文件。

```bash
cp src/test1.c src/test2.c
cp src/test1.c src/test3.c
cp src/test1.c src/test4.c
cp src/test1.c src/test5.c
cp src/test1.c src/test6.c
cp src/test1.c src/test7.c
cp src/test1.c src/test8.c
cp src/test1.c src/test9.c
cp src/test1.c src/test10.c
cp src/test1.c src/test11.c
cp src/test1.c src/test12.c
cp src/test1.c src/test13.c
cp src/test1.c src/test14.c
cp src/test1.c src/test15.c
cp src/test1.c src/test16.c
```

然后可以跟之前一样编译测试对比下编译效率。

继续执行 `time xmake -r -j1` 查看编译耗时，从下图红框中可以看到总耗时 1.211s，说明即使没开预编译，编译速度也已经很快了。

![5](实验10C_C++预编译头文件配置.assets/eb698b8f06226e0557a9c6120f9944cb-0)

然后，我们通过 `set_pcheader` 配置接口加上头文件预编译设置，修改 xmake.lua 内容如下。

```lua
target("pcheader")
    set_kind("binary")
    add_files("src/*.c")
    set_pcheader("src/stdafx.h")
```

我们再重新执行 `time xmake -r -j1` 查看编译耗时：0.819s，说明还是快了一点的，但是优势不是非常明显。

![6](实验10C_C++预编译头文件配置.assets/12b562a723913d1f6f8a1d6d61f14a2a-0)



## 实验总结



在本节实验中，我们了解了什么是预编译头文件，学习了如何配置编译头文件来优化 C++ 代码的编译速度，并且知道了 C 代码也是可以支持预编译头文件的，但是整体编译速度不会像 C++ 那样提升这么明显，因此针对 C++ 项目来说，我们使用预编译头文件还是可以改善不少项目的编译效率。

本实验的参考代码可以使用如下命令下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/2764/code10.zip
```