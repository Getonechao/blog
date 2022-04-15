+++

title= "实验9C/C++ 子工程和目录结构"
description= "蓝桥网课xmake笔记"
date= 2021-07-15T10:36:45+08:00
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

# C/C++ 子工程和目录结构

在本节实验中，我们会学到如何使用 xmake 去实现对更加复杂的大型 C/C++ 项目的构建维护，以及如何使用 includes 接口，并且还能了解和掌握子工程之间的配置继承关系。

#### 知识点

- 如何进行大型 C/C++ 工程项目构建维护
- xmake 的 includes 接口使用
- 子工程配置之间的继承和依赖关系



## 使用 includes 引入目录和文件

在之前的实验中，我们的工程项目比较简单，通常仅仅只需要在项目根目录下放置一个 xmake.lua 配置文件，就可以控制整个项目的编译。

但是对于一些大型项目，其源码的组织结构相对比较复杂，层级也比较深，需要参与编译生成的 target 目标也可能有十几甚至上百个，这个时候如果还是都在单一的根 xmake.lua 文件中维护，就有点吃不消了，会显得非常的臃肿并且不可维护。

这个时候，我们需要通过在每个子工程模块里面，单独创建 xmake.lua 来维护它们，然后使用 xmake 提供的 includes 接口，将它们按层级关系包含进来，最终变成一个树状结构。

在讲解如何使用前，我们先做一些准备工作，执行如下命令创建一个稍微复杂点的项目结构。

```bash
cd Code
xmake create includes_test
cd includes_test
mkdir tests
touch src/xmake.lua
cp src/* tests
```

执行完成后，执行 `tree .` 命令查看当前的项目结构。

```bash
tree .
.
├── src
│   ├── main.cpp
│   └── xmake.lua
├── tests
│   ├── main.cpp
│   └── xmake.lua
└── xmake.lua
```

通过上面的项目接口，我们大概可以看出来，这里面有两个模块，一个是 src 目录下的主程序，一个是 tests 目录下的测试程序，它们分别有自己的 xmake.lua 来维护构建，然后通过项目根目录的 xmake.lua 将这两个模块关联起来，构成一个项目树，大概的结构如下图。

![1](实验9C_C++子工程和目录结构.assets/ca435d943a281fae38078d6065bc2dd6-0)

参照上图的结构，我们开始编写对应的 xmake.lua 配置内容，首先是根目录下的 xmake.lua。

```lua
includes("src", "tests")
```

现在的根配置就非常精简了，就一行 includes 去引入 src 和 tests 目录下的子 xmake.lua 配置，之前的 `target()` 的定义也没有了，因为我们已经挪到了子目录 xmake.lua 中去分别定义它们。

接下来我们再来编辑下 `src/xmake.lua` 里面的配置内容，定义主程序 target 目标。

```lua
target("hello")
    set_kind("binary")
    add_files("*.cpp")
```

然后编辑 `tests/xmake.lua` 定义测试程序 target 目标。

```lua
target("test")
    set_kind("binary")
    add_files("*.cpp")
```

到这里为止，我们已经基本完成了一个带有两个子工程的 C++ 项目，相比之前的单级 xmake.lua 维护，现在的结构更加容易扩展，通过子工程拆分后，每个子 xmake.lua 维护会更加的清晰和精简。不同子工程的 target 目标之间也可以独立维护，互不干扰了。

最后执行 `xmake` 编译这个工程，看看能否正常编译引入的 hello 和 test 这两个子目标程序，如果一切顺利，会看到下图的输出结果。

![2](实验9C_C++子工程和目录结构.assets/5617560a92e8846c19f3b2f308288ba1-0)

通过 includes 关联子工程的好处就是，我们不需要再手动进入每个子工程目录下执行 xmake 编译，所有的 xmake 命令操作还是可以在项目根目录下统一完成，因为通过 includes 引入后，整个项目不管结构多复杂，都是一个整体，因此即使用户进入子目录执行 xmake 命令，实际的操作对象还是针对当前的整个项目而言的。





## 模式匹配实现批量 includes

