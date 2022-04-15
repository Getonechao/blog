+++

title= "实验21使用 xmake 开发 Qt 程序"
description= "蓝桥网课xmake笔记"
date= 2021-07-15T10:37:20+08:00
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

#　使用 xmake 开发 Qt 程序

在本节实验中，我们将会学习到如何使用 vscode 结合 xmake-vscode 插件去编译开发 Qt 应用程序，并且也会学习到一些 Qt 相关的基础概念，以及 Qt Widgets 和 Quick App 两种 Qt 框架的应用程序开发。

#### 知识点

- Qt 的基本概念
- Qt Widgets 和 Quick App 的基本概念
- 使用 vscode 开发 Qt 应用程序



##　Qt 介绍

Qt 是一个跨平台的 C++ 应用程序开发框架。它提供给开发者建立图形用户界面所需的功能，广泛用于开发 GUI 程序，也可用于开发非 GUI 程序。使用 Qt 开发的软件，相同的代码可以在任何支持的平台上编译与运行，而不需要修改源代码。会自动依平台的不同，表现平台特有的图形界面风格。

并且 Qt 官方提供了 QtCreator 来创建和编译 Qt 程序，通过 `.pro` 工程描述文件来维护项目，实际编译时，QtCreator 内部会去调用 qmake 或者是 cmake 来进行工程的构建。

不过 xmake 也完全支持 Qt 各种工程项目的构建编译，结合 xmake-vscode 插件可以让开发者脱离 QtCreator 使用 vscode 来编译开发 Qt 程序，另外 xmake 也提供了其它各种编辑器的插件，例如 IntelliJ IDEA、Visual Studio、Sublime Text 甚至 VIM 等等，使大家可以使用自己最习惯的编辑器来开发 Qt 程序。

#### 安装 Qt 开发库

首先在环境中安装 xmake，执行如下命令：

```bash
bash <(curl -fsSL https://xmake.io/shget.text) v2.3.7
source ~/.xmake/profile
# 检查版本，验证安装成功
xmake --version
```

在进一步讲解如何使用 xmake 去编译开发 Qt 程序之前，我们首先来安装 Qt 开发过程中需要的一些基础环境和依赖库。打开 Xfce 终端，输入如下命令就可以完成安装。

```bash
sudo apt update
sudo apt install -y qtcreator qt-default qtdeclarative5-private-dev
```

#### Qt WidgetApp 程序

Qt SDK 环境安装完成后，我们就可以通过 xmake 的命令行程序创建一个基于 Qt 的 Widget App 程序空工程，执行命令如下。

```bash
cd ~/Code
xmake create -t qt.widgetapp widgetapp
```

如果创建成功，就会看到下图的输出，里面除了 xmake.lua 文件，还生成了一些基本的 Qt 代码文件，`*.ui` 是用户描述 ui 界面，这也是 Qt Widget 应用程序所必备的，而 mainwindow.cpp 文件用来定义主窗口类。

![1](实验21使用xmake开发Qt程序.assets/678bbdd3a05e96bdc7396aa811d685f1-0)

接着，我们来看下 xmake.lua 里面的配置内容，通过这个配置才能够正常使用 xmake 完成 Qt 项目的编译。

```lua
add_rules("mode.debug", "mode.release")

target("widgetapp")
    add_rules("qt.widgetapp")
    add_headerfiles("src/*.h")
    add_files("src/*.cpp")
    add_files("src/mainwindow.ui")
    -- add files with Q_OBJECT meta (only for qt.moc)
    add_files("src/mainwindow.h")
```

从上面的配置中可以看到，除了一些基本的 C/C++ 配置以外，还有一些特殊的配置。比如我们将 `mainwindow.ui` 和 `mainwindows.h` 也添加到了 `add_files` 参与编译，这两个类型的文件，原生 C/C++ 程序是不需要的，但是由于我们通过 `add_rules("qt.widgetapp")` 针对当前目标生效了 `qt.widgetapp` 规则。

这个时候，`add_files` 就能自动处理 `*.ui` 文件的编译，以及对 `mainwindows.h` 文件的编译。

注：只有代码内包含 `Q_OBJECT` 宏的头文件才需要添加到 `add_files`，因为带有 `Q_OBJECT` 的头文件里面定义了 QObject 的一些 meta 信息，可以支持 Qt 的信号槽机制，并且通过这些头文件，QtCreator 和 xmake 会自动生成对应的 `moc_mainwindow.cpp` 文件参与编译，并不是单纯的头文件。

