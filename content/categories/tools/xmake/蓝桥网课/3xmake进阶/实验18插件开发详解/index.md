+++

title= "实验18插件开发详解"
description= "蓝桥网课xmake笔记"
date= 2021-07-15T10:37:08+08:00
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

# 插件开发详解

在本节实验中，我们会学习到如何去编写和运行一个基于 xmake 的扩展插件，另外我们还会学习到如何在 xmake.lua 中去配置定义内部任务，以及如何去直接调用运行它。

#### 知识点

- xmake 插件和任务的基本概念
- 如何编写和运行 xmake 扩展插件
- 内部 task 任务的定义和运行

## xmake 插件任务介绍



除了 `option()` 和 `target()` 两个对象配置域，xmake 还提供了另外一种配置域 `task()`，通过它我们可以自定义一些扩展的运行任务，甚至可以用来实现一些扩展插件。

而在上节实验中，我们通过 xmake 内置的工程生成插件，其实也已经大概了解了 xmake 插件是什么以及如何使用了。

所谓的插件，就是除了 xmake 提供的一些内置基本子命令，例如 `xmake build`、`xmake install` 之外的其它子命令，大家可以在 xmake.lua 中通过 `task("myplugin")` 自定义一个插件命令，然后通过 `xmake myplugin` 的方式运行我们定义的插件子命令。

#### 编写一个简单的插件

接下来，我们通过实现一个简单的 `echo` 插件，来了解下整个插件编写过程。由于我们是在自己项目工程的 xmake.lua 里面去编写插件任务，因此先执行下面的命令，创建一个空工程。

```bash
cd ~/Code
xmake create plugin_test
```

创建完成后，进入 plugin_test 目录，编辑里面的 xmake.lua 文件，修改成如下配置，在里面定义一个 `task("echo")` 插件任务配置域用来描述 echo 插件。

```lua
task("echo")
    set_menu {usage = "xmake echo [options]", description = "Echo the given info!", options = {}}
    on_run(function ()
        print("hello xmake!")
    end)

target("test")
    set_kind("binary")
    add_files("src/*.cpp")
```

通过上面的配置可以看到，整个插件的定义很简单，主要由以下三部分组成。

- `task()`：定义整个插件任务配置域。
- `set_menu()`：用于配置插件在 help 命令菜单中的显示和参数描述。
- `on_run()`：定义整个插件的脚本运行入口。

配置完成后通过运行 `xmake --help` 命令，查看 xmake 帮助菜单，找寻我们定义的 echo 插件，看看是否出现在帮助列表里面。

![1](实验18插件开发详解.assets/d55805b6d966f1c51e81af297b1ac558-0)

在上图的红框位置，就是我们新定义的插件，不过当前没有被划分到 `Plugins` 里面，而是在 `Tasks` 下面，这个只是 `task()` 的分类问题，默认定义的 `task()` 既不属于 Actions 也不属于 Plugins 分类，而是作为通用的 Tasks 分类。

当然，这仅仅只是 help 菜单显示分类的差异，并不影响我们把它作为插件来使用，不过我们也可以通过 `set_category("plugin")` 将其标记为 Plugins 分类。

继续编辑 xmake.lua 文件。

```lua
task("echo")
    set_menu {usage = "xmake echo [options]", description = "Echo the given info!", options = {}}
    set_category("plugin")
    on_run(function ()
        print("hello xmake!")
    end)
```

接下来，我们就可以尝试运行插件了，只需要执行下面的命令。

```bash
xmake echo
```

如果运行正常，就能执行到我们定义在 `on_run()` 中的脚本，然后回显 `hello xmake!` 字符串信息，如下图。

![2](实验18插件开发详解.assets/d7e1920f835ba78d7ef0cada483d5abf-0)

#### 传递外部参数给插件

刚刚的例子中，我们在插件里面硬编码字符串来显示，而在实际的插件中，通常我们需要能够通过传递参数的形式，来动态地将一些信息传入插件中去。

比如，改造下刚刚的 echo 插件，将显示的字符串内容，通过 `xmake echo "hello xmake!"` 的参数传递方式，动态地显示出来。

