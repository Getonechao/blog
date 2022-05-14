+++

title= "实验14io读写"
description= "蓝桥网课xmake笔记"
date= 2021-07-15T10:36:55+08:00
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

# io读写

在本节实验中，我们可以学习到 xmake 内置的 io 模块的基本操作，例如文件数据的快速读写和序列化，还能更深入地学习如何使用 print 接口来实现打印输出，以及在终端上获取用户的输入内容。

#### 知识点

- io 模块的介绍和基本使用
- 文件数据的读写和序列化
- 标准输入输出的读写，用户输入的获取



## io 模块介绍

在自定义脚本中，除了 os 模块对应的系统操作之外，还有文件 io 操作也是我们经常需要使用的，比如读写文件数据等等。而 xmake 内部提供的 io 模块是在 lua 原生模块的基础上做了一些改进，比如新增了序列化读写接口，文件锁接口等等，另外还对 unicode 编码也进行了很好的支持。

接下来，我们会逐一介绍下 io 模块中一些比较常用的接口是如何使用的。

#### 读写数据到文件

我们先来简单介绍下 io 模块中最简单方便地读写文件数据的接口操作，不过在此之前，我们还是先来创建一个用于当前实验操作的空工程。

```bash
cd Code
xmake create iotest
```

工程创建完成后，进入 iotest 文件夹，然后编辑 xmake.lua 文件，在编译前通过 io 模块读写 `src/main.cpp` 源文件，并做一些文本替换处理，然后重新写回后再进行编译。

```lua
target("iotest")
    set_kind("binary")
    add_files("src/*.cpp")
    before_build_file(function (target, sourcefile)
        local data = io.readfile(sourcefile)
        if data then
            print("readfile", data)
            data = data:gsub("world", "xmake")
            io.writefile(sourcefile, data)
            print("writefile", data)
        end
    end)
```

这里我们用到了 `io.readfile` 和 `io.writefile` 两个接口来快速的读写指定的文件，这两个接口并不是 lua 原生提供的接口，而是 xmake 为了方便 io 操作，额外封装的接口，可以让大家不必自己去 open 和 close 文件，就能快速读写文件数据，非常的实用。

通过执行 `xmake` 编译，我们会看到下图的数据内容，另外 `main.cpp` 中的 `world` 字符串也确实被替换成了 `xmake` 字符串。

![1](实验14io读写.assets/69fcddbe02f7f6300f6f61558c207790-0)

运行 `xmake run` 继续确认下输出结果是否真的被改写为 `hello xmake!`，如下图。

![2](实验14io读写.assets/0d6c3d53141068671e6699b09620db54-0)

当然除了快速读写，还可以使用原生的 `io.open` 接口来读写文件，虽然这样使用起来稍微繁琐些，但也是有一些不可替代的优势，比如对大文件的读写性能会非常的好，毕竟 `io.readfile` 需要一次性加载到内存，而通过 open 后返回的文件对象去读写可以按行读以及分批写入，极大地减少内存使用。

另外，这样的读写方式也更加的灵活，我们修改 xmake.lua 文件，将里面的读写方式改成使用 `io.open` 的原生接口。

```lua
target("iotest")
    set_kind("binary")
    add_files("src/*.cpp")
    before_build_file(function (target, sourcefile)
        local tmpfile = os.tmpfile()
        local ifile = io.open(sourcefile, 'r')
        local ofile = io.open(tmpfile, 'w')
        for line in ifile:lines() do
            line = line:gsub("xmake", "world")
            ofile:print("%s", line)
        end
        ifile:close()
        ofile:close()
        os.cp(tmpfile, sourcefile)
        os.rm(tmpfile)
    end)
```

这回，我们的脚本相比之前就稍显复杂了，我们先是通过 `io.open` 分别按只读模式 `r`、只写模式 `w` 打开两个文件，然后通过 `ifile:lines()` 接口按行读取源文件，然后做字符串替换，通过 `ofile:print()` 接口格式化写入 `os.tmpfile()` 创建的临时文件中去。

在脚本的最后，我们通过之前介绍过的 `os.cp` 和 `os.rm` 接口将文件覆盖回 sourcefile。