另外，xmake 会将这两个类型的文件优先在最开头参与编译，因为它们还会自动生成一些代码文件给其它源码引用，执行 `xmake` 编译这个 Qt 工程，编译通过后会显示下图信息。

![2](实验21使用xmake开发Qt程序.assets/028da3cd499bbbf7a68aa10fcfe714c7-0)

然后执行 `xmake run` 命令运行编译生成的目标程序，如下图所示，一个可视化的空白窗口被显示出来，说明我们第一个 Qt Gui 程序已经构建运行完成。

![3](实验21使用xmake开发Qt程序.assets/92cf3d07067dc0b4d0a2e6bb974ee47b-0)

我们再来执行下面的命令，开启详细输出编译，看下 xmake 实际传了哪些编译选项、链接选项给 gcc。

```bash
xmake -rv
```

从下图的红线部分可以看出，针对 Qt Widget 的应用程序，xmake 会自动将 `QtGui`、`QtWidget`、`QtCore` 等基础库传入进去参与链接，其中 `QtCore` 是所有 Qt 程序必须要用到的，内置一些常用基础类库，而 `QtWidget` 和 `QtGui` 在写一些可视化 ui 程序的时候都是需要的。

![4](实验21使用xmake开发Qt程序.assets/a0aa45014cfb48f130392534f959c59a-0)

#### 在 vscode 中开发 Qt 程序

刚刚我们是在终端下通过 xmake 的命令行程序来编译运行的 Qt 工程，既然上节实验已经学习到了使用 vscode 和 xmake-vscode 插件在 vscode 的编辑器环境来开发调试 C/C++ 程序，那么同样可以用来编译运行 Qt 程序。

我们首先打开 vscode 编辑器，然后继续打开刚刚创建的 widgetapp 工程根目录 `/home/shiyanlou/Code/widgetapp`。

在使用 xmake-vscode 插件之前，我们需要安装它，安装过程很简单，只需要切换到 vscode 的插件扩展 tab 页，然后输入 xmake 搜索相关插件。

如果看到下图所示内容，说明我们已经成功找到了 xmake-vscode 插件，然后点击旁边的 `Install` 字样按钮就可以完成安装了。

![x](实验21使用xmake开发Qt程序.assets/bf47bedefafe0e442b734033dabf50e3-0)

安装完成后这个插件检测到当前打开的目录里面存在 xmake.lua 文件，就会自动激活 xmake 的插件程序，在底下显示一排对应的插件操作按钮。

点击 Build 按钮完成编译，然后点击三角形箭头的按钮运行我们编译好的 Qt 程序，就可以在 vscode 中运行它了，具体操作步骤和运行效果如下图。

![5](实验21使用xmake开发Qt程序.assets/639d40ea0e24c091f7adcb5229fd0f40-0)

#### 在 vscode 中调试 Qt 程序

在 vscode 中调试 Qt 程序的过程，也跟上节实验中的操作方式完全一致：将 release 按钮修改成 debug 字样，然后点击 build 按钮重新编译带有调试符号的目标程序，接下来安装 C/C++ 插件，同时使用命令 `sudo apt install -y gdb` 安装 gdb。

接下来就只需要在我们需要关注的地方下断点。

![6](实验21使用xmake开发Qt程序.assets/21dbdec1be2674fe991b09f1e630f586-0)

我们点击调试按钮，如果一切顺利，就会出现下图的界面，说明断点已经被正常触发了，可以单步调试了。

![7](实验21使用xmake开发Qt程序.assets/fab8f9a1a8cb4dea7af62ac648b1f806-0)

## Qt QuickApp 程序

下面，我们再来讲解 Qt Quick 应用程序的开发，它也是 Qt 提供的一种 Gui 可视化程序框架类型，跟 Qt Widgets 程序的区别在于：

1. Qt Widgets 更老更成熟，但 Qt Quick 更新更加的现代化。
2. Qt Widgets 采用 `.ui` 文件来维护 UI 界面，而 Qt Quick 采用新的 QML 标记语言来描述 UI 界面。

总体比起来，Qt Quick 会使得开发更加的高效易维护，若要开发一些现代化的 UI 程序或者移动端程序，首选 Qt Quick，而对于一些传统的桌面程序就倾向于使用 Qt Widgets 来开发。

接下来，我们详细讲解 Qt QuickApp 程序是如何开发的，这里我们还是以 vscode 编辑器作为 Qt 开发环境。

执行如下命令创建一个 `~/Code/quickapp` 空目录。

```bash
mkdir ~/Code/quickapp
```

