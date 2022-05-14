+++

title= "实验17工程生成插件使用"
description= "蓝桥网课xmake笔记"
date= 2021-07-15T10:37:05+08:00
author= "chao"
draft= false
image= "" 
math= true
categories= [
    "app"
]

tags=  [
    "xmake"
]

+++

# 工程生成插件使用

在本节实验中，我们主要学习如何使用 `xmake project` 工程生成插件去生成各种第三方的工程文件，例如：Makefile、CMakeLists.txt 和 build.ninja 文件等，以及操作和体验实际 xmake 项目工程的编译。

#### 知识点

- 工程生成插件介绍
- Makefile 文件的生成
- CMakeLists.txt 文件的生成
- build.ninja 文件的生成

## 工程生成插件介绍

xmake 除了可以直接构建工程外，还提供了 `xmake project` 插件命令，使得可以像 cmake 那样生成第三方工程文件，例如生成 vs 工程、makefile 文件以及 CMakeLists.txt 等等。

这对于一些需要借助像 vs 此类的第三方 IDE 程序来编译调试程序的时候，会比较有用，也可以用于提供项目给一些没有安装 xmake 又想要编译的用户。

下面我们来逐一讲解各种工程文件的生成和使用。

#### 生成 Makefile 文件

我们可以通过 xmake 生成 Makefile 文件来使用 make 命令编译项目，生成 Makefile 文件后，我们就可以完全不依赖 xmake，只要用户的系统上有 make 命令，就可以完成编译。

为了实验如何生成 Makefile 文件，首先执行下面的命令创建一个空工程。

```bash
cd ~/Code
xmake create protest
```

然后我们进入 protest 文件夹，编辑 xmake.lua 文件改成如下配置，并在其中设置上一些简单编译配置，比如添加 `-DTEST` 宏定义，以及添加一些链接库。

```lua
target("protest")
    set_kind("binary")
    add_files("src/*.cpp")
    add_defines("TEST")
    add_syslinks("pthread", "m", "dl")
```

接着，执行下面的命令开始生成对应的 Makefile 文件。

```bash
xmake project -k makefile
```

`xmake project` 就是 xmake 提供的工程插件生成命令，而我们只需要通过 `-k` 参数设置工程类型为 `makefile` 就可以生成 Makefile 文件了。

运行上述命令后，可以看到当前项目根目录下新增了一个 Makefile 文件，如下图。

![1](实验17工程生成插件使用.assets/057057fd50e9cb5354974d1cc8a6a597-0)

我们可以执行 `gvim ./makefile` 看下实际生成的 makefile 文件内容。

![2](实验17工程生成插件使用.assets/b50509d5dba3498c24146ab41cf8ae27-0)

其中，最开头部分是对编译工具链的配置定义，然后是配置编译链接选项的定义，也就是图中红线部分，我们看到之前在 xmake.lua 配置的 `-DTEST` 宏定义以及 pthread 等链接库信息都成功配置进去了。

在最下面，就是 make 对 c++ 源文件的编译和目标程序的链接配置了，整体结构还是比较清晰明了的。接下来，我们直接使用 make 命令而不是 xmake 命令来编译下当前工程。

```bash
make
```

如果编译正常，会看到下图的输出结果。

![3](实验17工程生成插件使用.assets/056500dc07f539fdf7a5a1cb6ee5ac73-0)

编译完成后，我们再来执行 `tree` 命令，查看实际通过 `make` 命令编译生成的可执行路径在哪里，从下图中，可以确认实际的输出路径跟通过 xmake 命令编译的结果完全一致。

![4](实验17工程生成插件使用.assets/88d778f689c5bdd2959e9ab8b7519acb-0)

既然知道了生成的可执行文件路径，那么我们直接执行下它看看，是否能够正常运行。

```bash
./build/linux/x86_64/release/protest
hello world!
```

如果一切运行正常，我们会看到下面的运行输出结果。

![5](实验17工程生成插件使用.assets/61568859da45670a876b96948ba77499-0)

#### 生成 CMakeLists.txt

xmake 除了能够生成 Makefile 文件，还可以生成 CMakeLists.txt 文件（这是 cmake 专属的工程描述文件），使用户能够通过 cmake 来编译项目。虽然 cmake 也只不过是工程文件生成器，还是需要生成 Makefile 等文件才能继续编译。

这样感觉似乎有点多此一举，中间还多饶了一下，其实大部分情况下是这样的，但是毕竟很多 IDE 程序对 cmake 的支持力度会更好些，比如 Android Studio 下对 cmake 程序的调试支持会更加的完善。因此在某些特定场景，通过生成 CMakeLists.txt 还是可以变相解决不少问题的，也方便一些习惯使用 cmake 的用户来编译项目。

为了能够生成 CMakeLists.txt，其实我们不需要做什么改动，只需要执行下面的命令即可生成。

```bash
xmake project -k cmake
```

如果生成成功，我们会看到项目根目录下的 CMakeLists.txt 文件，然后我们执行 `gvim CMakeLists.txt` 看下里面的内容。

