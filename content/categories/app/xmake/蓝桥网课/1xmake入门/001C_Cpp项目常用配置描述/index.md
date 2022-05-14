+++
title= "实验1C_C++项目常用配置描述"
description= "蓝桥网课xmake笔记"
date= 2021-07-15T10:36:39+08:00
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

## C_C++项目

在本节实验中，我们可以学习到一些编译器和链接器相关的基本概念，并且掌握 xmake.lua 中的一些基础配置，比如：C/C++ 头文件、链接库的配置，编译优化的相关配置，源文件的模式匹配等等。

#### 知识点

- 编译器、链接器基础概念
- xmake.lua 基本配置和修改
- 编译优化相关选项的配置
- C/C++ 头文件、链接库相关配置
- C/C++ 源文件的添加和过滤配置



在讲解本节实验之前，我们需要先简单了解下 C/C++ 程序的大致构建流程，它主要分为：预编译、编译和链接三个阶段，而这三个阶段会使用不同的工具去处理。

- 预编译：使用预编译器去处理 C/C++ 源文件中头文件、宏定义等。
- 编译：使用编译器对处理后的 C/C++ 源文件进行源码编译生成 `.o` 对象文件。
- 链接：使用链接器将编译生成的 `.o` 对象文件进行完整链接生成最终的可执行文件。

大致的编译链接流程可以参考下图。

