+++

title= "实验13系统操作详解"
description= "蓝桥网课xmake笔记"
date= 2021-07-15T10:36:54+08:00
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

# 实验13系统操作详解

在本节实验中，我们可以学习到如何在自定义脚本中对文件系统操作，以及如何去执行外部 shell 命令程序并获取运行后的输出内容。

#### 知识点

- 系统 os 模块接口的介绍使用
- 文件和目录操作
- 外部命令的执行



## os系统模块介绍

xmake 提供了很多的内置模块和接口，可以让用户在自定义脚本域中随意使用，方便用户处理编译过程中的各种定制化逻辑。其中，我们多少会需要涉及到一些系统调用、io 操作等。

因为这些是最基本也是最常用的操作接口，而 lua 语言自身就有提供 os 和 io 内置模块去处理这些，但是 lua 自带的这两个模块功能有限，使用也不是很方便，并且也无法满足用户对编译的一些特殊需求。因此 xmake 在此基础上，做了些扩展和改进，封装了更多好用的接口提供给大家使用。

在本实验中，我们重点介绍如何使用 os 系统模块里面的各种常用接口。

#### 复制删除文件和目录

首先我们来讲解如何使用 os 模块去复制和删除指定的文件以及目录。

对于复制文件，xmake 提供了内置的 `os.cp(src, dst)` 接口，可以从源路径复制到目的路径，同时支持文件和目录的复制，并且支持 `*` 来模式匹配。

而对于删除文件，我们可以使用 `os.rm()` 接口，它也同时支持文件和目录的删除。在实验前，我们还是一样，先执行下面的命令，创建一个新的空工程。

```bash
cd ~/Code
xmake create ostest
```

接下来进入 ostest 目录，修改 xmake.lua 文件，加入对 os 模块接口的调用，具体修改如下。

```lua
target("ostest")
    set_kind("binary")
    add_files("src/*.cpp")
    after_build(function (target)
        os.cp(target:targetfile(), "$(buildir)/test")
        os.rm(target:targetfile())
    end)
```

上面的配置中，我们通过在 `after_build` 的自定义脚本中，额外调用 `os.cp` 接口将编译完成后生成的目标程序复制到 `build/test`，然后调用 `os.rm` 删除了源文件，实现了移动目标程序路径的目的。

当然，这里只是出于演示目的，其实要做到这个需求，并不需要这么复杂，只需要调用 `os.mv` 或者直接使用 `set_targetdir` 去修改目标路径更加的方便省事。

而对于 `os.cp` 等带有路径参数的接口，它们也可以像描述域配置那样支持将内置变量 `$(buildir)` 直接嵌入字符串。

执行 `xmake` 命令，并通过 `tree` 命令看下实际生成的目标路径。

![img](实验13系统操作详解.assets/1007ed2bb6b5e9f8a83606cdf92c087c-0)

从图中，我们看到 `release` 目录已经变成了空目录，因为 `test` 程序已经被移动到了 `build` 目录下。

#### 遍历获取文件和目录列表

既然有了文件的复制删除操作，那肯定少不了文件的遍历操作，这个也是我们经常需要用到的接口。而关于文件遍历，xmake 其实提供了三个接口来分别实现各种粒度的遍历，主要有如下几个。

| 遍历接口    | 接口说明           |
| ----------- | ------------------ |
| os.dirs     | 遍历所有目录       |
| os.files    | 遍历所有文件       |
| os.filedirs | 遍历所有文件和目录 |

这三个接口除了遍历获取的文件类型有不同外，其传参模式和返回值的处理完全相同。同样都支持通配符的匹配和递归遍历，甚至支持在遍历时指定忽略一批文件。

参数格式也完全跟 `add_files()` 相同，因为 `add_files` 内部就是基于这几个接口实现的。

我们再来改下 xmake.lua 文件，在编译之前去遍历 `src` 目录，在完成编译之后去遍历 `build` 目录，例如。

```lua
target("ostest")
    set_kind("binary")
    add_files("src/*.cpp")
    before_build(function (target)
        print(os.filedirs("src/*"))
    end)
    after_build(function (target)
        for _, filepath in ipairs(os.files("$(buildir)/**")) do
            print(filepath)
        end
    end)
```

上面的配置中，我们通过 `os.filedirs("src/*")` 去遍历 src 目录下的所有文件，使用 `*` 通配符可以实现目录下文件名的任意匹配，而 `os.files("$(buildir)/**")` 则会通过 `**` 通配符来实现递归的模式匹配。

另外，使用 print 可以直接打印出返回的整个列表内容，我们也可以使用 lua 原生提供的 `for/ipairs` 去遍历 `os.files` 返回的 table 列表内容。

执行 `xmake` 编译查看输出内容，如下图。

![2](实验13系统操作详解.assets/0dbd01ee0aa214ba31771018d2272942-0)

图中所遍历出来的 `build/.deps` 路径是用来存储自动生成的头文件依赖文件信息，而 `build/.objs` 主要用于存储自动生成的 `.o` 对象文件。

#### 判断文件类型

如果有些路径，我们想知道它到底是文件类型还是目录类型，又或者是否真的存在，那么可以使用下面几个接口。

| 遍历接口  | 接口说明             |
| --------- | -------------------- |
| os.isdir  | 是否为目录           |
| os.isfile | 是否为文件           |
| os.exists | 目录或者文件是否存在 |

其中，`os.isdir` 和 `os.isfile` 除了判断路径类型外，还自带存在性判断，如果不存在的话，不管是否文件还是目录，这两个接口肯定是会返回 false。

这几个接口比较简单，我们就不再修改 xmake.lua 配置去操作了，直接执行下面的命令来快速测试验证这些接口。

```bash
xmake l os.isdir src
xmake l os.isfile src/main.cpp
xmake l os.exists build
```

