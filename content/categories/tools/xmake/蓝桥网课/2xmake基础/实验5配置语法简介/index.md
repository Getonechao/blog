+++

title= "005配置语法简介"
description= "蓝桥网课xmake笔记"
date= 2021-07-15T10:36:41+08:00
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

# 配置语法简介

在本节实验中，我们可以大概了解 xmake.lua 的语法设计理念，以及一些基本配置的写法，通过本节实验，基本上能掌握如何去编写合理的 xmake.lua 配置来简化工程配置。

#### 知识点

- xmake.lua 的配置语法和设计理念
- 描述域和脚本域配置的基本概念
- 多目标程序的编译配置
- 全局配置作用域的概念



## 配置语法简介

xmake 的工程描述文件 xmake.lua 虽然基于 lua 语法，但是为了使得更加方便简洁得编写项目构建逻辑，xmake 对其进行了一层封装，使得编写 xmake.lua 不会像编写 makefile 那样繁琐，甚至比 cmake 的 CMakelists.txt 的 DSL 语法还更加的简洁直观，学习成本更低。

基本上写个简单的工程构建描述，只需三行就能完整构建一个 C/C++ 工程，例如。

```lua
target("test")
    set_kind("binary")
    add_files("src/*.c")
```

#### 配置分离

xmake.lua 采用二八原则实现了描述域、脚本域两层分离式配置。

什么是二八原则呢，简单来说，大部分项目的配置，80% 的情况下，都是些基础的常规配置，比如：`add_cxflags`、`add_links` 等，只有剩下不到 20% 的地方才需要额外编写一些复杂的逻辑脚本来满足一些特殊的配置需求。

而这剩余的 20% 配置通常比较复杂，如果直接充斥在整个 xmake.lua 里面，会把整个项目的配置弄的很混乱，不具有良好的可读性。

因此，xmake 通过描述域、脚本域两种不同的配置方式，来隔离 80% 的简单配置以及 20% 的复杂配置，使得整个 xmake.lua 看起来非常的清晰直观，可读性和可维护性都达到最佳。

我们可以通过下图大致了解哪些配置区域是描述域配置，哪些是脚本域。

![1](实验5配置语法简介.assets/cfb304cccd2edbf5960863d64c53297a-0)

也可以通过下面描述的特征来快速区分：

- 描述域：使用 `set_xx`、`add_xxx` 等配置接口进行的配置区域。
- 脚本域：使用 `on_xx`、`after_xxx` 和 `before_xxx` 等配置接口的内部区域。



## 描述配置域

对于刚入门的新手，或者仅仅是维护一些简单的小项目，只使用描述配置就已经可以完全满足需求了，例如下面的配置。

```lua
target("hello")
    set_kind("binary")
    add_files("*.c")
    add_defines("DEBUG")
    add_syslinks("pthread")
```

一眼望去，其实就是个 `set_xxx` 和 `add_xxx` 的配置集，而 `target("test")` 用来定义目标程序，所有的配置都会对当前定义的 target 目标生效。

对于新手，这样的配置，我们完全可以不把它当做 lua 脚本，仅仅作为普通的，但有一些基础规则的配置文件就行了，所以即大家完全没学过 lua 也是没有关系的。

这回我们不通过模板工程，直接手工创建一个空项目来实验描述域配置。首先新建一个空目录，然后写个输出指定字符串的 C 例子代码（如果之前已经存在 hello 目录，可以先执行 `rm -rf hello` 删除它）。

```bash
cd Code
mkdir hello
cd hello && vim main.c
```

然后编辑 main.c 编写如下代码。

```c
#include <stdio.h>

int main(int argc, char** argv)
{
    printf("hello %s\n", WORD);
    return 0;
}
```

C 代码写好后，在项目根目录下执行 `vim xmake.lua` 开始写配置文件，我们这里仅仅使用描述域配置。

```lua
target("hello")
    set_kind("binary")
    add_files("*.c")
    add_defines("WORD=\"foo\"")
```

所有的配置都会对 `target("hello")` 定义的 hello 目标程序生效，这里仅仅添加了一个 `-DWORD="foo"` 的宏定义。

最后，我们执行如下命令编译运行程序。

```bash
xmake
xmake run
```

![2](实验5配置语法简介.assets/12fc637dcdfbec4ebfb24a78ce15ddc4-0)

如果正常看到 `hello foo` 的输出，说明添加的宏定义配置确实正常生效了。

描述域虽然支持 lua 的脚本语法，但在描述域尽量不要写太复杂的 lua 脚本，比如一些耗时的函数调用和 for 循环，因为描述域目的主要是为了简化设置配置项。



## 脚本配置域

如果大家已经完全熟悉了 xmake 的描述域配置，并且感觉有些满足不了项目上的一些特殊配置维护，那么我们可以在脚本域做更加复杂的配置逻辑。

