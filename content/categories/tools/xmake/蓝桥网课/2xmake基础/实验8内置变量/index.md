+++

title= "实验8内置变量"
description= "蓝桥网课xmake笔记"
date= 2021-07-15T10:36:44+08:00
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

# 内置变量

在本节实验中，我们可以学习到 xmake 内置变量的基础使用，以及如何跟环境变量、shell、自定义选项配置交互来动态配置项目工程。

#### 知识点

- 了解 xmake 常用内置变量
- 构建配置与环境变量、shell 等交互



## 内置变量介绍

xmake 提供了 `$(varname)` 的语法，来支持在配置字符串中直接传递内置变量，例如：

```lua
add_cxflags("-I$(projectdir)")
```

它会在实际编译时，将内置的 `projectdir` 变量转换为实际的项目根目录：`-I/home/shiyanlou/hello`。

#### 配置传递项目根目录

在进入本实验前，我们先删除下之前的 hello 项目目录（如果存在的话），然后创建一个新的 hello 工程。

```bash
cd ~/Code
rm -rf hello
xmake create hello
```

然后进入创建的 hello 项目目录，直接执行如下命令快速打印输出内置变量进行测试。

```bash
cd hello
xmake l print '$(projectdir)'
```

上面的命令，相当于直接调用了 xmake 内置接口 print 去打印输出当前项目对应的 `$(projectdir)` 内置变量的值，也就是项目的根目录路径。

![图片描述](实验8内置变量.assets/e16509e99340989ffdac9bf6d743ebce-0)

这个变量会随着不同的项目自动适配对应的项目路径，因此我们可以在描述域配置中直接嵌入这个变量来使用。

接下来，我们将 xmake.lua 修改成如下内容。

```lua
target("hello")
    set_kind("binary")
    add_files("src/*.cpp")
    add_includedirs("$(projectdir)/inc")
```

通过上面的配置，我们可以大概看出，`$(projectdir)` 内置变量可用于在传参时快速获取和拼接变量字符串，使得传递 includedirs 时，可以获取到相对于项目根目录的路径字符串，也就是 `-I/home/shiyanlou/Code/hello/inc`。

在执行编译前，我们还需要创建缺失的 inc 目录，并且在里面创建一个空头文件。

```bash
mkdir inc
touch inc/stub.h
```

然后执行编译，并查看详细编译输出确认配置的头文件搜索路径是否生效。

```bash
xmake -rv
```

如果能够看到下图红线部分编译选项，则说明内置变量确实配置成功了。

![图片描述](实验8内置变量.assets/6dc051e6d59d558885e47ee5b2e65986-0)

除了获取项目根路径外，xmake 还提供了一些其它的内置变量，用于快速获取各种基本路径字符串，例如下面几个也是比较常用的路径。

- `$(buildir)`：实际构建输出目录。
- `$(tmpdir)`：临时目录。
- `$(curdir)`：当前运行目录。
- `$(scriptdir)`：当前 xmake.lua 配置脚本所在目录。
- `$(programdir)`：xmake 安装程序的 lua 脚本根目录。

#### 获取环境变量

除了获取各种基本路径字符串，我们还可以通过 `$(env varname)` 内置变量，在字符串中快速获取指定环境变量。

例如获取环境变量中的 `$HOME` 路径传递到 xmake.lua 配置中去。

```lua
target("hello")
    set_kind("binary")
    add_files("src/*.cpp")
    add_includedirs("$(env HOME)")
```

执行 `xmake -rv` 查看传入 gcc 的头文件搜索目录是否带上了 `$HOME` 路径。

![3](实验8内置变量.assets/336d5f24e7c8aed996a56dc26a47762a-0)

上图红线位置的 `-I/home/shiyanlou` 就是我们通过获取 `$HOME` 环境变量得到的字符串，至于如何获取当前终端下的所有环境变量，xmake 也提供了接口可以直接快速查看，执行如下命令即可，这个命令会打印所有的环境变量。

```bash
xmake l os.getenvs
```

![4](实验8内置变量.assets/9bf90277ea39a431dd264fedd8276cb5-0)

也可以指定查看特定环境变量的内容。

```bash
xmake l os.getenv HOME
```

其效果跟 `echo $HOME` 是一样的，但这个命令更加通用，其内部实际就是调用了 xmake 的内置接口 `os.getenv("HOME")`，关于 os 相关内置接口，我们会在后面的实验中详细讲解。

![5](实验8内置变量.assets/7acc1a329cba29b410f2437667085aed-0)

#### 执行获取 shell 输出内容

