+++

title= "实验7条件判断"
description= "蓝桥网课xmake笔记"
date= 2021-07-15T10:36:43+08:00
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

# 条件判断

在本节实验中，我们会学习到 xmake 条件判断语句的基础用法，以及常用的平台、架构等判断接口，另外还会学习如何判断自定义选项的值来控制编译。

#### 知识点

- xmake 常用条件判断接口
- xmake 条件判断语法
- 如何自定义配置选项



## 条件判断语法简介

由于 xmake 是基于 lua 脚本的，所以不管是在描述域还是脚本域中的配置，我们都可以通过基础的 lua 条件判断语法去控制配置逻辑。

整个条件控制的语法逻辑大概如下。

```lua
if expr then
    -- ...
elseif not expr then
    -- ...
else
    -- ...
end
```

整个语法结构非常的简单，并且同时适用于描述域配置和脚本域配置，其中 expr 部分为条件判断表达式，我们可以通过 not，and 和 or 三个逻辑关键字组合成各种复杂的逻辑条件，例如。

```lua
if a > b and not c or d then
    -- ...
end
```

不过，对于构建配置，我们不需要写这么复杂的条件，通常仅仅只需要判断下平台、架构等基础条件来控制编译配置就行了。因此为了简化条件判断，xmake 提供了一些内置的条件判断接口，用来快速判断它们，并且也简化了整个条件判断的逻辑，下面我们来详细讲解下。

#### 判断当前目标平台

在构建配置中，如果要支持跨平台构建，那么最常用的莫过于判断当前程序编译后的运行平台了，也就是通过判断我们编译生成的可执行文件和库程序实际的运行平台环境，来控制编译配置。

在开始实验前，我们先执行下面的命令创建一个新的空工程，然后在里面创建一个 `src/linux/stub.cpp` 的空代码文件。

```bash
cd Code
xmake create condition
cd condition && mkdir src/linux
touch src/linux/stub.cpp
```

其工程文件结构如下。

```txt
.
├── src
│   ├── linux
│   │   └── stub.cpp
│   └── main.cpp
└── xmake.lua
```

在准备工作做好后，我们开始编辑 xmake.lua 修改成下面的配置。

```lua
target("test")
    set_kind("binary")
    add_files("src/*.cpp")
    if is_plat("linux") then
        add_defines("LINUX")
        add_files("src/linux/*.cpp")
    end
```

在上面的配置中，我们通过内置的条件判断接口 `is_plat("linux")` 来判断当前的目标平台是否为 Linux，如果当前正在为 Linux 程序做编译，那么我们会切入里面的 if 分支，额外编译 `src/linux/*.cpp` 等针对 Linux 平台特有的源文件，并且同时添加上 `-DLINUX` 宏定义。

由于实验环境就是 Linux 系统，并且编译的目标程序也是针对 Linux 平台的，因此我们可以执行 `xmake` 命令编译确认 `src/linux/*.cpp` 源文件是否真的参与了编译。

![1](实验7条件判断.assets/490b5e4b0069b06b8563c7b81d63d181-0)

上图红线位置的源文件就是我们刚刚配置的仅对 Linux 平台才会参与编译的代码文件，我们再来执行 `xmake -rv` 查看详细的编译输出，`-DLINUX` 也被正常传递进了 gcc 编译器。

![2](实验7条件判断.assets/483a67e20edc03c376c62eb013df0ce9-0)

另外需要说明的是，目标编译平台是可以通过 `xmake f -p linux` 显式设置的，它会跟 `is_plat("linux")` 保持一致，如果编译前没有配置过平台，那么默认会使用当前主机平台来作为目标编译平台。

而显式指定编译平台，通常在交叉编译时候非常有用，例如在 Linux 系统上使用 Android NDK 编译 android 库程序，就可以通过 `xmake f -p android` 切换 Android 目标平台编译，这个时候就是对应条件配置 `is_plat("android")` 而不是 Linux。

由于 Android 也是基于 Linux 系统，所以有些配置是可以跟 Linux 保持一致的，这个时候，我们可以使用 `is_plat("android", "linux")` 来判断多个平台，它们之间是属于`或`的关系。

```lua
if is_plat("linux", "android") then
    add_defines("LINUX")
    add_files("src/linux/*.cpp")
end
```