默认情况下通过 `includes("tests")` 引用子目录，就可以直接导入子目录下 xmake.lua 配置，如果要导入其它文件名的配置文件，可以指定文件全名到 includes，例如 `includes("tests/xmake.lua")`。

基于此，再加上 includes 的模式匹配支持，我们就可以实现批量导入当前工程的所有子 xmake.lua，而不再需要挨个进入子目录手动去配置引入。

修改项目的根 xmake.lua 的内容改为如下配置。

```lua
includes("*/xmake.lua")
```

通过 `*/xmake.lua` 模式匹配，xmake 会自动遍历当前所有一级子目录下的 xmake.lua 文件，将其引入。整个模式匹配的用法跟 `add_files` 是完全一致的，因此我们也可以使用 `**/xmake.lua` 来实现递归模式匹配，具体使用哪个，就看大家自己的实际项目需求了。

改完后，我们重新执行 `xmake -r`，如果一切顺利，那么编译效果应该跟之前完全相同。



## 根 xmake.lua 配置

现在，我们整体的项目结构已经通过 includes 方式将所有的子工程全部关联到了一起，使得每个 target 都被放置在单独的子工程目录中去维护。

但是，如果我们有一些通用的配置，需要得这个项目的所有子 target 目标都生效，那么，我们可以直接在根 xmake.lua 的根域去配置它们，子目录下的 target 同样可以被继承到。

例如，我们继续完善项目根目录下的 xmake.lua 文件。

```lua
set_project("hello")
set_version("1.0.1")

add_rules("mode.release", "mode.debug")

set_warnings("all", "error")
set_languages("c++11")
add_defines("ROOT")

includes("*/xmake.lua")
```

我们在上述的根配置中，加了很多的常用配置，比如添加了 debug、release 的编译规则，设置工程名和版本（可选的），设置 C++ 语言标准到 c++11，设置编译警告为严格检测和报错，另外添加了一个额外的宏定义开关：`-DROOT`。

这些根配置会对 includes 里面的所有子 target 目标都生效，属于全局项目配置。这也是我们推荐的做法，就是在根 xmake.lua 中仅仅配置一些对所有 target 都通用的设置，而 target 相关的特定配置就单独放到子工程 xmake.lua 去配置它们。

执行 `xmake -v` 查看当前的编译输出效果，如下图。

![3](实验9C_C++子工程和目录结构.assets/80f3b4d2a7fbda9e2772bc124490dd96-0)

这回，我们看到 src 和 tests 目录下的两个目标程序的 C++ 源文件在编译的时候，都被加上了 `-DROOT -std=c++` 和 `-Wall -Werror` 等编译选项，说明根目录下的配置对所有子 target 目标程序都生效了。

## 子 xmake.lua 配置

全局根配置完成了，接下来需要开始对每个子 target 进行一些特定的配置，这些配置可以分别放置在 target 所在的子 xmake.lua 文件中。

我们在每个子工程目录中单独配置的 xmake.lua，里面的所有配置不会干扰父 xmake.lua，只对它下面的更细粒度的子工程生效，就这样一层层按 tree 状生效下去。

整个继承结构可以参考下图所示。

![4](实验9C_C++子工程和目录结构.assets/0371ee1ae8a578b86f608a0591bb9a43-0)

为了完成上图的子工程结构，能够更加直观的感受如何去配置子工程，我们需要再对之前的配置文件做一些调整，在 src 目录下额外新增两个子工程：foo 和 bar 目标程序。

为此先执行如下命令，创建这两个子工程。

```bash
cd ~/Code/includes_test
mkdir src/foo src/bar
cp src/main.cpp src/foo
cp src/main.cpp src/bar
rm src/main.cpp
touch src/foo/xmake.lua
touch src/bar/xmake.lua
```

创建完成后的目录结构如下所示，也就是上面图片中所展示的结构。

```bash
.
├── src
│   ├── bar
│   │   ├── main.cpp
│   │   └── xmake.lua
│   ├── foo
│   │   ├── main.cpp
│   │   └── xmake.lua
│   └── xmake.lua
├── tests
│   ├── main.cpp
│   └── xmake.lua
└── xmake.lua
```