然后，在 vscode 中打开这个空目录，如果之前的工程还没关闭，可以先关掉它后再打开，如下图。

![8](实验21使用xmake开发Qt程序.assets/f43b0c54df1867b81befd7ab9be9ec1b-0)

接着我们在这个空工程中创建 Qt QuickApp 的工程文件，还是根据上节实验中所讲的方式，从 View 菜单中打开命令面板，输入 xmake 找到并点击 `XMake: Create Project` 菜单项，如下图。

![9](实验21使用xmake开发Qt程序.assets/11be5c2a3d02d38c891c01b050e55ec1-0)

这个时候，由于当前还没有 xmake.lua 文件，会在底下弹出一个提示框，继续点击里面的蓝色按钮。

![10](实验21使用xmake开发Qt程序.assets/db4ccace608177319f5fd46bb27aa979-0)

然后，我们在显示的项目语言类型列表项中，选择 c++ 项目类型并点击它（因为 Qt 只能是 C++ 工程）。

![11](实验21使用xmake开发Qt程序.assets/69128c29d0402d0512e626d1e0ed980a-0)

接着，会看到工程模板类型列表，我们从中选择 `qt.quickapp` 工程模板，并点击它，如下图。

![12](实验21使用xmake开发Qt程序.assets/4aef1eb31885347349714290316f2fbf-0)

这个时候，我们整个 Qt Quick 工程就创建好了，其跟直接执行 `xmake create -t qt.quickapp quickapp` 命令是同样的效果，只不过这回我们是在 vscode 中去创建它，创建完成后的效果如下图。

![13](实验21使用xmake开发Qt程序.assets/bb657a76f1fd8003135edd123ee48d73-0)

从上图中我们可以看到生成的工程文件其实跟之前的 Qt Widgets 程序还是有很多不同的地方，例如没有了 `.ui` 文件，而是改用 `qml.qrc` 来构建 ui，其最终是在 main.qml 中描述所有的 ui 布局和事件处理。

可以从 xmake.lua 配置中看到，这回改用 `qt.quickapp` 规则来支持 `.qrc` 文件的构建，而不是 `qt.widgetapp` 规则。

```lua
add_rules("mode.debug", "mode.release")

target("quickapp")
    add_rules("qt.quickapp")
    add_headerfiles("src/*.h")
    add_files("src/*.cpp")
    add_files("src/qml.qrc")
```

Qt Quick 的特点之一就是改用 .qml 来描述 Qt 程序的 ui 布局，简单来说，QML 就是一种用户界面规范和标记语言，允许开发人员和设计师创建高性能、流畅的动画和视觉吸引人的应用程序。

我们再来看下生成的 main.qml 里面的内容。

```lua
import QtQuick 2.9
import QtQuick.Window 2.2

Window {
    visible: true
    width: 640
    height: 480
    title: qsTr("Hello World")
}
```

里面的配置主要用于定义 ui 布局，最开头通过 import 引入 qml 的一些在布局中会使用到的模块。

虽然生成了工程，不过如果我们现在就编译运行这个项目，多半会遇到如下错误。

```bash
QQmlApplicationEngine failed to load component
qrc:/main.qml:1 module "QtQuick" version 2.9 is not installed
```

这是由于实验环境安装的 Qt 版本是 5.5，算是一个相对比较老的版本，而 xmake 默认生成的模板工程 main.qml 中，import 导入的是 2.9 高版本的 QtQuick，也就是对应 Qt 5.9 的版本。

为了能够让程序正常编译通过，我们需要改下 main.qml 将 2.9 改成 2.5 即可，如下图。

![14](实验21使用xmake开发Qt程序.assets/330df781a71c3d70fd73b88008ade030-0)

改完后，我们就可以点击 Build 按钮编译这个 Qt Quick 程序了，如果编译通过，会显示下图的结果。

![15](实验21使用xmake开发Qt程序.assets/dcccaab4cddee5feb84802bbf1a66e54-0)

编译通过后，我们再来点击运行按钮运行此程序，如果成功，会看到一个空白的窗口显示出来，就跟 Qt Widgets 的效果差不多。

![16](实验21使用xmake开发Qt程序.assets/f5c34c92a168fd838eebe35cacbe0710-0)