只要是类似：`on_xxx`、`after_xxx` 和 `before_xxx` 等字样的配置接口内部的脚本，都是属于脚本域。

继续修改 hello/xmake.lua 配置，在编译完成后输出目标程序的实际存储路径，配置修改如下。

```lua
target("hello")
    set_kind("binary")
    add_files("*.c")
    add_defines("WORD=\"foo\"")
    after_build(function (target)
        print("my path: %s", target:targetfile())
    end)
```

上面的配置中，`after_build()` 定义的内部代码区域就是脚本配置域，可以在里面写各种复杂的 lua 脚本来实现更灵活的配置需求。在这里，我们仅仅通过 `print` 内置接口去打印输出 `target:targetfile()` 的内容，也就是目标程序的实际输出路径。

执行 `xmake` 编译结束，会在编译最后面输出配置信息，例如下图的红框部分。

![3](实验5配置语法简介.assets/f070438befbb55a3b4d93237f88ef252-0)

xmake 还对每个 target 提供了 `on_load` 阶段，每个目标程序的配置都会加载这里面的脚本，因此我们可以在这个脚本域中，实现更加灵活复杂的基础配置脚本。

例如，我们可以把之前在描述域中定义的宏定义放置在 `on_load` 里面去动态配置。

```lua
target("hello")
    set_kind("binary")
    add_files("*.c")
    on_load(function (target)
        target:add("defines", "WORD=\"foo\"")
    end)
```

上面的配置跟之前在描述域的配置完全等价，任何 `set_xxx`、`add_xxx` 等描述域接口，在脚本域中都可以通过 target 里面的 `target:set("xxx", ...)` 和 `target:add("xxx", ...)` 模块接口，更加灵活的调用和配置。

我们可以再执行 `xmake -rv` 进行编译，查看编译输出，`-DWORD=\"foo\"` 也被传入的 gcc 编译器，说明动态的在脚本域设置宏定义也生效了。

![4](实验5配置语法简介.assets/5f64ef63423379dd9c5e02856bacd522-0)

#### 分离脚本域配置脚本

对于一些简短的配置脚本，像上面这样内置写就足够了，如果需要实现更加复杂的脚本配置逻辑，那么跟描述域配置充斥在一个 xmake.lua 里面，就会显得很臃肿。这个时候可以把脚本域的配置分离到独立的 lua 文件中去维护，这也是描述域和脚本域设计的初衷，可以把复杂的逻辑独立出来单独维护。

这里还是以刚才的配置为例，稍微做一些修改，把 `on_load()` 里面的所有配置独立到单独的 `modules/load.lua` 脚本文件中去，整个工程的目录结构如下。

```txt
.
├── main.c
├── modules
│   └── load.lua
└── xmake.lua
```

接下来使用 vim 编辑 `modules/load.lua` 文件，将之前 `on_load` 里面的配置挪进来，例如。

```lua
function main(target)
    target:add("defines", "WORD=\"foo\"")
end
```

可以看到，里面的配置跟之前的脚本没什么不同，仅仅加了个 `main` 函数作为主入口，而之前在 `on_load(function (target) end)` 中定义的是匿名的入口函数。

最后修改原来的 xmake.lua 配置，将 `on_load()` 里面的配置指向到刚刚创建的 `modules/load.lua` 文件即可。

```lua
target("hello")
    set_kind("binary")
    add_files("*.c")
    on_load("modules.load")
```