然后我们编辑 `src/xmake.lua` 文件，删除之前的配置内容，并在文件中新配置一个全局的宏定义 `-DHELLO`，然后继续通过 `includes("*/xmake.lua")` 引用下级子配置文件，例如。

```lua
add_defines("HELLO")
includes("*/xmake.lua")
```

由于根配置没有通过递归匹配引入所有 xmake.lua，所以我们在这里需要通过 includes 再引入下级目录，当然如果根配置中是通过 `**/xmake.lua` 递归引入了所有配置文件，那么这里就可以省略了。

接下来，我们编辑 `src/foo/xmake.lua` 文件，改成如下配置。

```lua
add_defines("FOO")
target("foo")
    set_kind("binary")
    add_files("*.cpp")
```

然后编辑 `src/bar/xmake.lua` 文件，改成如下配置。

```lua
add_defines("BAR")
target("bar")
    set_kind("binary")
    add_files("*.cpp")
```

最后的这两个子工程配置文件，我们实际定义了两个可执行目标程序：foo 和 bar，并且在它们的子全局作用域分别加了 `-DFOO` 和 `-DBAR` 宏定义。

需要注意的是，虽然这两个宏定义是在 xmake.lua 的全局根作用域配置，但是仅仅只对当前 xmake.lua 以及它们的子 xmake.lua 生效，对于父 xmake.lua 中的其它 target 和其它同级 xmake.lua 里面的 target 都是没有影响的。

简单的说，整个配置作用优先按 xmake.lua 的引用路径结构 tree 状层次继承配置，然后每个 xmake.lua 里面所有 target 继承当前配置文件的根设置。

对了，还有 `tests/xmake.lua` 我们也需要修改一下，加上 `-DTEST` 的宏定义配置。

```lua
add_defines("TEST")
target("test")
    set_kind("binary")
    add_files("*.cpp")
```

整个配置继承效果，我们可以重新看下之前的图。

![5](实验9C_C++子工程和目录结构.assets/90300f801111f390a97b279abe82afb4-0)

执行 `xmake -rv` 来验证刚刚的配置结果。

![6](实验9C_C++子工程和目录结构.assets/81086db108775488d9b3f47383dc7f4a-0)

从上图的实际编译输出看到，foo 程序继承了根文件配置的 `-DROOT`，父文件配置中的 `-DHELLO`，以及自身配置文件中的全局设置 `-DFOO`，因此最终结果是 `-DROOT -DHELLO -DFOO`，而 bar 也就是 `-DROOT -DHELLO -DBAR`，另外 test 目标跟它们完全不在一个继承路径上，因此互不干扰，只有 `-DROOT -DTEST`。

然后，我们再执行 `xmake run` 命令看看运行效果。

![7](实验9C_C++子工程和目录结构.assets/5ef851f3fdabfd3e5e44268f1d65577b-0)

默认情况下，xmake 会运行当前项目里面定义的所有可执行目标程序。



## 跨 xmake.lua 间目标依赖

只要是通过 includes 引用的所有 xmake.lua，其中的 target 定义即使在不同的子 xmake.lua 中，xmake 也会将它们全部加载进来，因此这些 target 之间的依赖关系还是可以正常配置，哪怕是跨 xmake.lua 也不会有任何影响。

比如，我们让 test 程序去依赖另外两个子程序 foo 和 bar，那么我们可以使用之前的依赖配置方式去配置关联依赖。

只需要修改 `tests/xmake.lua` 里面的配置内容，添加上这两个依赖目标即可。

```lua
add_defines("TEST")
target("test")
    set_kind("binary")
    add_files("*.cpp")
    add_deps("foo", "bar")
```

然后执行 `xmake -r` 去重新编译，如果依赖已经生效，那么在链接 test 程序之前，foo 和 bar 会优先编译和完成链接，如下图。

![8](实验9C_C++子工程和目录结构.assets/242d96c9b1721a5f2b937ceeaf5cf7bd-0)

## 实验总结



在本节实验中，我们学习了如何使用 includes 接口去组织编译带有多级子工程的 C/C++ 项目工程，并学习了 includes 的基本用法，也了解 xmake 在多级 xmake.lua 中的配置继承关系。

本实验的参考代码可以使用如下命令下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/2764/code9.zip
```