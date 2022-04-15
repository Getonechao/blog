+++

title= "实验15扩展模块导入和使用"
description= "蓝桥网课xmake笔记"
date= 2021-07-15T10:36:57+08:00
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

# 扩展模块导入和使用

在本节实验中，我们将会学习到 xmake 中模块的基本概念和种类，并且还会学到如何在项目配置中通过 import 去导入和使用内部扩展模块，以及用户和第三方模块。

#### 知识点

- xmake 的模块类型
- import 模块导入接口的基本使用
- 内部扩展模块的导入
- 用户和第三方模块的导入



## import 模块导入接口介绍

虽然我们可以在自定义脚本中使用 xmake 提供的一些内置模块，例如上两节实验中介绍的 os 系统模块，以及 io 文件操作模块等等。

但是内置的模块毕竟有限，单纯地使用内置模块是无法满足所有用户在自定义脚本中的各种复杂的配置需求，而如果我们把更多的模块集成进 xmake 的内置模块中，这样不仅会让 xmake 非常的臃肿，而且更容易导致命名冲突等其它问题。

因此，我们可以使用 xmake 提供的 import 接口来导入扩展模块进行使用，这样既保证了 xmake 精简的同时，又能让其具备灵活强大的扩展性。

在具体介绍如何使用 import 导入模块之前，我们先简单介绍下 xmake 提供的各种模块种类。

- 内置模块：不导入就可以直接使用，不需要 import 额外导入。
- 内置的扩展模块：模块本身还是在 xmake 安装包内的，但是默认没有导入进来，只有通过 import 导入后才能使用。
- 外部的扩展模块：用户自己实现的模块、或者第三方的 lua 模块文件，通过 import 指定路径后导入进来使用。

#### 导入内置的扩展模块

xmake 本身提供了大量的内置扩展模块，虽然默认没有导入到自定义脚本中来，但是用户可以根据自己项目的实际配置需求，选择性导入一些需要的扩展模块来使用。

基本上，xmake 的扩展模块已经可以满足大部分用户 80% 的脚本配置需求，这里我们主要列举和介绍一些比较常用的模块包。

| 模块包       | 描述                         |
| ------------ | ---------------------------- |
| core.base    | xmake 的核心基础模块         |
| core.project | xmake 的核心工程配置相关模块 |
| lib.detect   | 各种检测模块                 |
| net          | 网络访问相关模块             |
| utils        | 实用工具模块                 |
| ui           | 终端字符 ui 库模块           |

其中，我们大部分的功能模块都在 `core` 这个核心模块包中，里面有各种基础库，以及对 xmake 工程结构和信息的各种访问操作接口。

而 import 的使用方式其实跟 java、kotlin 这类语言很像，传入 import 的模块名参数也是基于 `.` 来按包名层级划分的，例如 `import("core.project.config")`。

接下来，我们可以创建一个空工程来尝试如何使用 import 导入 xmake 的内部扩展模块。

```bash
cd ~/Code
xmake create import_test
```

创建完成后，进入 import_test 目录，然后编辑里面的 xmake.lua 文件，在 `on_load` 阶段通过 import 导入工程的 config 模块来获取 `xmake f/config -p xxx -m debug` 里面的编译平台、架构以及编译模式等配置信息。

```lua
target("import_test")
    set_kind("binary")
    add_files("src/*.cpp")
    on_load(function (target)
        import("core.project.config")
        print("plat: %s", config.get("plat"))
        print("arch: %s", config.get("arch"))
        print("mode: %s", config.get("mode"))
    end)
```

通过上面的配置，我们大概了解到，只要通过 import 导入了 `core.project.config` 模块后，我们就可以在导入的当前作用域内直接使用 config 模块了。

执行 `xmake` 查看能否输出打印的配置信息，如果运行正常，那么会显示下图的结果。

![1](实验15扩展模块导入和使用.assets/a1350cc10ddb32f32a986a24b628f46c-0)

下面，我们再来尝试一个例子，这回导入 `core.project.project` 模块来使用，通过这个模块，我们可以获取到所有在 xmake.lua 中定义的 target 目标列表，然后通过这些 target 对象，获取到每个目标程序名和实际输出路径。

```lua
target("test2")
    set_kind("binary")
    add_files("src/*.cpp")

target("import_test")
    set_kind("binary")
    add_files("src/*.cpp")
    after_build(function (target)
        import("core.project.project")
        for _, t in pairs(project.targets()) do
            print("%s: %s", t:name(), t:targetfile())
        end
    end)
```

