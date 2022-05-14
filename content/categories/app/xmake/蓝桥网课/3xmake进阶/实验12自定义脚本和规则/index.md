+++

title= "实验12自定义脚本和规则"
description= "蓝桥网课xmake笔记"
date= 2021-07-15T10:36:51+08:00
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

# 自定义脚本和规则

在本节实验中，我们会更加深入地了解什么是自定义脚本，以及如何去拦截 C/C++ 构建的整个处理流程，并且还能学习到如何通过 `rule()` 去自定义构建规则来处理未知的文件。

#### 知识点

- 什么是自定义脚本
- 构建过程的几个基本处理阶段
- 构建过程每个阶段的拦截和处理
- 自定义构建规则的基本使用



## 自定义脚本介绍

在之前的实验中，我们一直都是在 xmake.lua 的描述域中进行配置，这对于大部分项目而言，已经足够满足需求，但是对于一些更加复杂的项目，仅仅通过描述域的配置逻辑是不够用的。我们需要更加灵活复杂的配置逻辑，能够处理一切的用户定制化配置需求，这个时候就得在 xmake 的自定义脚本域中进行配置了。

关于什么是描述域，什么是自定义脚本域，我们在实验五《xmake 基础之配置语法简介》中，已经详细介绍过了，这里再来简单的复习下：

- 描述域：`set_xxx`、`add_xxx` 相关的配置接口所在的作用域。
- 脚本域：`on_xxx`、`after_xxx` 和 `before_xxx` 接口函数内部的作用域

我们可以从下面的配置注释中，大概了解下。

```lua
-- 描述域
target("test")
    set_kind("binary")
    add_files("src/*.cpp") -- 描述域
    on_load(function (target)
        -- 脚本域
    end)
    after_build(function (target)
        -- 脚本域
    end)
```

#### 在构建阶段执行特定脚本

xmake 的 `target()` 目标配置块中，我们可以设置一些自定义的脚本域，在 xmake 构建的各个阶段中处理一些用户的自定义逻辑。我们可以把它们理解为类似 C/C++ 中的 Hook (钩子)。相当于在 xmake 的整个构建中，注入一些自定义的 lua 脚本代码。

例如，我们可以在指定 target 目标程序编译前后执行一些自定义的脚本操作。在开始本实验前，我们先执行下面的命令创建一个新的空工程。

```bash
cd Code
xmake create script_test
```

然后进入 script_test 文件夹，开始编辑里面的 xmake.lua 文件，在构建的前后阶段加上自己的一些自定义脚本逻辑，例如。

```lua
target("test")
    set_kind("binary")
    add_files("src/*.cpp")

    before_build(function (target)
        print("start building %s", target:targetfile())
    end)

    after_build(function (target)
        print("finish building %s", target:targetfile())
    end)
```

上面的配置中，我们额外配置了 `before_build` 和 `after_build` 两个自定义的 lua 脚本域，用于在当前目标程序的构建开始和结束两个阶段，打印一些信息。

执行 `xmake` 编译信息，会看到下图的结果，红线位置部分就是我们在脚本中自定义显示的输出信息。

![1](实验12自定义脚本和规则.assets/266bbc62c52bb9359e6662dcd8ebb337-0)

这里，出于演示目的，我们仅仅只是简单的显示了目标程序的实际生成路径，而在真正的项目中，大家可以在里面做更加复杂的事情。

另外，在每个自定义脚本的配置入口，我们可以设置额外的平台、架构匹配模式，使得仅仅在特定平台触发脚本，例如将 xmake.lua 文件改成下面的配置。

```lua
target("test")
    set_kind("binary")
    add_files("src/*.cpp")

    on_build(function (target)
        print("build %s", target:targetfile())
    end)

    on_build("linux", function (target)
        print("build %s on $(plat)", target:targetfile())
    end)

    on_build("windows", function (target)
        print("build %s on $(plat)", target:targetfile())
    end)

    on_build("macosx", function (target)
        print("build %s on $(plat)", target:targetfile())
    end)
```

上面的配置，我们通过自定义 `on_build` 配置接口，完全替换内置的构建规则，也就是说，当前这个目标程序如果不去实现 `on_build`，那么整个编译过程就是空操作，因为 xmake 完全将如何去编译它的任务托管给了我们设置的 `on_build` 来实现。

