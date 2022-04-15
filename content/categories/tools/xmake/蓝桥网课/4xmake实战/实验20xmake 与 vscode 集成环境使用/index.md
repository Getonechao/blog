+++

title= "实验20xmake-vscode 插件介绍"
description= "蓝桥网课xmake笔记"
date= 2021-07-15T10:37:15+08:00
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



在本节实验中，我们将学习如何在 vscode 中使用 xmake 编译开发 C/C++ 程序，如何进行断点调试以及 xmake-vscode 插件的一些基本操作和使用。

#### 知识点

- xmake-vscode 插件基本使用
- 使用插件进行断点调试
- 命令面板的使用
- 插件的参数配置和修改





## xmake-vscode 插件介绍



我们之前的所有实验，都是使用 xmake 的命令行程序在终端下操作完成的，这对于一些初学者来说还是有不少门槛的，并且操作起来也不能够像其它 IDE 等带有可视化界面的开发环境那样顺手，尤其是代码的编辑、编译和断点调试都需要不停的切换各种终端、编辑器环境才能完成。

为了方便我们的日常开发，xmake 官方提供了可以快速无缝集成到 Visual Studio Code 编辑器的 [xmake-vscode 插件](https://github.com/xmake-io/xmake-vscode)，使用这个插件我们可以通过 vscode 编辑器环境来一站式进行 C/C++ 程序开发，内置 xmake 编译、断点调试、编译错误分析定位、编译配置的快速切换等各种实用功能。

<img src="实验20xmake 与 vscode 集成环境使用.assets/03b71834190574523cd4d56449e00f56-0"/>





而 Visual Studio Code 编辑器是微软推出的一款轻量级跨平台的编辑器，具有非常好的跨平台性、可扩展性，我们在实验环境的桌面上，就能找到带有 Visual Studio Code 字样的图标，双击运行就可以打开它。

#### 安装 xmake-vscode 插件

首先在环境中安装 xmake，执行如下命令：

```bash
bash <(curl -fsSL https://xmake.io/shget.text) v2.3.7
source ~/.xmake/profile
# 检查版本，验证安装成功
xmake --version
```

在使用 xmake-vscode 插件之前，我们需要安装它，安装过程很简单，只需要切换到 vscode 的插件扩展 tab 页，然后输入 xmake 搜索相关插件。

如果看到下图所示内容，说明我们已经成功找到了 xmake-vscode 插件，然后点击旁边的 `Install` 字样按钮就可以完成安装了。

<img src="实验20xmake 与 vscode 集成环境使用.assets/bf47bedefafe0e442b734033dabf50e3-0"/>

#### 创建和打开 C/C++ 工程

插件安装完成后，我们可以先来创建一个空的 C 工程，之前介绍过可以使用 `xmake create` 命令来创建工程。但是这里我们打算直接使用 xmake-vscode 插件提供的工程创建功能来完成 C/C++ 工程的创建。

在创建之前，需要先准备一个空目录用来存放工程文件，例如将工程放置在 `~/Code/vscode_test` 目录下，如果还不存在此目录，可以先手动创建下。

然后，我们直接在 vscode 里面打开这个空目录，只需要通过点击下图红框位置的按钮。

<img src="实验20xmake 与 vscode 集成环境使用.assets/c8dd3c0af495231179389909b7e39763-0"/>

点击后，我们进入刚刚创建的 vscode_test 目录，选中并打开它。

<img src="./实验20xmake 与 vscode 集成环境使用.assets/2c34503389fad3fff3370e3a37516af5-0"/>

打开目录后，我们就可以开始创建工程文件了，继续进入菜单的 view 子菜单打开 vscode 里面的命令面板，如下图。

<img src="./实验20xmake 与 vscode 集成环境使用.assets/c8ccab2904e2e8d7a47af375d1fbedb4-0"/>

命令面板打开后，我们输入 xmake 字符串，就会看到一系列跟 xmake 插件相关的命令，然后从其中找到带 `Create Project` 字样的命令，如图。

<img src="./实验20xmake 与 vscode 集成环境使用.assets/ba0d31515a3807ec00519a976939978d-0"/>

这个时候，由于当前还没有 xmake.lua 文件，会在底下弹出一个提示框，继续点击里面的蓝色按钮。

<img src="./实验20xmake 与 vscode 集成环境使用.assets/db4ccace608177319f5fd46bb27aa979-0"/>

接着就会弹出编程语言的选择列表，这里选择第二项，也就是 C++ 语言。

<img src="./实验20xmake 与 vscode 集成环境使用.assets/40adb147a8dd183a2306a27b045c76b2-0"/>

选择完语言后，还会提示选择工程类型，这里选择 console 终端程序类型。

<img src="./实验20xmake 与 vscode 集成环境使用.assets/fd6f5ab2aa96f8288315db7c20671373-0"/>

<img src="./实验20xmake 与 vscode 集成环境使用.assets/88b727ce22ab5e1618747f227b5a05a1-0"/>

选择完成后，整个项目就创建好了，下图左边就是新创建的工程文件，根目录下有 xmake.lua，而图最底下的工具栏就是 xmake 插件的操作面板了，我们的大部分操作都可以通过这个面板快速完成，包括：编译、运行、调试以及配置切换等等。

#### 工具栏面板介绍

创建完成工程后，我们会看到 vscode 底部工具链出现了一排跟 xmake 相关的操作面板，xmake 的大部分操作都可以通过这个面板上的工具按钮来快速完成，如下图。

<img src="./实验20xmake 与 vscode 集成环境使用.assets/2118a2b7d665f65241d0dfddaf9cbbe8-0"/>

另外，不仅仅是创建工程，如果安装 xmake-vscode 插件后，使用 vscode 打开一个带有 xmake.lua 文件的 C/C++ 工程根目录，那么 vscode 底部的 xmake 工具栏面板也会被自动激活。

通过上图，我们大概能知道每个按钮的具体功能，具体更进一步的使用方式，我们会接下来会挨个讲解。

#### 编译 C/C++ 程序

刚刚我们通过 vscode 创建的一个 C++ 项目工程，其内部就是执行了 xmake 的 `xmake create vscode_test` 命令来完成的，整个工程文件结构如下。

```bash
.
├── src
│   └── main.cpp
└── xmake.lua
```

在编译之前，先调整下生成的 xmake.lua 文件，修改成如下配置。

```lua
add_rules("mode.debug", "mode.release")

target("test")
    set_kind("binary")
    add_files("src/*.cpp")
```

其实，也就是将程序目标名改成 test，然后点击 vscode 底下的 xmake 工具栏里面的 Build 按钮，来编译我们新创建的 C++ 工程，编译效果如下图。

<img src="./实验20xmake 与 vscode 集成环境使用.assets/b49ecdc6a7f33dede8479f5a1820d0e0-0"/>

#### 运行程序

编译完成后就可以尝试运行程序了，还是使用底下的工具栏按钮，点击运行图标，也就是下图红框中的按钮。

<img src="./实验20xmake 与 vscode 集成环境使用.assets/7bf746bfb5f96b29a2bd803e4763be02-0"/>

如果运行成功，就会看到实际的运行输出信息：`Hello world!`。

#### 调试程序

接下来，我们重点讲解如何通过 vscode 配合 xmake-vscode 插件来实现断点调试 C/C++ 程序。

不过在调试前还需要做一些准备工作，由于默认 xmake 采用 release 模式编译的目标程序，是不带调试符号信息的，因此我们需要先将编译模式切换到 debug 调式编译模式去重新编译它，使其带上调试符号信息。

具体如何切换到 debug 模式，可以参考下图的操作，点击底下 `release` 文本所在的按钮，然后在上面列出的列表中，选择 debug 项即可。

<img src="./实验20xmake 与 vscode 集成环境使用.assets/c90d83e787e8e81e175e4b25855827b1-0"/>

完成切换后可以看到底下的 release 按钮已经变成了 debug 字样，然后点击 build 按钮，就可以重新编译带有调试符号的目标程序了。

<img src="./实验20xmake 与 vscode 集成环境使用.assets/b613c71e1eb30f1b8743db09376ab4b1-0"/>

调试版本程序编译完成后，还需要额外做一件事，那就是安装 vscode 的 C/C++ 插件，因为 xmake-vscode 插件的调试功能是基于这个插件的，我们仅仅只需要首次使用时安装它即可。

重新切到插件市场页面，搜索 C/C++，显示出来的第一项就是，点击 Install 安装即可，如下图。

<img src="./实验20xmake 与 vscode 集成环境使用.assets/9e24b7095f8ec69a722b55b4d827d945-0"/>

安装好 C/C++ 插件，就可以开始调试操作了。首先点击 main.cpp 打开源文件，然后在需要下断点的代码行左侧位置点击下断点，如果出现小红点，说明已经成功下好了调试断点，如下图。

<img src="./实验20xmake 与 vscode 集成环境使用.assets/bfea06b01c9fba21892d946a4863ff42-0"/>

断点下好后，就可以点击底下的调试按钮，开启断点调试了，如果一切顺利，程序运行起来后就会命中刚刚设置的断点，如下图。

```bash
# 需要先在环境中安装 gdb
sudo apt install -y gdb
```

<img src="./实验20xmake 与 vscode 集成环境使用.assets/38b60efa035573b205d0c94362b14b29-0"/>





## 目标程序切换

接下来再来详细讲解编译，之前我们已经使用过 build 按钮来编译工程，但如果一个项目中存在多个目标程序又不想全部编译，我们就需要指定编译哪个目标程序。

在命令行中，我们可以通过 `xmake build test` 命令来显示的指定编译哪个目标，而在 vscode 中，xmake 插件也提供了很方便的目标切换操作，来快速切换编译。

首先修改 xmake.lua 文件，新增一个名为 test2 的目标程序，用于之后的目标切换测试，例如。

```lua
add_rules("mode.debug", "mode.release")

target("test")
    set_kind("binary")
    add_files("src/*.cpp")

target("test2")
    set_kind("binary")
    add_files("src/*.cpp")
```

然后点击底下 default 字样的按钮，之后会在顶部显示整个项目所有目标程序名的列表，我们点击其中的 test2 目标程序，就可以完成切换。

如果切换成功，底下的 default 文本就会变成 test2，由于默认编译 xmake 会自动编译所有目标程序，所以最初显示的是 default 文本。
<img src="./实验20xmake 与 vscode 集成环境使用.assets/be42d714a0eae7edb2970463699f47ef-0"/>

这个时候，我们再执行编译，就能看到实际仅仅只编译了我们指定的 test2 目标程序。
<img src="./实验20xmake 与 vscode 集成环境使用.assets/30afcbd89ad353846eb7e8f325ca9be1-0"/>



#### 编译错误信息

如果我们的工程代码没写对导致编译出错，可以直接从编译输出中看到错误信息，例如下图。
<img src="./实验20xmake 与 vscode 集成环境使用.assets/6e0996f5e932b770037ef54e6b9f0de3-0"/>



而 xmake-vscode 插件还会自动解析编译错误输出信息，分类每个编译错误，并可通过双击指定错误，跳转定位到指定的错误代码位置，也就是下图所示位置。
<img src="实验20xmake 与 vscode 集成环境使用.assets/f620adcfd5c25239ad8deb81c000c0a4-0"/>



#### 查看编译详细信息

在命令行中，我们可以通过 `xmake -v` 来查看编译过程中的详细命令参数信息，而在 vscode 中同样可以通过配置开启详细输出。

首先打开菜单，点击 `File` -> `Preferences` -> `Settings` 子菜单，如下图。
<img src="./实验20xmake 与 vscode 集成环境使用.assets/dc05caf79d6f9f713e5e1bc2c483cad2-0"/>



然后在打开的 Setting 配置页面，输入 xmake 找到所有跟 xmake 插件相关的配置项，其中有一项是 BuildLevel，它就是用于设置编译过程中的输出信息级别。

默认是 warnings 级别，仅仅输出编译警告信息以及正常信息，我们把它改为 verbose 级别，就可以输出完整的编译命令行参数了。
<img src="./实验20xmake 与 vscode 集成环境使用.assets/e29080d4ec31c9cc3eb4e07583664ede-0"/>

至于 debug 级别对应的就是 `xmake -vD` 的诊断信息，还会进一步打印出错的栈信息。

我们这里将配置切换成 verbose 级别后，再重新构建下程序，看看实际的输出是怎样的。不过由于底部只提供了 build 按钮，没有 rebuild 按钮，为了执行重新编译，我们需要从菜单里面的命令面板中，找到 xmake 的 Rebuild 命令，点击执行才行。

从下图的红色箭头位置找到对应的命令面板。
<img src="./实验20xmake 与 vscode 集成环境使用.assets/8cb30e5c61ba43a149e549654aa54fa8-0"/>

然后再打开的命令面板中输入 xmake 过滤出所有跟 xmake 相关的命令列表，找到 Rebuild 命令后点击编译即可，如下图。
<img src="./实验20xmake 与 vscode 集成环境使用.assets/5b3b425dc410962e44e3974096ef1267-0"/>

开启 verbose 级别后，我们就能看到编译输出中完整的命令参数了，如下图。
<img src="./实验20xmake 与 vscode 集成环境使用.assets/c142df9ee8281f93f32f34067eda9595-0"/>

#### xmake.lua 编辑和自动补全

xmake-vscode 插件还内置了对 xmake.lua 文件编写时的自动提示和补全支持，我们只需要输入 `add_`、`set_` 等字样的文本，就会自动列举出与其相关的所有 api 供我们使用，来方便快速配置 xmake.lua。

我们可以通过编辑 xmake.lua 文件对里面 test2 目标程序添加一个 `add_defines("TEST2")` 来体验下自动补全功能。

最终的完整配置内容如下。

```lua
add_rules("mode.debug", "mode.release")

target("test")
    set_kind("binary")
    add_files("src/*.cpp")

target("test2")
    set_kind("binary")
    add_files("src/*.cpp")
    add_defines("TEST2")
```

而我们输入时的补全特性可以通过下面的图片体会到。
<img src="./实验20xmake 与 vscode 集成环境使用.assets/dd007db9f4534867827d78a57f13e9e1-0"/>

完成配置后，我们再来点击底下的 Build 按钮执行编译，看下详细命令输出，应该能够正常看到新加上的 `-DTEST2` 宏定义了。
<img src="./实验20xmake 与 vscode 集成环境使用.assets/868c1abb025eb7844f99ff304f83d7fc-0"/>



## 实验总结



在本节实验中，我们学习了如何在 vscode 中使用 xmake-vscode 插件来编译开发 C/C++ 程序，以及如何进行断点调试，并且学习了 xmake 工具栏和命令面板的使用以及如何进行参数配置。

本实验的参考代码可以使用如下命令下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/2764/code20.zip
```