这里使用 `modules.load` 的格式去引用当前 modules 目录下的 load.lua 文件，其中每个 `.` 对应一级子目录。至于为什么不用路径分隔符，是为了跟 [import 导入接口](https://xmake.io/#/zh-cn/manual/builtin_modules?id=import) 保持一致，都是用来导入指定的 lua 脚本模块。关于 import 接口，我们会在后面的实验中详细讲解。

执行如下命令验证编译和运行是否通过。

```bash
xmake
xmake run
```

编译和运行是否通过可以对照下面的图片结果。

![2](实验5配置语法简介.assets/12fc637dcdfbec4ebfb24a78ce15ddc4-0)

## 多目标程序配置

在之前的实验中，我们的工程都只有一个目标程序，也就是仅仅定义了一个 `target()`，但实际的项目，可能需要同时编译多个目标程序。

这种情况下只需要在 xmake.lua 中再额外多配置一个 `target()` 的定义块就行了，每个 `target("test")` 定义唯一对应一个目标程序，它可以是可执行程序，也可以是静态库、动态库程序目标。

通过缩进的方式，可以显式区分哪些设置是对哪个 target 目标生效，当然缩进配置只是语法风格的约定，使得配置更加简洁，可读性更好，但这并不是强制要求的。

现在继续修改之前的 xmake.lua 配置，里面定义两个执行程序目标，而实际的 C 代码我们还是复用之前的 `main.c`。

```lua
target("test1")
    set_kind("binary")
    add_files("*.c")
    add_defines("WORD=\"TEST1\"")

target("test2")
    set_kind("binary")
    add_files("*.c")
    add_defines("WORD=\"TEST2\"")
```

通过上面的配置看到，我们还对每个 target 额外添加了不同的宏定义，`-DTEST1` 定义给 test1 目标程序，`-DTEST2` 定义给 test2 目标程序。

需要注意，所有的 `set_xx` 和 `add_xx` 设置都是针对上面最近的 `target()` 定义的特定目标程序，而不是全局的。

我们可以把它理解为字典的 key/value 设置，上面的配置等价于下面的伪描述 json 定义。

```json
"test1":
{
    "kind" = "binary",
    "files" = "*.c",
    "defines" = "WORD=\"TEST1\""
},
"test2"
{
    "kind" = "binary",
    "files" = "*.c",
    "defines" = "WORD=\"TEST2\""
}
```

也可以显式的加上 `target_end()` 来明确每个 target 定义块的作用范围，比如下面这样。

```lua
target("test1")
    set_kind("binary")
    add_files("*.c")
    add_defines("WORD=\"TEST1\"")
target_end()

target("test2")
    set_kind("binary")
    add_files("*.c")
    add_defines("WORD=\"TEST2\"")
target_end()
```

执行 `xmake` 来尝试编译这两个目标程序，可以看到，我们实际编译链接生成了两个可执行程序：test1 和 test2。

![5](实验5配置语法简介.assets/4235de0100dd314fbaeb3ab3addcf61e-0)

再执行 `xmake -rv` 查看详细输出，这两个宏定义已经分别传入了对应的 target 中去。

![6](实验5配置语法简介.assets/1828ef47b7de679af600383f4a08f335-0)

分别运行这两个可执行程序查看输出。

```bash
xmake run test1
xmake run test2
```

如果看到下面的输出，说明我们确实已经将不同的 WORD 宏定义分别传入了 test1 和 test2 这两个程序中去。

![7](实验5配置语法简介.assets/295a108a80e1e6af563217b32f6cfd95-0)

#### 全局配置和子配置

如果我们的项目工程需要参与编译的目标程序很多，不止一两个，有可能十几个，如果每个 target 都单独配置一遍，那么整个 xmake.lua 中会有很多的重复配置存在，非常冗余，毕竟不可能每个 target 的配置都是完全不同的，多少总会有一些共享的通用配置。

我们可以把每个 target 中都会使用到的通用配置抽离出来，放在最上面的全局配置域中设置，这样就会对所有 target 都生效了。

在之前的配置中，在全局根域加上一些通用配置，比如改成下面的内容。

```lua
add_defines("ROOT")
set_optimize("fastest")
set_languages("c99")
add_includedirs("/tmp")

target("test1")
    set_kind("binary")
    add_files("*.c")
    add_defines("WORD=\"TEST1\"")

target("test2")
    set_kind("binary")
    add_files("*.c")
    add_defines("WORD=\"TEST2\"")
```

执行 `xmake -rv` 进行编译，从输出中看到红框里面的部分，可以确认在全局根域的这些设置对每个 target 都生效了，而 `-DWORD=` 的宏定义设置在不同的 target 中还是不相同的，因为它们是在特定 target 域中分别设置的。

![8](实验5配置语法简介.assets/85fba208313c73bb363be47390413fd0-0)

至于为什么要抽离通用设置到全局根域，在前面也解释了原因，就是为了简化 xmake.lua 的配置，这也是 xmake 的初衷，让工程项目的构建配置能够尽可能的简洁可读，这样方便维护也更加友好。

如果不放在根域，而是每个 target 都去单独配置一遍，我们可以对比下看看，这是非常冗余臃肿的，可读性也很差，就如下面的配置（不要这样去写）。

```lua
target("test1")
    set_kind("binary")
    add_files("*.c")
    add_defines("WORD=\"TEST1\"")
    add_defines("ROOT")
    set_optimize("fastest")
    set_languages("c99")
    add_includedirs("/tmp")

target("test2")
    set_kind("binary")
    add_files("*.c")
    add_defines("WORD=\"TEST2\"")
    add_defines("ROOT")
    set_optimize("fastest")
    set_languages("c99")
    add_includedirs("/tmp")
```

所以，我们要记住，如果有通用的设置，请尽可能设置到全局作用域中对所有 target 生效，而特殊配置才在指定的 target 中去单独配置它们。





## 实验总结

在本节实验中，我们了解了 xmake.lua 的语法设计理念，以及基本的配置方式，并且学习了什么是描述域和脚本域，也学习了如何去配置编译多个目标程序，另外还知道了怎么通过全局设置去简化配置内容。

本实验的参考代码可以使用如下命令下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/2764/code5.zip
```