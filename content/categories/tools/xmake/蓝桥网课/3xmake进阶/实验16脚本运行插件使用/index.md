+++

title= "实验16脚本运行插件使用"
description= "蓝桥网课xmake笔记"
date= 2021-07-15T10:37:01+08:00
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

# 脚本运行插件使用

在本节实验中，我们会学习如何使用 `xmake lua` 插件来更快速地运行 lua 脚本和模块接口，使我们能够更高效的调试自定义配置脚本。

#### 知识点

- 交互式运行（REPL）的基本概念和使用
- 指定 lua 脚本的运行
- 模块接口的快速运行
- 字符串脚本片段的运行



## 脚本运行插件介绍

在上两节实验中，我们已经简单熟悉了如何在自定义脚本中去编写更加灵活的配置脚本，不过每次都需要修改 xmake.lua 然后运行 `xmake` 去编译才能运行到里面我们编写的脚本代码，如果是在 `after_build` 等阶段，那每次测试还得等到完整编译通过。

整个调试过程会非常的低效，因此为了方便用户和开发者调试和快速验证 xmake 内部的各种模块，以及用户自定义的扩展模块。

xmake 提供了一个 `xmake l/lua` 插件，可以快速的运行我们提供的独立 lua 脚本、脚本片段代码，甚至还可以直接调用指定的模块接口或者开启交互运行模式来快速测试和验证。

#### 运行交互命令（REPL）

首先我们来讲解和使用 `xmake lua` 插件提供的交互运行模式，也就是 REPL。简单的说，它就是一种交互式解释执行环境 R（read）、E（evaluate）、P（print）、L（loop）。用户输入值，交互式解释器会读取输入内容并对它求值，再返回结果，并重复此过程，因此我们可以实时执行自己的 lua 脚本然后快速获取运行结果。

在交互模式下，运行命令可以更加方便地测试和验证一些模块接口，也更加地灵活，不需要去额外写一个脚本文件来加载。

我们只需要执行 `xmake lua` 命令就可以进入交互模式，例如。

```bash
$ xmake lua
xmake>
```

执行后的效果图如下，如果我们正常看到 `xmake>` 提示用户输入的前缀字符串，那么说明现在已经成功进入交互模式了。

![1](实验16脚本运行插件使用.assets/10c6affd787ab358434d4c6030f23a84-0)

接下来，我们可以尝试做一些基础操作直观感受下，比如进行表达式计算。

```bash
xmake> 1 + 2
< 3
```

运行完，会输出计算结果 3，然后我们再来尝试下变量的赋值和打印。

```bash
xmake> a = {1, 2, b = 3}
xmake> a
< {
    1,
    2,
    b = 3
  }
```

我们对变量 a 赋值了一个 table 对象，然后敲 a 就可以将其内容序列化打印出来。

还可以执行多行输入实现 for 循环迭代，只需要正常输入 for 循环体的每行代码，然后按回车，就会自动提示其它行的输入，直到整个 `for-do-end` 完成输入。

```bash
xmake> for _, v in pairs({1, 2, 3}) do
xmake>> print(v)
xmake>> end
1
2
3
```

如果我们看到前置提示符变成了 `xmake>>`，说明现在进入了多行输入模式。

对于刚刚的一些输入测试，整个运行效果如下图。

![2](实验16脚本运行插件使用.assets/df3712c28550edbb7799bc931feadf04-0)

注：如果在多行输入中，我们想取消继续执行，退出多行模式，只需要按 `q` 后回车即可，例如。

```bash
xmake> for _, v in ipairs({1, 2}) do
xmake>> print(v)
xmake>> q         # 取消多行输入，清空先前的输入数据
xmake> 1 + 2
< 3
```

我们也能够通过 `import` 来导入各种扩展模块，直接调用里面接口做相关测试，例如上个实验中的 mymod.lua 里面对 `lib.detect.find_tool` 模块的使用，这里可以直接在交互模式下导入后调用运行。