虽然在脚本域，我们可以通过 `os.run` 等接口快速执行其它 shell 程序，但这对初学者来说，使用起来还是繁琐些，因此 xmake 也提供了 `$(shell command)` 内置变量的方式，在描述域配置中快速执行 shell 命令，并嵌入执行输出结果到配置字符串中。

例如，现在有个需求，我们想用在编译 Linux 程序时，调用 `pkg-config` 获取到实际的第三方链接库名，也可以这么配置。

```lua
target("hello")
    set_kind("binary")
    add_files("src/*.cpp")
    if is_plat("linux") then
        add_ldflags("$(shell pkg-config --libs sqlite3)")
    end
```

在执行编译前，执行如下命令安装 libsqlite3 库到系统。

```bash
sudo apt update
sudo apt install -y libsqlite3-dev
```

执行 `xmake -rv` 命令编译就可以看到效果了，如图。

![7](实验8内置变量.assets/366d0ef11fde13d05a4b28f42a005a66-0)

上图红线部分就是执行了我们配置中传入的 shell 命令获取的输出结果。

不过通过执行外部 shell 命令来集成依赖库，我们并不推荐，这里只是出于演示目的，xmake 更推荐使用之前实验中介绍的 find_packages 或者 add_requires 配置接口来集成依赖库。

## 读取配置选项

其实，除了 xmake 内置的这些变量以外，只要是在 `xmake f/config` 配置命令中的所有参数选项配置，都是可以通过内置变量的方式来获取使用。

比如 `xmake f --plat=linux` 配置了目标平台，那么就能够通过 `$(plat)` 内置变量来获取；通过 `xmake f --mode=debug` 配置了编译模式，那么就可以通过 `$(mode)` 来获取到。

所有的配置选项，我们可以通过 `xmake f --help` 来查看配置菜单列表获取到，比较常用的有 `$(plat)`、`$(arch)`、`$(host)` 和 `$(mode)` 等，而之前说的 `$(buildir)` 也是其中一员，对应配置项 `xmake f -o buildir`。

接下来修改之前的 xmake.lua 尝试如何使用这些变量，具体修改如下。

```lua
target("hello")
    set_kind("binary")
    add_files("src/*.cpp")
    set_targetdir("$(projectdir)/bin/$(plat)/$(arch)/$(mode)")
```

这里，我们通过 `set_targetdir` 接口修改默认的目标程序输出路径，将其编译生成到项目根目录的 `bin` 目录下，并且根据不同的平台、架构和编译模式自动分子目录存放。

执行命令 `xmake -rv` 进行编译，然后执行 `tree bin` 查看实际生成的路径接口，如下图所示。

![8](实验8内置变量.assets/351341bca81ebfcffaa3f7fc22e49643-0)

注：这里只修改了可执行目标程序的存储路径，而 build 目录还是没有被修改，因为 build 目录下还会存储构建过程中的其它的一些中间对象文件。

#### 读取自定义选项配置

既然任意的 `xmake f/config` 配置选项参数都可以通过内置变量的方式获取到，那么我们自定义的配置选项，也可以用同样的方式获取到。

在上一节实验中，我们已经初步了解了如何自定义配置选项 `myopt`，现在继续在之前的配置基础上，通过内置变量的方式获取传入 `myopt` 中的值。

修改 xmake.lua 配置，在里面新增 myopt 选项定义，然后添加一个宏定义来获取 `myopt` 的配置值，根据传入的配置参数，动态切换宏定义开关值。

```lua
option("myopt")
    set_showmenu(true)
    set_description("The test config option")
option_end()

target("hello")
    set_kind("binary")
    add_files("src/*.cpp")
    add_defines("$(myopt)")
```

执行如下命令，传入 TEST1 到 myopt 选项参数，来定义 `-DTEST1` 宏。

```bash
xmake f --myopt=TEST1
xmake -rv
```

下图红线标注位置的编译参数就是我们通过 myopt 动态传入的宏定义。

![9](实验8内置变量.assets/14fa63ba9fc966e507ca9312fd131c46-0)

继续执行如下命令，传入 TEST2 到 myopt 选项参数，将宏定义修改为 `-DTEST2`。

```bash
xmake f --myopt=TEST2
xmake -rv
```

结果如下，宏定义又被改成了 `-DTEST2`。

![10](实验8内置变量.assets/84176b8a34d5fa61c5402a5ba16828f1-0)



## 实验总结

在本节实验中，我们了解了什么是内置变量，并且学习了如何去使用内置变量获取 shell 输出、环境变量和自定义选项值来配置项目。

本实验的参考代码可以使用如下命令下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/2764/code8.zip
```