为此，我们需要修改之前的 xmake.lua 配置，改成如下内容。

```lua
task("echo")
    set_menu {usage = "xmake echo [options]", description = "Echo the given info!", options = {
        -- 定义参数列表
        {nil, "contents", "vs", nil, "The info contents." }
    }}
    on_run(function ()
        import("core.base.option")
        print("%s", table.concat(option.get("contents") or {}, " "))
    end)

target("test")
    set_kind("binary")
    add_files("src/*.cpp")
```

这回，我们在 `task("echo")` 插件任务的 options 参数选项列表配置中，加入了一项参数定义配置，用于获取 `xmake echo` 命令之后的所有参数值。

而在 `on_run` 里面，通过 import 导入了 xmake 提供的内置扩展模块 `core.base.option` 来获取用户输入的参数选项，也就是 `option.get("contents")`。

由于我们定义的 `contents` 参数属于 `vs` 类型参数，也就是值列表参数，因此会获取到 `xmake echo arg1 arg2 ...` 后面的所有参数列表。然后通过 `table.concat` 接口，将其拼接成字符串后显示出来。

配置修改完成后，通过 `xmake echo --help` 查看下 echo 插件的帮助菜单，看下能否看到新增的参数选项，如下图。

![3](https://doc.shiyanlou.com/courses/2764/27526/1b3e4418ebfb354ae3833b67fcae46eb-0)

在图中最下面，我们看到了新增的 `contents` 参数选项，后面显示的 `...` 说明此选项是 `vs` 多值列表类型。

然后，我们重新执行下这个插件，传递参数进去看下效果。

```bash
xmake echo 'hello xmake!' # 注：此处单引号
xmake echo hello xmake!
```

执行上面两个命令后，都能正常输出 `hello xmake!` 字符串信息，第一个命令仅仅只传递一个字符串参数，而第二个命令是传递了一个字符串参数列表，也就是 `{"hello", "xmake!"}`，然后在 `on_run` 里面通过 `table.concat` 拼接成了最终的字符串：`hello xmake!`。

但不管哪种参数传递，其最终的输出结果都是相同的，如下图所示。

![4](实验18插件开发详解.assets/103c431229660800f1a3f4af534e9f55-0)

#### 插件参数选项详解

刚刚我们通过添加下面的配置行给 `options = {}` 来新增了一个插件参数选项，接下来将详细讲解这个参数定义列表的每个配置值的含义。

```lua
{nil, "contents", "vs", nil, "The info contents." }
```

在介绍之前，先对参数类型做一下分类，也就是上述配置中第三个值 `vs` 所定义的类型，主要有以下几类。

- `k`：单纯的 key 类型参数，例如：`-k/--key` 这种传参方式，通常用于表示 bool 值状态。
- `kv`：键值类型传参，例如：`-k value` 或者 `--key=value` 的传参方式，通常用于获取指定参数值。
- `v`：单个值传参，例如：`value`，通常定义在所有参数的最尾部。
- `vs`：多值列表传参，也就是刚刚我们配置的参数类型，例如：`value1 value2 ...` 的传参方式，可以获取多个参数值作为列表输入。

而第一个参数值，主要用于设置参数名的简写形式，例如：`--key` 对应的 `-k` 简写，这里没有用到，所以设置了 nil，第二个 `contents` 就是参数全名，第四个配置值是参数的默认值，最后一个值就是参数的描述字符串。

为了更加深入了解插件参数的配置，我们再对 echo 插件新增两个参数，`-c v/--color=value` 参数用于指定输出文本的颜色，而 `-b/--bright` 用于启用文本的高亮输出。

相关的参数配置修改如下。

```lua
task("echo")
    set_menu {usage = "xmake echo [options]", description = "Echo the given info!", options = {
        {'b', "bright", "k",  nil, "Enable bright."},
        {'c', "color", "kv", "black", "Set the output color.",
                                      "    - red",
                                      "    - blue",
                                      "    - yellow",
                                      "    - green",
                                      "    - magenta",
                                      "    - cyan",
                                      "    - white"},
        {},
        {nil, "contents", "vs", nil, "The info contents." }
    }}
    on_run(function ()
        import("core.base.option")
        local color = option.get("color")
        local bright = option.get("bright")
        local content = table.concat(option.get("contents") or {}, " ")
        if color ~= "black" or bright then
            cprint("${%s %s}%s", color, bright and "bright" or "", content)
        else
            print("%s", content)
        end
    end)

target("test")
    set_kind("binary")
    add_files("src/*.cpp")
```

通过上面的配置可以看到，新增的 `-b/--bright` 参数主要用于标识 bool 状态，所以仅仅设置了 `k` 参数类型，而 `-c v/--color=value` 是属于键值对参数类型 `kv`，另外我们还给它配置了默认值，也就是 `-c black` 或者 `--color=black`。

执行 `xmake echo --help` 命令再查看 help 帮助菜单里面的参数列表，就会是下面图片中的这个样子。

![5](实验18插件开发详解.assets/6d8c6ec9c7659a657cfead74e8eae031-0)

接着再来讲解上面 `on_run` 配置块中的脚本，我们通过 `option.get("color")` 和 `option.get("bright")` 判断颜色、高亮是否被设置，来决定是否开启 `cprint("${red bright}hello xmake!")` 等高亮色彩输出。

如果我们执行下面的命令，就会显示红色的字符串文本，例如。

```bash
xmake echo -c red 'hello xmake!'
```

![6](实验18插件开发详解.assets/0bbd108687386ba4b1733c0200bb22bb-0)

这里我们仅仅通过简写的方式传递了 `-c red` 参数，其实也可以写全参数名传递，效果是相同的，例如 `--color=red`。

然后我们继续传递 `-b/--bright` 参数开启高亮输出，也就是执行下面的命令，就会显示高亮后的红色字符串文本，例如。

```bash
xmake echo -b -c red 'hello xmake!'
```

或者

```bash
xmake echo --bright --color=red 'hello xmake!'
```

![7](实验18插件开发详解.assets/f2a4f43f2c38a7f972c3291bbc3b344f-0)

#### 内部任务的定义和使用

通过 `task()` 定义的任务，并不一定是插件任务，只能说我们可以使用 `task()` 来暴露子命令实现插件扩展，当然也可以不对外开放 `task()` 的任务调用，仅仅提供给 xmake.lua 内部的配置脚本使用。

只要我们定义的任务没有调用 `set_menu` 配置上对应帮助菜单和参数选项，那么这个任务就不是对外开放的，无法通过子命令的方式直接执行。

例如，我们把刚刚的插件改造成内部任务，不作为插件提供。

```lua
task("echo")
    on_run(function (content, opt)
        if opt.color or opt.bright then
            cprint("${%s %s}%s", opt.color or "", opt.bright and "bright" or "", content)
        else
            print("%s", content)
        end
    end)

target("test")
    set_kind("binary")
    add_files("src/*.cpp")
    after_build(function (target)
        import("core.base.task")
        task.run("echo", {}, "hello xmake!", {color = "red", bright = true})
    end)
```

只要我们不设置 `set_menu` 给 `task()`，那么这个任务就属于内部任务，虽然无法通过外部命令的方式调用，但是我们可以在自定义脚本中通过 `core.base.task` 模块的 `task.run` 接口直接去调用它。

```lua
task.run("echo", {}, "hello xmake!", {color = "red", bright = true})
```

其中，第一个参数是任务名称，第二个参数就是 `set_menu` 中定义的命令参数选项的实际值列表（这里没有，我们调用 `xmake echo` 等插件命令时候可以传递），第三个参数之后的值都是直接传入给 `on_run()` 中的函数参数。

因此，我们即使不走命令行参数，也可以快速的把参数传递进任务脚本中，这里的配置例子，会在编译完成后，运行 echo 任务去打印红色的字符串信息，如下图。

![8](实验18插件开发详解.assets/524d4cb06469ed622598468927c5a08f-0)



## 实验总结

在本节实验中，我们了解了 xmake 插件的基本概念，并且通过一个完整的 echo 插件例子来学习整个插件的编写过程，同时还学习了如何去编写和调用内部的任务脚本。

本实验的参考代码可以使用如下命令下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/2764/code18.zip
```