由于刚刚我们把 `main.cpp` 中的 `world` 字符串替换成了 `xmake`，这里又重新替换回了 `world`。首先执行 xmake 编译通过，然后运行 `xmake run` 确认下最终的替换结果。

![3](实验14io读写.assets/7b7cac17d674708101149453a15ead12-0)

#### 序列化和反序列化

上面介绍的文件读写接口，都是针对字符串数据的读写操作，然而有时候我们在读写数据时，还想额外的解析其中的数据结构。如果直接从字符串解析会非常繁琐，导致整个配置脚本可读性很差也不方便维护。

因此，xmake 扩展了 io 模块，新增了 `io.load` 和 `io.save` 两个读写接口，专门用来快速的序列化存储 lua 对象到文件，以及直接加载被序列化后的文件到 lua 对象。

这里所谓的 lua 对象，其实就是脚本中所有基础 lua 变量类型，比如：string、table、boolean 和 number 等，而我们通常都会去直接序列化 table 对象，这是 lua 原生的数据类型，它可以是字典类型也可以是数组类型，例如：`{1, 2, 3, a = 1}`。

通常，我们可以使用对象的序列化存储来实现一些在构建过程中对一些配置、状态信息的缓存和加载功能。比如，在 target 的 `before_build` 脚本内缓存当前目标程序的源文件列表，然后在 `after_build` 脚本中去加载显示出来。

修改 xmake.lua 文件为如下内容。

```lua
target("iotest")
    set_kind("binary")
    add_files("src/*.cpp")

    before_build(function (target)
        io.save("$(buildir)/sourcefiles", {sourcefiles = target:sourcefiles()})
    end)

    after_build(function (target)
        local sourcefiles = io.load("$(buildir)/sourcefiles")
        print(sourcefiles)
    end)
```

我们把序列化的数据保存在 `build/sourcefiles` 文件中，执行完 `xmake` 编译后，我们就通过 `print` 将 `io.load` 加载的数据对象整个 dump 了出来，并保留了原始的数据结构，如下所示。

![4](实验14io读写.assets/d74d3b529b3e0c3cdee8af631946bcbc-0)



## 获取用户输入

关于 io 操作，除了对于文件的读写，我们有时候还需要读写标准输入输出，也就是 stdout、stdin 这些，其中 stdout 就是我们通常使用 print 接口在终端下的输出，其实也就是内部对 `io.print` 的封装调用，而如果我们要接受用户在终端下输入，就需要通过 `io.read` 去读取 stdin 了。

为了获取用户输入，我们继续改造下 xmake.lua，通过在编译前提示用户输入 `y` 来确认是否需要继续编译，如果用户输入了其它字符，那么就终止编译。

```lua
target("iotest")
    set_kind("binary")
    add_files("src/*.cpp")

    before_build(function (target)
        cprint("please input: ${bright}y${clear} to continue building project (y/n)")
        io.flush()
        local confirm = io.read()
        if confirm:trim() ~= 'y' then
            print("build abort!")
            os.exit()
        end
    end)
```

上面的配置中，在输入前，我们会先打印一些提示信息来提示用户输入，然后调用 `io.flush` 刷新下之前已有的输入缓存后，就可以通过 `io.read()` 接受终端的输入数据了。

直到用户输入完毕按下回车，`confirm` 变量中就会存储实际的输入结果，根据这里面的内容判断用户是否输入了 `y` 字符。如果确实是 `y` 那么 xmake 就会继续完成编译，而如果不是，那么就会调用到配置里面的 `os.exit` 接口，强制中断整个 xmake 命令的执行。

大概的输入提示内容如下。

```bash
$ please input: y to continue building project (y/n)
# 等待用户输入
```

如果执行 `xmake -r` 命令，等到接受输入后，输入 `y`，可以看到整个编译继续正常完成了。

![5](实验14io读写.assets/babbf18bdae09c74b27e82e0f46c43e6-0)

而如果输入 `n`，编译过程就被中断了。

![6](实验14io读写.assets/2b35524b4413f34271f4519301e27a4d-0)

#### 打印标准输出