![6](实验17工程生成插件使用.assets/cd373694590040624d59d306f5d942ef-0)

从上图可以看出，红框部分就是在 xmake.lua 配置中加入的宏定义和链接选项配置，这些也已经被正常加入到了 CMakeLists.txt 文件中。

在尝试使用 cmake 来处理生成的 CMakeLists.txt 之前，我们需要先执行 `cmake --version`，确认 cmake 已经被安装到实验环境，如果还没有安装，可以执行下面的命令安装下。

```bash
sudo apt update
sudo apt install cmake
```

如果已经安装完成，就可以使用 cmake 来编译项目了，因为我们已经通过 xmake 生成了 CMakeLists.txt 文件，因此只需要执行下面的命令进入 build 目录后生成 makefile 文件，然后执行 make 来编译即可。

```bash
cd build
cmake .. # 调用 cmake 去生成 Makefile 文件
make
```

运行结果如下图，也正常完成了编译。

![图片描述](实验17工程生成插件使用.assets/056ca5da5fd4f72f272ff2c19f0a2bc2-0)

#### 生成 build.ninja

xmake 除了可以生成 Makefile 和 CMakeLists.txt 以外，还可以直接生成 build.ninja 文件，支持 Ninja 来编译项目。Ninja 是一个比较轻量并且主打编译效率的构建工具，特点是编译非常的快（当然，xmake 直接编译也是极快的，两者速度上差异不大）。

为了使用 Ninja，执行如下命令去安装它。

```bash
wget https://github.com/ninja-build/ninja/releases/download/v1.10.1/ninja-linux.zip
unzip ./ninja-linux.zip
sudo cp ./ninja /usr/local/bin/ninja
```

安装完成后，通过 `xmake project` 插件生成对应的 build.ninja 文件，只需要执行下面的命令。

```bash
cd ~/Code/protest
xmake project -k ninja
```

由于生成的 build.ninja 内容比较多，这里就不展示了，只要我们确认当前工程根目录下确实生成了 build.ninja 文件，就可以执行下面的命令去调用 ninja 编译了。

```bash
ninja
```

编译和运行效果如下图。

![8](实验17工程生成插件使用.assets/24ee2fad14c0db85d0e6b63eea76b2f8-0)

#### 实战之 tbox 工程编译

由于之前的测试工程仅仅只有一个简单 C++ 文件，不管是用 xmake 直接编译，还是生成 cmake 和 ninja 工程文件来编译，都不太能直观的感受出区别来。

因此这里，我们拿一个真实的基于 xmake.lua 维护的 [tbox 项目](https://github.com/tboox/tbox) 来尝试各种编译。

首先，我们先从 gitee 上拉取下代码（github 上下载有点慢）。

```bash
cd ~/Code
git clone https://gitee.com/tboox/tbox --depth 1
```

拉取代码下来后，先进入 tbox 的项目根目录尝试直接用 xmake 编译感受下。

```bash
cd tbox
xmake
```

如果编译成功，会显示下面的信息。

![9](实验17工程生成插件使用.assets/282c76960643e9760ff990fb4f5a077b-0)

注：上图最底下黄色部分的警告信息不用管它，这是由于我们的实验环境中，gcc 编译器版本太低，tbox 项目里面配置的一些编译选项 gcc 不知道导致自动被 xmake 忽略了，但不会影响正常编译。

然后，我们尝试生成 build.ninja 文件，调用 ninja 编译。

```bash
xmake project -k ninja
ninja
```

我们可以看到，ninja 的整个编译输出风格上，还是跟 xmake 有些差异的，xmake 默认是回滚显示编译进度，而 ninja 始终在一行上回显进度信息，如下图。

![10](实验17工程生成插件使用.assets/1b67ad8d8777eccf44a229a891f024a7-0)

这个进度显示风格，仅仅只是个人喜好问题，并没有什么其它影响，不过 xmake 也可以通过切换主题风格，实现跟 ninja 一样的进度输出信息。

只需要执行下面的命令，全局配置切到 ninja 主题即可。

```bash
xmake g --theme=ninja
```

切过去后，我们再尝试执行下 `xmake -r` 编译看看输出效果，应该基本跟 ninja 保持一致了，如下图。

![11](实验17工程生成插件使用.assets/0f131f4f3ad4ff0492772be73d6bfe1a-0)

不过，这里只是临时性测试下，实验完成后，记得执行下面的命令，重置到默认主题风格。

```bash
xmake g -c
```



## 实验总结



在本节实验中，我们主要学习了如何使用 xmake 生成其它第三方的工程文件，例如：Makefile、CMakeLists.txt 和 build.ninja 等等，其实 xmake 还可以生成 vcproj 等 vs 工程文件，不过由于实验环境关系，这里没有详细讲解，有兴趣的同学，可以直接到 [xmake 的官方文档](https://xmake.io/#/zh-cn/plugin/builtin_plugins?id=生成visualstudio工程) 查阅。

本实验的参考代码可以使用如下命令下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/2764/code17.zip
```