```bash
xmake> find_tool = import("lib.detect.find_tool")
xmake> find_tool("gcc")
< {
    program = "/usr/bin/gcc",
    name = "gcc"
  }
```

运行效果如下图。

![3](实验16脚本运行插件使用.assets/198ad7c745f4cdf12b2a9ad890e0ad96-0)



## 执行指定 lua 脚本

虽然交互模式运行很方便，但是对于我们自己写的扩展模块脚本，基本上都是独立的 lua 文件，如果我们能够直接去运行它们，那会更加的方便，并且绕过了 xmake.lua 和编译过程。

如果指定的 lua 模块脚本里面提供了 `main()` 入口函数，那么我们可以通过运行下面的命令直接加载运行它，还是以上节实验中我们创建的 `import_test` 工程中的 mymod.lua 为例，这里尝试直接执行它。

```bash
# 如果没有保存上一个实验的代码，可以通过如下命令下载使用
wget https://labfile.oss.aliyuncs.com/courses/2764/code15.zip
cd ~/Code/import_test
xmake lua xmake_modules/mymod.lua
```

执行效果如下图，可以看到 `find_tool("gcc")` 的执行结果已经正常了，但是 `print("mymod: %s", str)` 的执行由于缺少参数，这里仅仅显示 `%s`。

![4](实验16脚本运行插件使用.assets/4b6bcb8081f459f97958a6ba05448463-0)

为了传递参数到 mymod.lua 模块文件中的 `main(str)` 入口参数，我们只需要追加参数到 `xmake lua` 命令的最后面，例如。

```bash
xmake lua xmake_modules/mymod.lua hello
```

传入参数后，我们就可以看到下图红线部分的参数内容了。

![5](实验16脚本运行插件使用.assets/d219dc4be3df15eb63da34d64c7d4060-0)

#### 执行指定 lua 字符串脚本

`xmake lua` 除了能够直接加载运行 lua 脚本文件外，还可以通过传递 `-c` 参数来直接执行一段简短的字符串脚本，例如下面的命令。

```bash
xmake lua -c 'for i = 1, 10 do print(i) end'
```

如果运行成功，我们会看到实际的输出结果，如下图。

![6](实验16脚本运行插件使用.assets/d7b71309c599e5da73298b19e17a657c-0)

#### 快速调用和测试模块接口

最后，我们还可以使用 `xmake lua` 来直接调用 xmake 提供的任何内置模块的接口，来快速获取输出结果，例如运行下面的命令去查找 gcc 编译器路径信息。

```bash
xmake lua lib.detect.find_tool gcc
```

结果如下。

![7](实验16脚本运行插件使用.assets/a6ecf135aa8d6c5516e1f21f4ca659d9-0)

也可以执行下面的命令，去快速探测安装的 C/C++ 依赖包，例如。

```bash
xmake lua find_packages zlib
```

上面的命令，我们通过直接执行 `find_packages` 接口去查找 zlib 包，如果检测成功，就会看到实际的 zlib 库信息。

![8](实验16脚本运行插件使用.assets/100a9fdedb7d73d498de24921090bd7d-0)

又或者，我们可以直接调用 os 模块里面的接口，比如快速遍历获取 src 目录下 C++ 文件列表。

```bash
cd ~/Code/import_test
xmake lua os.files 'src/*.cpp'
```

我们进入之前的 import_test 工程目录，然后获取到 src 的所有源文件，结果如下图。

![9](实验16脚本运行插件使用.assets/03bffcb0fed4c1868cd32804efb2d19d-0)



## 实验总结

在本节实验中，我们学习了如何使用 `xmake lua` 插件来快速运行指定的 lua 脚本模块，代码片段以及模块接口，方便大家快速进行模块脚本调试，另外我们还学习到了如何进入交互模式来执行一些脚本逻辑。

本实验的参考代码可以使用如下命令下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/2764/code16.zip
```