既然，我们可以使用 `io.read()` 来读取用户输入，那么也可以通过 `io.write()` 来显示输出到终端，不过通常不需要直接使用这个接口，可以使用 xmake 提供的 `print()` 接口打印输出。

它们的区别在于，我们扩展了 lua 原生的 `print` 接口，使其拥有更加强大的输出功能，可以同时支持格式化输出、对象序列化输出，自动换行等特性。而 `io.write` 仅仅只能显示原始的字符串数据到终端。

因此这里我们重点讲解 print 接口的使用，这在自定义脚本中通过打印来调试配置脚本时是非常有用的。

首先，我们先来尝试下格式化输出显示字符串信息，其传参格式和用法基本上跟 C/C++ 里面的 printf 函数完全一致，同样支持 `%s`、`%d` 等格式化参数。

修改 xmake.lua 文件，在里面的 `on_load()` 阶段的脚本中打印一些格式化字符串信息。

```lua
target("iotest")
    set_kind("binary")
    add_files("src/*.cpp")
    add_defines("TEST")
    add_syslinks("pthread", "dl", "m")
    on_load(function (target)
        print("defines: %s", target:get("defines"))
        print("syslinks: %s", target:get("syslinks"))
    end)
```

这里，我们通过 `target:get()` 接口获取了当前 target 程序的一些描述域配置信息，然后通过 `print` 的 `%s` 格式化参数作为字符串打印出来。

然后执行 `xmake` 查看 target 加载完成后，实际打印输出了哪些信息。

![7](实验14io读写.assets/32c0de6ad699d889d0b668b518901a8d-0)

从上图中，我们看到 `defines` 信息被正常输出显示了，但是 `syslinks` 的配置信息仅仅显示了 `table: 0x405578f0` 字样，看不出具体有哪些配置。

这是因为我们给 `syslinks` 设置了多个链接库，所以它当前的数据类型其实是 array 对象（在 lua 也就是 table 类型）。

如果要将其正常输出，还是可以用 print，不过就不能通过格式化字符串的方式输出了，而是直接去打印它。继续修改刚才的配置。

```lua
target("iotest")
    set_kind("binary")
    add_files("src/*.cpp")
    add_defines("TEST")
    add_syslinks("pthread", "dl", "m")
    on_load(function (target)
        print("defines: %s", target:get("defines"))
        print("syslinks:", target:get("syslinks"))
    end)
```

这回，我们把 `syslinks: %s` 改成了 `syslinks:`，也就是去掉了 `%s` 的格式化参数，这样 print 就不会走格式化模式，而是直接走 dump 对象模式。它会完整打印整个对象结构。

执行 `xmake` 查看结果。

![8](实验14io读写.assets/f1faa18200b2d4ca6e390b7598752c5e-0)

注：虽然在描述域也可以调用 print，但是通常不建议这么做，如果要调试打印 target 的配置信息，请尽量在 `on_load` 里面完成。

#### 在终端显示色彩文本

除了使用 print 进行正常的格式化打印输出，我们还可以使用 cprint 来显示色彩文本输出，而且当前的主流终端都基本上已经可以支持 16 色的 color codes 输出了。

关于这块，并不是我们构建项目的重点，因此只需要简单了解下用法就可以了，这个接口支持将颜色名称通过 `${red}` 的格式传入 cprint 接口，就能将后面的字符串文本，按前面指定的颜色输出。

这里，我们直接通过执行下面的命令来快速调用 `cprint("${red}hello ${green}xmake!")` 输出带有色彩的文本字符串。

```bash
xmake l cprint '${red}hello ${green}xmake!'
```

执行后的效果如下图所示。

![9](实验14io读写.assets/45db1a7b36905532c83a9d038c904aab-0)

xmake 不仅支持常用的 16 色输出，还支持 24 位色高彩输出以及 emoji 符号输出，关于这块详情，可以看下相关的文章 [色彩高亮显示](https://tboox.org/cn/2016/07/14/plugin-print-colors/)。



## 实验总结

在本节实验中，我们学习了如何通过 io 模块接口去读写文件数据，以及如何通过序列化的方式读写数据，另外我们还学习到了如何在终端上输出信息以及获取用户的输入内容。

本实验的参考代码可以使用如下命令下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/2764/code14.zip
```