关于 Android 等其它平台的切换编译详情，可以看下 [官方文档](https://xmake.io/#/zh-cn/guide/configuration?id=android)，这里就不多做介绍了。

而目前 xmake 支持的所有平台可以简单列举下：

- Windows
- Cross
- Linux
- macOS
- Android
- Iphoneos
- Watchos
- Freebsd

#### 判断编译指令架构

编译指令架构指的是生成的目标程序实际执行的指令架构，例如：x86_86、i386、armv7 等指令集。

在 Windows 系统上通常是 x64、x86 指令集，而在 Linux/macOS 系统上通常是 x86_64、i386 指令集，另外在一些嵌入式设备、移动端操作系统上通常是 armv7、arm64、mips 等指令架构。

通过判断指令架构，我们可以针对性配置处理不同架构的编译宏、代码以及特殊编译优化选项等。比如可以判断当前编译指令架构如果是 x86_64 的话，就再额外编译 `src/x86_64/stub.S` 汇编文件，里面可以针对这个架构实现一些特殊的汇编优化代码。

将 condition/xmake.lua 修改为如下配置。

```lua
target("test")
    set_kind("binary")
    add_files("src/*.cpp")
    if is_plat("linux") then
        add_defines("LINUX")
        add_files("src/linux/*.cpp")
    end
    if is_arch("x86_64") then
        add_files("src/x86_64/*.S")
    end
```

我们新增了对 x86_64 架构的判断去编译一些特定的汇编代码，当然这里我们仅仅为了演示，所以只是通过下面的命令创建一个空的汇编代码文件，里面没有添加任何实现。

```bash
mkdir src/x86_64
touch src/x86_64/stub.S
```

我们再来强制执行下编译，看下编译输出，由于当前的实验环境就是 Linux 系统的 x86_64 架构，所以会触发新加的条件配置，结果如下图红线位置所示。

![3](实验7条件判断.assets/c44ce0fd4ecb52b181039666242c6630-0)

#### 判断宿主平台架构

除了上述的目标程序实际运行平台和架构，有时候我们还需要判断当前宿主环境的系统平台和指令架构（也就是编译器实际运行的系统平台）。

更直观点说，比如我们在 Linux 上编译生成 Android 系统的 so 库程序，那么 Linux 就是宿主平台，而 android 就是实际的目标程序平台，这通常在交叉编译环境中，更需要做如此区分。

但是对于编译的目标程序也是在当前系统环境下运行的，那么宿主平台和目标平台是完全一致的。

可以通过 `is_host()` 配置接口来判断宿主平台，xmake 不会从 `xmake f -p/--plat` 中去取目标平台的实际值，因为宿主平台在 xmake 运行时候就是固定的，所以我们可以直接从 `os.host()` 和 `os.arch()` 接口去取对应的值，例如执行如下命令，直接查看当前的宿主平台和架构是多少。

```bash
xmake l os.host
xmake l os.arch
```

运行结果如下图。

![4](实验7条件判断.assets/864e44f60e9578096c32b02240de2b50-0)

至于刚刚所说的交叉编译，也就是在当前主机环境下使用交叉编译工具链生成只能在其它设备才能运行的目标程序，如果在 Linux 上编译 Android 程序或者编译其它 arm，mips 等架构的嵌入式目标程序，这些编译后的目标程序在当前的宿主环境是没办法执行的，只能在对应的设备上才能运行。

这个时候，宿主平台就是我们的 Linux 系统平台，也就是交叉编译工具链的执行环境，而目标平台就是对应程序所在设备上的系统平台，它并不一定是宿主系统的 Linux 环境。

除了一些已知的 Android，Iphoneos 等特定系统平台，其它各种交叉编译工具链平台，我们都可以统一使用 `xmake -p cross` 交叉编译平台来编译。

这里为了演示宿主平台和目标平台的区别，我们可以通过 `xmake f -p cross` 命令切换到 cross 交叉编译平台下执行 `xmake -r` 重新编译，这个时候目标平台就不再是 Linux 了。这里由于切换了平台，因此我们追加 `-c` 强制触发下编译环境的检测。

```bash
xmake f -p cross -c
xmake -r
```

由于这个时候我们的配置还是判断的目标平台，但已经通过命令切换到了 cross 编译平台下，所以不再是 Linux 和 x86_64 了，实际的编译就会变成下图所示。

![5](实验7条件判断.assets/76fd2c2e9950888adaf5019af42265b7-0)

Linux 和 x86_64 相关的代码文件就不再参与编译了，接下来我们将之前的配置改成判断宿主平台和架构。

```lua
target("test")
    set_kind("binary")
    add_files("src/*.cpp")
    if is_host("linux") then
        add_defines("LINUX")
        add_files("src/linux/*.cpp")
    end
    if os.arch() == "x86_64" then
        add_files("src/x86_64/*.S")
    end
```

然后我们再切到 cross 平台，执行相同的编译。

```bash
xmake f -p cross -c
xmake -r
```

这回我们看到，Linux 和 x86_64 的代码重新参与了编译，这是因为虽然编译平台切换到了 cross 交叉编译，但是实际 xmake 运行的宿主平台还是我们的 Linux 实验环境，永远不会改变。

![图片描述](实验7条件判断.assets/bb8f31171ecd0c4dbd1f36b07605fc6a-0)



## 判断编译模式

虽然 xmake 提供了默认的编译模式规则可以让大家很方便的通过 `xmake f -m debug` 切换各种编译模式，不过有时候内置的 `mode.debug`，`mode.release` 等编译模式不一定完全满足需求。这个时候，就需要大家自己来判断当前处于什么编译模式，然后自己去控制编译优化、调试符号等各种编译选项的开启和关闭。

下面，我们将尝试完全使用 `is_mode()` 来自定义判断配置 debug 和 release 编译模式下的一些特定编译选项，实现和内置的 `add_rules("mode.release", "mode.debug")` 编译规则一样的控制效果。

继续修改 condition/xmake.lua 文件的配置为如下所示。

```lua
if is_mode("release") then
    set_symbols("hidden")
    set_optimize("fastest")
    set_strip("all")
elseif is_mode("debug") then
    set_symbols("debug")
    set_optimize("none")
end

target("test")
    set_kind("binary")
    add_files("src/*.cpp")
```

这次我们通过 `is_mode()` 来条件判断当前编译模式是否为 debug 还是 release，然后分别设置不同的编译选项，并且把这些选项设置到全局根域，这样可以对所有 target 生效，也就避免了每个 target 都去重复设置一遍。

如果是 release 编译，也就是 xmake 默认的编译模式，那么配置中，我们启用了 fastest 优化编译，并且去除了所有调试符号信息，相对于 gcc 编译选项就是 `-fvisibility=hidden -fvisibility-inlines-hidden -O3`。

执行如下命令切换到 release 编译模式进行验证。

```bash
xmake f -m release
xmake -rv
```

![7](实验7条件判断.assets/0daec8b26687a964b2aa5c0b737c7554-0)

而如果启用了 debug 编译模式，那么我们会开启调试符号信息，并且禁用所有优化编译，也就是相对于 gcc 中的 `-g -O0` 编译选项。

执行如下命令切到 debug 编译模式进行验证。

```bash
xmake f -m debug
xmake -rv
```

![8](实验7条件判断.assets/6ee6be2634a7427296d471cda8cf7173-0)

#### 判断自定义配置选项

除了使用 xmake 提供的一些内置条件判断接口来做条件外，为了更加灵活的扩展性，xmake 提供了一种可以让大家自由扩展配置选项的方式，使得在 xmake.lua 中更加定制化的判断配置项。

例如，我们自定义一个配置项 `--myopt` 到命令行。

```console
xmake f --myopt=hello
```

当然，现在直接执行肯定会报错的，因为我们还没有去定义这个选项。

首先，我们在 condition/xmake.lua 文件开头的全局作用域部分加入下面的配置。

```lua
option("myopt")
    set_showmenu(true)
    set_description("The test config option")
option_end()
```

其中，`set_showmenu(true)` 用于启用对外导出，这样 help 帮助菜单中才能看到这个选项，用户才能在命令下使用。

执行如下命令，查看是否已经有了这个选项。用 `grep myopt` 来快速定位新创建的配置选项。

```bash
xmake f --help | grep myopt
```

![9](实验7条件判断.assets/8a4bb3f4540554032a5f263c6d1c87d5-0)

现在能够在帮助菜单中看到自定义选项了，接下来我们就可以在 xmake.lua 中去判断定义的配置选项来控制编译了。例如我们判断 myopt 的配置值如果是 hello，那么就添加 `-DHELLO` 的宏定义参与编译。

```lua
option("myopt")
    set_showmenu(true)
    set_description("The test config option")
option_end()

target("test")
    set_kind("binary")
    add_files("src/*.cpp")
    if is_config("myopt", "hello") then
        add_defines("HELLO")
    end
```

上面就是我们的完整配置，通过 `is_config("myopt", "hello")` 来判断自定义选项配置 myopt 当前是否为 hello，如果是就会自动追加 `-DHELLO` 宏定义。

执行如下命令，设置 myopt 配置值后，查看编译输出。

```bash
xmake f --myopt=hello
xmake -rv
```

如果一切顺利，那么就会看到红线位置的 `-DHELLO` 已经被传入了 gcc 编译器。

![10](实验7条件判断.assets/852221719fc35be79e174d609e8e002b-0)

而如果我们清除 myopt 配置，那么 `-DHELLO` 宏定义也就不会被设置上。

```bash
xmake f -c
xmake -rv
```

可以看到下图中已经没有 `-DHELLO` 了。

![11](实验7条件判断.assets/4e820bef3eadc2bb6a17c4641bd9d3e7-0)



## 实验总结

在本节实验中，我们学习了如何去判断条件语句控制编译逻辑，也了解了一些 xmake 内置的平台、架构判断接口，另外还学会了如何判断自定义的选项开关。

本实验的参考代码可以使用如下命令下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/2764/code7.zip
```