为了使得遍历 target 目标的效果更加明显，我们这里在配置中额外添加了一个 test2 的目标程序，并且这回我们是在 `after_build` 阶段遍历的 targets，这是因为在 `on_load` 阶段，xmake 还没完成对所有 targets 的加载，因此是无法获取所有目标的。

执行 `xmake -r` 重新编译程序，在构建完成后的阶段遍历所有 targets 目标程序，如下图。

![2](实验15扩展模块导入和使用.assets/e5e801279f14ab5c1fb95e4c911cff53-0)

关于 xmake 的更多模块介绍和使用，可以查看 xmake 的 [扩展模块官方文档](https://xmake.io/#/zh-cn/manual/extension_modules)，也可以直接到 [xmake 扩展模块源码目录](https://github.com/xmake-io/xmake/tree/master/xmake/modules) 中去翻阅查找。

## 导入外部第三方模块

除了 xmake 自身提供的扩展模块，import 还支持导入大家自己编写的模块、以及其它的第三方 lua 模块。大家只需要将自己的 lua 模块放置在跟 import 调用脚本同级的目录下，比如 mymod.lua，那么 `import("mymod")` 就可以通过相对路径的方式直接导入进来。

我们按这个思路，先在当前项目根目录创建 mymod.lua 文件。

```bash
touch mymod.lua
```

其整个目录结构大概如下。

```bash
tree
.
├── mymod.lua
├── src
│   └── main.cpp
└── xmake.lua
```

然后执行 `vim mymod.lua` 编辑添加这个自定义模块的代码，在里面定义导出一个 `hello()` 的模块接口。

```lua
function hello(str)
    print("mymod: %s", str)
end
```

完成 mymod.lua 的模块实现后，我们就可以在 xmake.lua 中将其导入进来使用了。

```lua
target("import_test")
    set_kind("binary")
    add_files("src/*.cpp")
    after_build(function (target)
        import("mymod")
        mymod.hello("xmake")
    end)
```

通过 `import("mymod")` 导入后，就可以调用 `mymod.hello` 的模块接口了，这里我们传入一些参数进去并且在 mymod 模块里面将其打印出来。

执行 `xmake -r` 运行后的效果如下图红线部分的输出。

![3](实验15扩展模块导入和使用.assets/6ccaf307d6b7e2c793e27ef149cf0466-0)

#### import 的模块查找顺序

我们现在大概知道了，可以直接从当前路径下导入自己的模块文件，其实 import 还会从很多其它路径查找模块。因此在进一步了解模块的导入规则之前，我们先来了解下 import 接口导入模块的整个查找流程。

1. 优先从用户在 xmake.lua 中通过 `add_moduledirs` 接口指定的模块根目录下查找指定模块。
2. 从 `$XMAKE_MODULES_DIR` 环境变量指定的模块目录中查找模块。
3. 从 `~/.xmake/modules` 全局模块目录中查找模块。
4. 从 xmake 的安装目录里面的模块目录中查找模块。
5. 从当前 import 被调用的脚本文件所在目录下查找模块。

从上面的流程中可以知道，刚刚我们从当前目录下加载模块，就是上面最低的查找优先级，也就是第 5 种方式。而对于项目维护，其实我们更推荐使用第一种方式来加载自己的扩展模块。

通过第 1 种查找方式，我们使用 `add_moduledirs` 接口固定每个项目的扩展模块存储根目录，统一管理存放，也方便了用户的管理，以及跟 C/C++ 代码文件的独立存储。

为了更加深刻的了解第 1 种方式的模块导入和维护，我们继续改进下项目工程，执行下面的命令将 mymod.lua 挪一下位置，将其放置到 `xmake_modules` 模块目录下。

```bash
mkdir xmake_modules
mv mymod.lua xmake_modules
```

整个项目结构大概如下。

```bash
.
├── src
│   └── main.cpp
├── xmake.lua
└── xmake_modules
    └── mymod.lua
```

然后编辑 xmake.lua 文件，通过 `add_moduledirs` 接口，设置 `xmake_modules` 作为我们自己的扩展模块根目录。

```lua
add_moduledirs("xmake_modules")

target("import_test")
    set_kind("binary")
    add_files("src/*.cpp")
    after_build(function (target)
        import("mymod")
        mymod.hello("xmake")
    end)
```

除了新加入的 `add_moduledirs("xmake_modules")` 指定模块目录，其它配置完全跟之前保持一致，虽然现在 mymod.lua 已经不在当前 xmake.lua 文件同级目录下了，但是我们还是可以通过 `import("mymod")` 直接找到并导入进来。这是因为 xmake 现在会自动先从指定的 xmake_modules 目录下查找是否存在 mymod.lua 模块文件，如果有就会加载进来。

执行 `xmake -r` 命令，可以看到模块的执行效果也跟之前完全一样，也正常输出了对应的信息。

![3](实验15扩展模块导入和使用.assets/6ccaf307d6b7e2c793e27ef149cf0466-0)

#### 在独立模块文件中导入

我们不仅可以在 `after_build` 等自定义脚本中进行模块导入，还可以在任意的模块脚本文件中使用 import 来继续导入其它模块。还是拿之前的 mymod.lua 模块文件为例，我们继续修改完善它。

```lua
import("lib.detect.find_tool")

function hello(str)
    print("mymod: %s", str)
    print(find_tool("gcc"))
end
```

由于是在独立的脚本文件中，因此 import 导入调用没必要放置在 function 内部，仅需要放置到文件最开头即可。这样，当前文件内所有的函数实现都可以访问到被导入的模块。

这里，我们导入了 `lib.detect.find_tool` 模块，用来探测当前环境下的 gcc 命令是否存在，以及对应的路径信息。

由于实验环境已经安装了 gcc，因此如果我们继续执行 `xmake -r` 命令，应该会看到下图的输出信息。

![4](实验15扩展模块导入和使用.assets/576b117270381a4c7fcff49b55ac07b9-0)

#### 导入作为函数接口使用

除了可以调用导入模块的接口之外，其实还可以将被导入的模块作为函数来调用，例如 `mymod("xx")` 相当于通过 import 导入了一个接口函数。

这通常也是非常有用的，不过为了支持这种方式调用，我们需要修改 mymod.lua 文件，在里面提供一个 `main()` 的入口函数才行，import 接口会判断加载的模块是否存在 main 入口，如果存在就可以通过这种方式调用我们导入的模块。

修改 mymod.lua 中的代码，加上 main 入口。

```lua
import("lib.detect.find_tool")

function _hello(str)
    print("mymod: %s", str)
    print(find_tool("gcc"))
end

function main(str)
    _hello(str)
end
```

这里，我们还把 `hello()` 重命名成了 `_hello()`，这是可选的操作，只不过带下划线前缀后，`_hello` 就会被当做私有接口，不参与导出，也就是我们无法再直接调用 `mymod._hello()` 了。

接下来，我们再来稍微改动下 xmake.lua，改成函数式调用。

```lua
add_moduledirs("xmake_modules")

target("import_test")
    set_kind("binary")
    add_files("src/*.cpp")
    after_build(function (target)
        import("mymod")
        mymod("xmake") -- 此处变成直接函数调用
    end)
```

可以看到上面的配置中，我们不再是调用 `mymod.hello()` 了，而是改成了 `mymod()` 作为函数接口来直接调用，其它地方没有任何变化。

执行 `xmake -r` 查看结果，如果运行正常，应该跟之前一样的输出结果。

![4](实验15扩展模块导入和使用.assets/576b117270381a4c7fcff49b55ac07b9-0)

#### 导入模块包和别名

之前我们都是写全了完整模块路径来导入指定模块，其实每个 `.` 之前的名称就是模块所属的父目录，每一级对应一层子目录。比如：`core.project.config` 其实就是对应的 `moduledir/core/project/config.lua`。

而每级的子目录在 import 中称之为包名，里面可以包含很多的模块，并且我们可以通过 import 导入整个包，而不是单个模块，例如修改 mymod.lua。

```lua
import("lib.detect", {alias = "lib_detect"})

function _hello(str)
    print("mymod: %s", str)
    print(lib_detect.find_tool("gcc"))
end

function main(str)
    _hello(str)
end
```

这里，我们仅仅通过 import 导入了整个 `lib.detect` 包，然后通过扩展参数 `{alias = "lib_detect"}` 对导入的包取了一个别名 `lib_detect`，这个参数是可选的，仅仅是为了避免作用域内与变量重名，导致冲突。

然后，我们就可以通过 `lib_detect.find_tool` 方式，将这个包作为一个模块，调用里面的 `find_tool.lua` 模块接口。当然也可以只导入 `lib` 包，然后调用 `lib.detect.find_tool()`。

最后执行 `xmake -r`，输出结果应该跟之前完全一样。



## 实验总结



在本节实验中，我们主要了解了 xmake 里面模块的概念，并学习了如何去使用 import 接口导入 xmake 提供的各种内置扩展模块，以及如何去导入用户自己的扩展模块。

本实验的参考代码可以使用如下命令下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/2764/code15.zip
```