这对于一些并不想采用内置编译逻辑的特殊目标程序，或者一些压根不是 C/C++ 程序但是也需要处理一些编译、预处理逻辑的其它程序而言，就非常有用了。

这里还有一个需要注意的地方是，我们定义了四个 `on_build`，而不是一个，下面的三个传递了额外的平台参数，用来告诉 xmake，只有匹配到实际的编译平台才会被执行到，也就是说，如果我们现在的需要编译的是 Linux 程序，实际上仅仅只会执行到 `on_build("linux", ...)` 的入口脚本，另外三个是不会被执行到的。

而最上面不带平台参数的 `on_build()` 则是默认的通用设置，仅仅只有当前的平台完全不是下面配置的 `linux`，`windows` 以及 `macosx` 三者之一，那么才会进入默认的通用入口脚本。

其实我们也可以用在通用的入口脚本中，使用 `if is_plat() then` 的方式来实现相同的目的，但这样看上去不是很精简，具体如何配置还是得看大家实际的需求来决定。

接下来，继续执行一遍 `xmake` 查看实际的编译输出。

![2](实验12自定义脚本和规则.assets/f5e59d7b0560becb6b12c86401cb4081-0)

由于我们通过 `on_build` 重写了内部构建逻辑，并且仅仅只是显示一行输出，因此这回的编译其实除了那一行打印输出外没有做任何事情。

但是，从这行输出信息中，我们可以看到，当前其实执行的是 `on_build("linux", ...)` 里面配置的自定义脚本，因为实验默认平台是 Linux 系统。

#### 在源文件编译阶段执行特定脚本

虽然，`on_build`、`before_build` 等阶段配置可以自定义构建逻辑，但是粒度相对还是太粗了，其实我们可以更加细粒度的在每个源文件的编译阶段，添加自定义的脚本逻辑。

例如，我们想在每个 C/C++ 源文件编译之前的阶段，做一些自定义的预处理操作，比如替换代码里面的特定文本等等。继续修改之前的 xmake.lua 改成下面的配置内容。

```lua
target("test")
    set_kind("binary")
    add_files("src/*.cpp")
    before_build_file(function (target, sourcefile)
        io.gsub(sourcefile, "hello world", "hello xmake")
    end)
```

这次我们做了一件稍微有点实际意义的事情，就是在编译每个 C/C++ 代码之前，通过 `io.gsub` 接口预先将里面的代码中包含有 `hello world` 的文本替换成了 `hello xmake`，然后继续参与编译。

由于我们是通过 `xmake create` 创建的空工程，默认生成的 `src/main.cpp` 里面会打印 `hello world` 字符串，可以执行 `cat ./src/main.cpp` 确认下。

![3](实验12自定义脚本和规则.assets/92c7b9424872566180f022f262c3aeba-0)

而当我们执行完 `xmake` 编译任务后，再执行 `xmake run` 命令查看实际的运行结果，会发现里面的输出字符串已经了变成 `hello xmake`，如下图所示。

![4](实验12自定义脚本和规则.assets/383b716ffcefb86eef29beee1159e995-0)

除了上面配置中的 `before_build_file`，其实跟 `on_build` 类似，我们还有对应的 `on_build_file` 配置来重写对单个源文件的编译逻辑，以及在编译后执行 `after_build_file` 中配置的自定义脚本，用法完全相同，因此这里就不重复讲解了。

#### 其它自定义脚本配置阶段

除了在构建阶段，我们可以插入一些自己的脚本逻辑外，还可以通过配置注入到其它的操作阶段。因为 xmake 除了构建，还有清理、安装、卸载、打包等其它的基本操作。

这里可以简单列举下，目前我们可以注入和修改的一些基本操作阶段。

| 自定义脚本配置接口 | 描述                           |
| ------------------ | ------------------------------ |
| on_load            | 自定义目标加载脚本             |
| on_link            | 自定义链接脚本                 |
| on_build           | 自定义编译脚本                 |
| on_build_file      | 自定义编译脚本, 实现单文件构建 |
| on_build_files     | 自定义编译脚本, 实现多文件构建 |
| on_clean           | 自定义清理脚本                 |
| on_package         | 自定义打包脚本                 |
| on_install         | 自定义安装脚本                 |
| on_uninstall       | 自定义卸载脚本                 |
| on_run             | 自定义运行脚本                 |