`xmake l/lua` 插件会直接调用 `os.isfile` 等接口来脱离 xmake.lua 做一些快速的模块接口测试（关于这块我们会在后面的实验中详细讲解）。

这里如果运行正常，我们会在终端下看到三个 `true` 输出。



## 执行外部程序

关于 os 模块的另一个比较常用的操作就是运行外部程序，比如执行一些 shell 命令或者通过其它命令程序获取其运行的结果输出等等。

xmake 对此也提供了一系列接口来满足各种命令执行需求，具体主要有下面这些接口。

| 接口      | 接口说明                           |
| --------- | ---------------------------------- |
| os.run    | 静默运行程序                       |
| os.runv   | 静默运行程序，带参数列表           |
| os.exec   | 回显运行程序                       |
| os.execv  | 回显运行程序，带参数列表           |
| os.iorun  | 运行并获取程序输出内容             |
| os.iorunv | 运行并获取程序输出内容，带参数列表 |

从上面的列表可以看出，单纯运行的外部程序，也细分了各种使用场景对应的不同接口，具体差异已经在上面详细说明。

#### 回显运行程序

下面我们先来简单介绍和使用最简单的 `os.exec` 接口，修改 xmake.lua 配置文件。

```lua
target("ostest")
    set_kind("binary")
    add_files("src/*.cpp")
    after_build(function (target)
        os.exec("echo hello xmake")
        os.execv("bash", {"-c", "echo hello $WORD"}, {envs = {WORD = "xmake"}})
    end)
```

上面的配置中，我们在编译完成后通过 `os.exec` 调用 `echo hello xmake` 命令来回显输出 `hello xmake`，另外还使用了 `os.execv` 命令来调用 bash 子命令来实现相同的输出。

这两个接口在传参上稍微有一些区别，`os.exec` 是通过格式化字符串的方式传参，而 `os.execv` 是通过额外的参数列表传参，这种方式更加的灵活，并且还能在第三个参数中传递额外的可选信息。比如在执行过程中，我们可以同时传递 `WORD=xmake` 环境变量给子命令，使得 echo 可以正常显示输出 `$WORD` 变量的值。

执行 `xmake` 命令，查看编译后的输出信息，如果运行正常，应该会像下图所示，显示两行 `hello xmake` 字符串。

![3](实验13系统操作详解.assets/efeeb105904ddf59f802ff037a893fef-0)

#### 静默运行程序

有时候，我们在编译项目的过程中只是想静默执行一些外部命令，不回显子命令自身的输出内容，就可以使用 `os.run` 和 `os.runv` 接口来运行它们。

其使用方式完全跟 `os.exec` 和 `os.execv` 一致，仅仅只是回显和静默的区别。由于子命令运行过程中不会输出任何信息。因此，我们只有在运行失败后，才能看到实际的错误输出信息，而正常运行过程中，是不会显示任何输出信息的。

把 xmake.lua 改成如下配置内容来继续实验下这两个接口。

```lua
target("ostest")
    set_kind("binary")
    add_files("src/*.cpp")
    after_build(function (target)
        os.run("echo hello xmake")
        os.runv("echo", {"hello", "xmake"})
    end)
```

这个时候，我们执行 `xmake` 命令编译，除了自身编译输出外，不会再看到额外的 `hello xmake` 输出了，之前编译正常通过不中途报错，那么就说明 `os.run` 已经运行成功了。

#### 捕获外部命令执行结果

最后，我们再来讲解下 `os.iorun` 和 `os.iorunv` 的使用，这两个接口主要用于获取子命令运行完成后的输出结果，也就是获取它的 stdout 和 stderr 输出内容，方便在自定义脚本中做后续的处理。

它们的传参完全跟其它的执行接口一致，主要还是多了返回值的差别，用于获取运行结果，例如。

```lua
target("ostest")
    set_kind("binary")
    add_files("src/*.cpp")
    after_build(function (target)
        local stdout, stderr = os.iorun("echo hello xmake")
        print("stdout: %s", stdout)
        print("stderr: %s", stderr)
    end)
```

这里，我们通过运行 `os.iorun` 接口，运行完成 `echo hello xmake` 子命令后，获取了其输出内容，然后再将它们重新打印了出来。记住，`os.iorun` 自身也是静默运行的，并不会自动回显执行结果。

执行 `xmake` 编译后的输出内容如下图，stdout 用于返回标准输出，stderr 用于返回错误输出。

![4](实验13系统操作详解.assets/aae6a1bffd930572fdec287c28189a78-0)

再来个更加实际点的例子，通常的子命令输出是多行的，我们可以获取输出后，按行分隔然后遍历每行内容文本，并且显示对应的行号信息，例如。

```lua
target("ostest")
    set_kind("binary")
    add_files("src/*.cpp")
    after_build(function (target)
        local stdout = os.iorun("cat src/main.cpp")
        for idx, line in ipairs(stdout:split('\n')) do
            print("%d: %s", idx, line)
        end
    end)
```

这里，我们通过 `cat` 命令显示 `src/main.cpp` 文件的所有内容，然后我们通过 `os.iorun` 运行并捕获输出后，重新按行显示了出来。另外，我们还用到了 string 模块的 split 接口来按行分隔字符串。

具体运行 xmake 命令编译后的运行输出效果，如下图所示。

![5](实验13系统操作详解.assets/f3ea9b89265e82e5fb683c33d115b8c3-0)



## 实验总结



在本节实验中，我们主要学习了一些 os 模块的常用接口，比如如何在自定义脚本域中调用 os 模块接口去执行一些文件操作，也学习了如何在 xmake.lua 中执行外部命令。

本实验的参考代码可以使用如下命令下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/2764/code13.zip
```