![1](https://doc.shiyanlou.com/courses/2764/27526/6795acf26e4f15326f1d9970f80db3bd-1)

从图中，我们可以看到链接器除了会去链接所有 `.o` 对象文件外，如果提供了 `.a` 静态库和 `.so` 动态库，那么链接器也会对它们进行链接。

通常我们使用的一些第三方 C/C++ 库以及系统库，都是通过静态库或者动态库的方式提供，这两者的区别在于：

- 静态库：链接后会直接把里面所有的代码都合并生成到可执行程序中。
- 动态库：链接后仅仅将动态库中导出的函数提供给可执行程序调用，实际运行过程中，还是会单独加载这个动态库。

因此，使用静态库完整链接后的程序大小通常比使用动态库的程序大很多，但这也有自己的优势，就是依赖少、便于升级维护和部署。

#### 编译器和链接器

目前市面上常用的编译器有：gcc，clang，msvc 等，而这里主要使用 gcc 作为我们的编译器，可以在之前的 hello 工程下执行 `xmake -rv` 查看当前使用的编译器和链接器。

![2](001C_C++项目常用配置描述.assets/f6f5131977f1e9118e9b0d28bbff65e1-0)

从上图可以看出，gcc 就是我们当前使用的编译器，它可以直接编译 `.cpp` 源文件到 `.o` 文件，而 `g++` 就是我们使用的链接器，用来将所有生成的 `.o` 文件链接成可执行文件。

其中需要注意的是，其实 `g++` 也仅仅只是个编译器前端，它实际上会去内部调用真实的 `ld` 链接器程序，相比直接使用 `ld`，我们使用 `g++` 来链接的一个好处是，它会自动搜索和链接内置提供的 libc++ 等依赖库，这会省事不少，也更通用，否则如果直接使用 `ld` 的话，那么我们只能自己处理对这些依赖的搜索路径等配置选项了。

而关于实际的内部链接器，比较常用的有 ld.bfd、ld.gold 和 lld，其中 bfd 用的最多，而 lld 是 llvm 新出的链接器，链接速度更加快，关于这块细节与本实验关系不大，就不再过多展开讨论了。



## xmake.lua 基础配置

接下来，我们开始逐一介绍 xmake 的配置文件 xmake.lua 里面的一些基础配置接口使用方式，只要大概掌握了这些配置，基本上常规的 C/C++ 项目编译和维护都能快速搞定了。

#### 配置目标类型

首先，我们来介绍下 xmake 支持的几种基础目标类型，目前主要也就是四种类型。

| 值     | 描述       |
| ------ | ---------- |
| phony  | 空目标程序 |
| binary | 二进制程序 |
| static | 静态库程序 |
| shared | 动态库程序 |

#### 可执行程序类型

比如我们之前一直在用的可执行目标程序类型 `binary`，可以通过 `set_kind("binary")` 来设置，如下面的配置。

```lua
target("hello")
    set_kind("binary")
    add_files("src/*.cpp")
```

#### 静态库程序类型

我们接下来尝试编译生成一个带有静态库的目标程序，直接使用 xmake 来创建个空工程，例如。

```bash
cd ~/Code
xmake create -t static hello_static
```

`-t static` 参数用于指定当前工程类型为静态库，后面是工程项目名，生成后的项目结构如下。

![图片描述](001C_Cpp项目常用配置描述.assets/26ce1f0bc2d4ab2c3e64991386dfb579-0)

其中，项目根目录下还是会自动生成一个 xmake.lua 用于维护整个项目构建，而 src 源码目录下 `interface.cpp` 就是用来参与静态库编译，`main.cpp` 用来生成可执行程序，在里面调用静态库的接口。

我们可以进入 hello_static 项目根目录后，执行 `gvim ./xmake.lua` 查看下里面的内容，大致如下。

![4](001C_Cpp项目常用配置描述.assets/50281287c151a7b38d0d7ecf585a28d2-0)

里面有两个编译目标，通过 `target()` 定义维护，一个是 static 静态库类型，一个是 binary 可执行程序类型，它们之间通过 `add_deps("hello_static")` 进行依赖关联，这样 xmake 在编译的时候，会优先编译依赖的静态库，并且把静态库自动集成到对应的可执行程序上去。

执行 `xmake` 对这个工程进行编译，如果输出结果跟下图基本一致，说明我们成功生成了一个链接静态库的目标程序。

![5](001C_Cpp项目常用配置描述.assets/4de8234eedd78b4da1331687d4b5a336-0)

然后我们执行 `xmake run` 运行下这个程序。

![6](001C_Cpp项目常用配置描述.assets/3af8597a38cce093c496ea1084e04e5a-0)

#### 动态库程序类型

动态库的创建跟静态库类似，只需要把类型名改成 `shared` 就行了，例如。

```bash
cd ~/Code
xmake create -t shared hello_shared
```

生成的工程结构和 xmake.lua 内容都跟静态库的基本一致，唯一的区别就是目标类型变成了 `shared`。

```lua
add_rules("mode.debug", "mode.release")

target("hello_shared")
    set_kind("shared") -- 设置为动态库目标程序
    add_files("src/interface.cpp")

target("hello_shared_demo")
    set_kind("binary")
    add_deps("hello_shared")
    add_files("src/main.cpp")
```

然后我们执行下编译和运行。

```bash
cd hello_shared
xmake
xmake run
```

编译运行结果也跟静态库基本一致。

![7](001C_Cpp项目常用配置描述.assets/5e07dc323beadec5332bff84c2dc2584-0)

#### Phony 目标类型

其中，phony 是一个特殊的目标程序类型，使用它定义的 target 目标不会生成任何实际的程序文件，也不会执行任何编译操作，通常用于组合其它目标程序的依赖关系，实现关联编译。

为了更好的理解 phony 类型，我们这回手工创建个带有 phony 目标的工程进行测试，首先创建个空目录。

```bash
cd ~/Code
mkdir phony_test
cd phony_test
```

然后执行 `vim ./xmake.lua` 编辑 xmake.lua 写入下面的内容。

```lua
target("foo")
    set_kind("phony")
```

可以看到，这边我们没有添加任何源文件，这个 foo 目标仅仅只是个空目标程序，执行编译也是空执行，不会有任何源文件参与编译。

![8](001C_Cpp项目常用配置描述.assets/6eb55fd3da9f7044dcb1ca8cf2c4593a-0)

而关于使用 phony 目标跟其它目标程序的关联编译，我们会在实验 6 《xmake 基础之目标依赖》中详细讲解。



## 添加 C/C++ 编译选项

在实验 2 《xmake 的基本命令使用》中，我们介绍了通过命令行的方式快速添加 C/C++ 编译选项，不过这种方式不适合永久保存和对外发布，仅仅用于本地编译。如果我们想将自己的项目发布后提供给其他用户编译，那么就需要在 xmake.lua 中去设置这些编译选项。

进入之前创建的 hello 程序目录，然后使用 `vim xmake.lua` 编辑里面的 xmake.lua 文件，修改成如下内容。

```lua
add_rules("mode.debug", "mode.release")

target("hello")
    set_kind("binary")
    add_files("src/*.cpp")
    add_cxflags("-DTEST1")
```

通过 `add_cxflags("-DTEST1")` 设置就能对所有的 C/C++ 代码定义上 `TEST1` 宏开关，然后通过执行 `xmake -v` 查看完整编译命令中，是否真的生效了。

![9](001C_Cpp项目常用配置描述.assets/e8cce8f2cdf89db44a61e712fbd9d9db-0)

上图中，我们确实可以看到 `-DTEST1` 被传递到了 gcc 编译器进行宏定义。

另外在上图中，我们还看到了一些编译选项的自动检测，这是由于通过 `add_cxflags()` 等设置接口添加的选项都是特定某些编译器的，并不一定是完全跨编译通用的，因此 xmake 会去检测用户的设置，如果不支持的选项被设置进去，xmake 会检测失败并提示警告信息然后忽略它。

#### 添加宏定义

除了可以通过 `add_cxflags` 方式添加宏定义，其实 xmake 还提供了内置的配置接口 `add_defines` 去更方便设置它，这个配置接口相比 `add_cxflags` 使用原始编译选项的方式更加的通用，跨编译器，也是我们更加推荐的配置方式。这个接口会自动处理各种编译的支持方式，所以不会触发 flags 的自动检测机制，更加的快速可靠。

我们还是基于之前的 hello 项目的配置，做一些修改，通过 `add_defines("TEST2")` 添加新的宏定义开关 `TEST2`。

```lua
add_rules("mode.debug", "mode.release")

target("hello")
    set_kind("binary")
    add_files("src/*.cpp")
    add_cxflags("-DTEST1")
    add_defines("TEST2")
```

然后，我们重新执行 `xmake -v` 去编译查看新增的宏定义是否生效，如图。

![10](001C_Cpp项目常用配置描述.assets/7c5bb8721171913ac459409760248b40-0)

从图中，我们可以看出，相比之前的输出，新增加了一个 `-DTEST2` 宏定义参数选项传入了 gcc，说明我们的配置确实生效了。

#### 头文件路径和链接库配置

xmake 也提供了方便的配置接口去配置头文件搜索路径、链接库、库搜索路径，分别是如下几个接口。

- `add_includedirs`：添加头文件搜索目录。
- `add_linkdirs`：添加库搜索目录。
- `add_links`：添加链接库。
- `add_syslinks`：添加系统链接库。

上述几个接口都是跨编译器配置，相比直接设置到 ldflags 更加的通用。至于 `add_syslinks` 和 `add_links` 这两个的区别就是：`add_syslinks` 通常用于添加一些系统依赖库，比如 pthread，这样 xmake 会把这些系统库链接放置的更靠后些。这是因为链接器在处理库链接时候，是会依赖顺序的，放置在最左边的链接会优先处理，例如 libfoo 库依赖 libpthread 库中的符号，那么我们必须严格按照这个顺序添加链接库依赖 `-lfoo -lpthread`，否则链接器就会报找不到 pthread 库符号的错误。

而链接库和头文件搜索路径，顾名思义，就是设置后告诉编译器、链接器应该从这些目录位置尝试查找指定的链接库和头文件。

继续配置 hello 工程中的 `xmake.lua` 文件。

```lua
add_rules("mode.debug", "mode.release")

target("hello")
    set_kind("binary")
    add_files("src/*.cpp")
    add_cxflags("-DTEST1")
    add_defines("TEST2")
    add_links("z")
    add_syslinks("pthread")
    add_linkdirs("/tmp")
    add_includedirs("/tmp")
```

然后执行 `xmake -v` 查看编译输出，里面的红框部分就是新设置的编译选项：

- `-I/tmp`：添加头文件搜索路径。
- `-L/tmp`：添加链接库搜索路径。
- `-lz -lpthread`：添加的链接库，由于我们是通过 `add_links` 添加的 zlib 库，而 pthread 库是作为系统库添加的，所以被放置在 pthread 的左边优先链接。

![11](001C_Cpp项目常用配置描述.assets/e39af17e83f3a839cf7aadc012ad0f70-0)

#### 设置语言标准

我们可以使用 `set_languages` 接口设置目标代码编译时候的语言标准，比如是基于 c99 标准，还是 c++11、c++14 标准等。

目前 xmake 支持的语言标准主要有以下几个：

| 值      | 描述                    |
| ------- | ----------------------- |
| ansi    | c 语言标准: `ansi`      |
| c89     | c 语言标准: `c89`       |
| gnu89   | c 语言标准: `gnu89`     |
| c99     | c 语言标准: `c99`       |
| gnu99   | c 语言标准: `gnu99`     |
| c++98   | c++ 语言标准: `c++98`   |
| gnu++98 | c++ 语言标准: `gnu++98` |
| c++11   | c++ 语言标准: `c++11`   |
| gnu++11 | c++ 语言标准: `gnu++11` |
| c++14   | c++ 语言标准: `c++14`   |
| gnu++14 | c++ 语言标准: `gnu++14` |
| c++1z   | c++ 语言标准: `c++1z`   |
| gnu++1z | c++ 语言标准: `gnu++1z` |
| c++17   | c++ 语言标准: `c++17`   |
| gnu++17 | c++ 语言标准: `gnu++17` |

并且 c 标准和 c++ 标准可同时进行设置，例如：

```lua
-- 设置 c 代码标准：c99， c++ 代码标准：c++11
set_languages("c99", "c++11")
```

将这个配置添加到刚刚修改的 xmake.lua 文件中去，就跟上一节的配置方式一样，放置到 `target("hello")` 配置域下面，然后执行 `xmake -v` 看下编译输出结果。

![12](001C_Cpp项目常用配置描述.assets/35f4b7d3a34c05ccc62c7c86159b000a-0)

由于当前没 C 代码，所以实际只有 C++11 的标准设置生效了，也就是上图红框位置的编译选项 `-std=c++11`。

#### 设置编译优化

其实，xmake 默认创建的工程 xmake.lua 文件中，已经设置了 `add_rules("mode.debug", "mode.release")` 这两个编译规则，而默认 xmake 编译就是 release 模式编译，它会开启所有内置的编译优化选项，并不需要用户设置什么。

![13](001C_Cpp项目常用配置描述.assets/9ce81350b7740ae9abd6509ee2d8da3c-0)

上图红框的部分都是编译优化相关的一些选项，比如最直接的 `-O3` 优化，还有 `-fvisibility=hidden -fvisibility-inlines-hidden` 用于去重一些符号字符串数据，使得编译后的程序更小。

当然，我们也可以不使用 xmake 提供的内置编译规则，自己控制应该如何优化编译，比如继续改成下面配置。

```lua
target("hello")
    set_kind("binary")
    add_files("src/*.cpp")
    add_cxflags("-DTEST1")
    add_defines("TEST2")
    add_links("z")
    add_syslinks("pthread")
    add_linkdirs("/tmp")
    add_includedirs("/tmp")
    set_languages("c99", "c++11")
    if is_mode("release") then
        set_optimize("fastest")
        set_strip("all")
        set_symbols("hidden")
    end
```

我们去除了 `mode.release` 的编译规则，通过 `is_mode("release")` 自己判断和控制优化编译，可以达到跟之前一样的优化效果。

其中 `set_optimize("fastest")` 就是添加 `-O3` 的编译优化开关，而 `set_strip("all")` 和 `set_symbols("hidden")` 用于去掉调试符号数据，使得程序更小。

配置完成以后再执行 `xmake -v` 看看，最后的编译输出里面这些编译优化选项还是同样存在的。

#### 添加源文件

在本实验最后，再讲解下如何添加源文件参与编译，其实之前的配置中，我们已经大概知道，可以通过使用 `add_files` 接口来添加源文件，这里进一步了解下这个配置接口的使用。

其实这个接口还是很强大的，不仅仅支持单个源文件的添加，还可以支持通过模式匹配的方式批量添加源文件，并且支持同时添加不同语言的源文件，不过这里主要讲解对 C/C++ 源文件的添加。

我们先来看个实例，简单了解下用法。

```lua
add_files("src/test_*.c", "src/**.cpp")
```

其中通配符 `*` 表示匹配当前目录下源文件，而 `**` 则递归匹配多级目录下的源文件。

在之前的配置中，我们是用 `add_files("src/*.cpp")` 仅仅匹配 src 单级目录下的 c++ 源文件，现在有了大致的了解后，编辑 hello/xmake.lua 文件来实验如何递归添加源文件，将该文件改为下面的配置。

```lua
add_rules("mode.debug", "mode.release")

target("hello")
    set_kind("binary")
    add_files("**.cpp")
```

然后我们在 `hello/src/test` 目录下，通过下面的命令新创建一个 C++ 的空代码文件，用来测试验证递归模式匹配。

```bash
cd ~/Code/hello
mkdir -p src/test
touch src/test/stub.cpp
```

创建好后执行 `xmake` 编译工程，如果顺利就可以看到新添加的 stub.cpp 文件也参与了编译，如下图。

![14](001C_Cpp项目常用配置描述.assets/7826a1a02f632ddfc65a68cb5012b57c-0)

#### 过滤源文件

模式匹配的方式虽然很方便，但如果源码目录层级结构复杂，在添加过程中，需要排除一些不需要的文件时，就不是那么灵活了，不过 xmake 也提供了在模式匹配过程中排除一批文件的方式，同样还是这个配置接口，我们只需要通过 `|` 符号指定后面的排除匹配模式即可，例如。

```lua
target("hello")
    set_kind("binary")
    add_files("src/**.cpp|test/*.cpp")
```

其中分隔符 `|` 之后的都是需要排除的文件，这些文件也同样支持匹配模式，并且可以同时添加多个过滤模式，只要中间用 `|` 分割就行了，而这里的配置就是在 `src/**.cpp` 基础上，忽略掉其中 `test` 子目录下所有的 C++ 源文件，最终的结果也就是仅仅编译 `src/main.cpp`。

我们再来执行下 `xmake -r` 重新编译验证下结果，如下图。

![15](001C_Cpp项目常用配置描述.assets/1fffc429e83c688a883f3512419d9666-0)

上图中，可以看到我们成功过滤掉了 `src/test/stub.cpp` 仅仅编译 `src/main.cpp`。

还有需要注意的一点是，为了使得描述上更加的精简，`|` 之后的过滤描述都是基于前一个模式：`src/**.cpp` 中 `*` 所在的目录作为根目录，也就是 `src` 目录的基础上来过滤匹配的，所以后面的过滤子模式只需要设置 `test/*.cpp` 而不是 `src/test/*.cpp`。

## 实验总结

在本节实验中，我们学习了编译链接的基本概念，如何去配置头文件和链接库，如何去添加编译优化选项、宏定义以及如何添加源文件等一些 xmake 的常用配置，通过这些配置，我们基本上已经可以解决大部分 C/C++ 基础工程的编译需求。

本实验的参考代码可以使用如下命令下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/2764/code3.zip
```