其中，除了 `on_load` 以外，其它的所有 `on_xxx` 阶段配置接口，都有对应的 `before_xxx` 和 `after_xxx` 版本，在前后阶段配置执行自定义脚本。

为了了解 xmake 是在哪些阶段实际执行了这些配置脚本，我们修改之前的 xmake.lua，把上面的不同阶段脚本都配置上，例如。

```lua
target("test")
    set_kind("binary")
    add_files("src/*.cpp")

    before_build(function (target)
        print("before_build")
    end)
    after_build(function (target)
        print("after_build")
    end)

    before_build_file(function (target, sourcefile)
        print("before_build_file: %s", sourcefile)
    end)
    on_build_file(function (target, sourcefile)
        print("on_build_file: %s", sourcefile)
    end)
    after_build_file(function (target, sourcefile)
        print("after_build_file: %s", sourcefile)
    end)

    before_link(function (target)
        print("before_link")
    end)
    on_link(function (target)
        print("on_link")
    end)
    after_link(function (target)
        print("after_link")
    end)
```

上面的配置代码，我们配置注入了大部分阶段的脚本处理，而 `on_run`、`on_install` 这种，很明显就是对应 `xmake run` 和 `xmake install` 等基本操作的处理，这里就不列举上去，不然配置太长了，大家可以自己再做一些相关尝试。

这里我们主要是通过配置，拦截整个构建流程中会执行到的一些步骤，比如文件编译、链接阶段等等，配置完成后，执行 `xmake` 查看整个编译输出。

![5](实验12自定义脚本和规则.assets/e60b699cd1632efe269640747e1e75a8-0)

通过上图的输出，我们可以大概看出整个构建的执行流程。这里会优先执行 `before_build`，然后会执行每个源文件的编译，接着执行链接过程，最后是 `after_build`。这里我们没有设置 `on_build`，是因为如果设置了这个接口，就会重写内部的所有构建过程，也就不会执行 `on_build_file` 和 `on_link` 了，一切构建操作都需要大家自己去完成。

下图描述了整个构建过程中每个阶段的执行时机。

![6](实验12自定义脚本和规则.assets/6d21c9b63e00679ed04110c2d4931c92-0)

## 通过定义规则处理其它文件



虽然我们可以在 target 中通过 `on_build_file` 等自定义脚本来定制化处理编译逻辑，但是如果每个 target 都要配置相同的处理逻辑，或者要处理多个不同种类的逻辑，并且全在 target 中去定义实现它们，那么整个配置会非常臃肿并且难以维护。

因此，xmake 提供了一种模块化的方式，也就是通过 `rule()` 自定义规则，然后使用 `add_rules()` 接口实现对单个 target 目标同时配置生效多个不同的编译规则。这些规则可以复用、对多个 target 生效。

而之前我们在 target 中的那些自定义脚本阶段的配置接口，例如 `on_build` 等，都可以在 `rule()` 中定义和实现。其实 xmake 内部的 C/C++ 代码编译本身也是使用 `rule("c++")` 规则来实现的，只不过默认就已经对 C/C++ 代码生效了，而用户在 xmake.lua 中自己定义配置的规则，就需要手动添加 `add_rules` 到对应的 target 才能生效。

直接这么说，也许还不太能很清晰理解什么是自定义规则，我们先来改造下之前的配置文件 xmake.lua，通过定义 `rule()` 的方式，来实现相同的目的，例如。

```lua
rule("myrule")
    before_build(function (target)
        print("before_build")
    end)
    after_build(function (target)
        print("after_build")
    end)

    before_build_file(function (target, sourcefile)
        print("before_build_file: %s", sourcefile)
    end)
    on_build_file(function (target, sourcefile)
        print("on_build_file: %s", sourcefile)
    end)
    after_build_file(function (target, sourcefile)
        print("after_build_file: %s", sourcefile)
    end)

    before_link(function (target)
        print("before_link")
    end)
    on_link(function (target)
        print("on_link")
    end)
    after_link(function (target)
        print("after_link")
    end)
rule_end()

target("test")
    set_kind("binary")
    add_files("src/*.cpp")
    add_rules("myrule")
```

我们通过抽离 target 中的所有自定义脚本到 `rule("myrule")` 中，然后通过 `add_rules("myrule")` 的方式对指定的 target 生效。

执行 `xmake -r` 重新编译看看，能输出哪些信息。