另外，至于刚刚说的 main.qml 中 Qt Quick 版本与 Qt 版本如何对应，我们可以参考 Qt 的官方文档 [Qt 和 Qt Quick 历史版本对照表](https://doc.qt.io/qt-5/qtquickcontrols-index.html#versions)，也就是下图所示。

![17](实验21使用xmake开发Qt程序.assets/36190bedd27310a78a3774e380ce1c91-0)

虽然，上面没有我们当前 Qt 5.5 的版本，但是依次从上往下推算，Qt 5.5 差不多就是对应 Qt Quick 2.5 版本。

#### Quick QML 布局配置修改

虽然我们的实验课程只要讲解 xmake 的使用，不过为了更加深刻的体验使用 xmake 来开发构建 Qt 程序，我们最后以一个实际的例子为例，在之前的 quickapp 工程的基础上，增加一个 Button 按钮部件响应点击事件，去随机改动窗口的背景色。

其实我们只需要改动 main.qml 里面的 ui 布局代码，就能完成所需功能，修改后的内容如下。

```lua
import QtQuick 2.5
import QtQuick.Window 2.2
import QtQuick.Controls 1.2

Window {
    id: root
    visible: true
    width: 640
    height: 480
    title: qsTr("Hello World")
    Button {
        text: "Ok"
        onClicked: {
            root.color = Qt.rgba(Math.random(), Math.random(), Math.random(), 1);
        }
    }
}
```

通过引入 Button 部件到当前的窗口中，然后定义 onClicked 事件去修改 root 窗口的背景色，修改完成后。

每次改完任何代码和 qml 配置后，都需要重新编译。为了执行重新编译，我们需要从菜单里面的命令面板中，找到 xmake 的 Rebuild 命令，点击执行才行。

从下图的红色箭头位置找到对应的命令面板。

![图片描述](实验21使用xmake开发Qt程序.assets/cc869c03320ac1d67f7a7c219e0b105a-0)

然后再打开的命令面板中输入 xmake 过滤出所有跟 xmake 相关的命令列表，找到 Rebuild 命令后点击编译即可，如下图。

![25](实验21使用xmake开发Qt程序.assets/5b3b425dc410962e44e3974096ef1267-0)

最后点击运行，首次运行的效果如下，我们会额外看到一个 Button 按钮在窗口中。

![18](实验21使用xmake开发Qt程序.assets/2a213ff4ef4755353d02abbfbf35fa13-0)

点击这个 Button 按钮，就会看到窗口的颜色开始随机变动了，如下图。

![19](实验21使用xmake开发Qt程序.assets/2bc006990550891659c5aa0d7b8f553f-0)

所以，我们也能通过此实验确定即使不使用 Qt Creator，仅仅使用 vscode 和 xmake 也是可以正常开发和构建 Qt 应用程序的，并没有太大的区别。

#### 添加 Qt 依赖库

除了 QtGui、QtCore 等基础 Qt 库，我们还可以通过 `add_frameworks` 接口去配置一些额外的 Qt 依赖库。

它跟 `add_links` 的区别在于：`add_links` 仅仅只会简单地将库作为 `-lQtGui` 方式来链接；而 add_frameworks 主要用于添加 QT 相关库，除了会添加 `-lQtGui` 之外，还会添加 `-I/xxx/QtGui` 等头文件搜索路径，以及链接搜索路径，另外还会添加对应的 `-DQT_XML_LIB` 宏定义。

我们修改刚刚生成的 Qt Quick 项目中的 xmake.lua 文件内容，通过 `add_frameworks("QtXml")` 添加 QtXml 依赖库，详细配置如下。

```lua
add_rules("mode.debug", "mode.release")

target("quickapp")
    add_rules("qt.quickapp")
    add_headerfiles("src/*.h")
    add_files("src/*.cpp")
    add_files("src/qml.qrc")
    add_frameworks("QtXml") -- 添加 QtXml 依赖库
```

这回，我们在命令行下运行 `xmake -rv` 命令，查看详细的编译命令参数。

```bash
cd ~/Code/quickapp
xmake -rv
```

可以看到，虽然我们仅仅只是添加了 `add_frameworks("QtXml")` 配置，但是 xmake 会自动追加与使用 QtXml 库相关的所有编译链接选项，也就是下图的红线位置部分，这比起手动使用 `add_links`、`add_includedirs` 等挨个设置比起来更加的方便，也更加的跨平台。

![20](实验21使用xmake开发Qt程序.assets/65e8d697b8394a956b68ff76f843e5cf-0)





## 实验总结



在本节实验中，我们了解了 Qt 的一些基本概念，也学习了如何去开发编译 Qt Widget 和 Qt Quick 应用程序，另外我们还学会了使用 vscode 编辑环境去开发 Qt 程序。

本实验的参考代码可以使用如下命令下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/2764/code21.zip
```