![7](实验12自定义脚本和规则.assets/8e98a13fe9250ae8f59c2a47282181b7-0)

我们看到 `before_build` 还有 `on_link` 等相关的阶段都正常输出了，跟之前完全一样，但是 `on_build_file` 等阶段，还是没有输出。这是因为通过 `add_rules()` 的方式应用的规则，默认是不会去覆盖内部的 `rule("c++")` 规则中的 cpp 文件编译，只是额外附加了一些自定义脚本。

但是，如果是处理一些其它的未知文件，比如：`*.md` 等，由于 xmake 内部没有相关的规则可以识别处理，那么 `rule("myrule")` 中的 `on_build_file` 也就会被执行到。

因此，我们先执行下面的命令，创建一个 `src/test.md` 文件用于定制化处理。

```bash
touch src/test.md
```

然后修改 xmake.lua 配置，通过添加 `set_extensions(".md")` 到 `rule("myrule")` 使得 myrule 规则可以识别到新创建的 `src/test.md` 文件。

```lua
rule("myrule")
    set_extensions(".md") -- 指定这个规则仅对 md 文件生效
    before_build(function (target)
        print("before_build")
    end)
    after_build(function (target)
        print("after_build")
    end)

    before_build_file(function (target, sourcefile)
        print("before_build_file: %s", sourcefile)
    end)
    on_build_file(function (target, sourcefile)
        print("on_build_file: %s", sourcefile)
    end)
    after_build_file(function (target, sourcefile)
        print("after_build_file: %s", sourcefile)
    end)

    before_link(function (target)
        print("before_link")
    end)
    on_link(function (target)
        print("on_link")
    end)
    after_link(function (target)
        print("after_link")
    end)
rule_end()

target("test")
    set_kind("binary")
    add_files("src/*.cpp", "src/*.md") -- 添加上 md 文件
    add_rules("myrule")
```

正如上面的配置中看到，我们也需要在 xmake.lua 中将 `src/test.md` 通过 `add_files` 添加进来，这样 myrule 规则才能识别到它。

接着执行 `xmake -r` 查看输出，如果一切顺利，这回我们会看到 `on_build_file` 也被执行到了，用于 `.md` 文件的定制化处理，而 c++ 文件不受任何影响。

![8](实验12自定义脚本和规则.assets/02a0db9845f32816142b8e7bfcf4bc18-0)

这里，我们仅仅只是简单的调用了 `print` 接口，实际的项目配置过程中，可以通过自定义的 `rule()` 规则配置，实现除了 c++ 代码外的其它任何未知文件的处理和编译，甚至可以扩展编译其它的编译语言的代码文件。

#### 在自定义规则中配置目标编译选项

在实验五《xmake 基础之配置语法简介》中，我们知道可以通过 `on_load` 自定义脚本在 target 中更加灵活的配置各种编译选项，不会再受到描述域的一些限制。而在自定义规则中，我们也可以通过定义 `on_load`，在里面动态添加一些编译选项给个 target 程序。

我们还是拿刚才的工程项目中 myrule 为例，修改下 xmake.lua。

```lua
rule("myrule")
    on_load(function (target)
        target:add("defines", "MYRULE")
        target:add("syslinks", "pthread")
        if is_plat("linux") then
            target:add("defines", "LINUX")
        end
    end)
rule_end()

target("test")
    set_kind("binary")
    add_files("src/*.cpp")
    add_rules("myrule")
```

这里，我们仅仅对 `myrule` 规则添加了 `on_load` 自定义脚本，并在里面动态的设置了一些宏定义和链接库，例如 `-DMYRULE`、`-DLINUX` 等等。

然后只需要通过 `add_rules("myrule")` 将其生效到指定的 target 上去就行了。

执行 `xmake -rv` 编译，可以看到输出结果中，已经加上了在 `on_load` 中新加的一些额外编译选项。

![9](实验12自定义脚本和规则.assets/414c11bb460769cce0b17bc0e9418b2b-0)

## 实验总结



在本节实验中，我们学习了如何配置自定义脚本在拦截目标程序的整个构建过程，实现一些定制化的处理逻辑，并且大概了解了构建的几个基本执行阶段，也学会了如何通过自定义编译规则去处理其它未知文件。

本实验的参考代码可以使用如下命令下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/2764/code12.zip
```