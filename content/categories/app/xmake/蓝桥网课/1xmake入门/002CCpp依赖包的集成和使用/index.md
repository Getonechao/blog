+++

title= "实验2C/C++ 依赖包的集成和使用"
description= "蓝桥网课xmake笔记"
date= 2021-07-15T10:36:40+08:00
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

# C/C++ 依赖包的集成和使用

在本节实验中，我们可以学习到如何快速集成第三方的 C/C++ 依赖包，以及与第三方包仓库的接入集成。

#### 知识点

- xmake 包管理的整体架构流程
- 系统库以及第三方库的快速集成
- Conan 包管理的介绍
- 语义版本的配置说明
- 依赖包的安装和卸载
- zlib 库的集成和使用



## C/C++ 依赖库介绍



C/C++ 库不像其它新的高级语言那样官方提供好用的依赖包管理和集成，整个生态比较杂乱不统一，集成起来非常的不方便，这也是 C/C++ 开发的痛点之一。

我们经常需要到处找库，研究如何编译、安装它们，并且如何去集成使用，如果需要跨平台，遇到的问题可能还更多点，为了集中管理这些依赖包，现在市面上已经有不少成熟的第三方包管理器来维护和解决这些问题，比如：conan，vcpkg，homebrew 等等，不过我们还是需要有一种更加方便的集成方式将其与我们的 C/C++ 项目工程进行关联集成，才能使用它们。

为此，xmake 内置了对这些第三方包管理仓库的集成，另外 xmake 还有自己的包依赖仓库来更好的解决依赖包集成问题，我们将在下面的实验中逐一讲解。

#### 直接集成使用系统库

首先，我们先从最简单的集成方式开始，尝试直接集成系统中已经安装的第三方库，由于 zlib 库在大多数 Linux 系统环境中都是内置了的，并且在本实验环境也是已经存在的，那么我们就先从集成 zlib 库开始实验。

由于 zlib 是 C 库，为了方便起见，这回我们来创建一个 C 工程来尝试它，只需要在创建工程的时候指定下语言即可，例如。

```bash
cd ~/Code
xmake create -l c crc32
```

`-l c` 参数指定当前创建的 crc32 工程是基于 C 语言的，其目录结构如下，跟之前的 hello 项目很相似，仅仅只是 `main.cpp` 源文件变成了 `main.c`。

![1](002CCpp依赖包的集成和使用.assets/a05183d1a967b3542ce02f42dde97ec9-0)

进入 `crc32` 目录中，执行 `vim src/main.c` 编辑源文件，编写一个简单实用、使用 zlib 库中 `crc32()` 接口去计算 crc32 的例子，代码如下。

```c
#include <stdio.h>
#include <zlib.h>

int main(int argc, char** argv)
{
    printf("crc32: %x!\n", crc32(0, argv[1], strlen(argv[1])));
    return 0;
}
```

上述代码通过计算输入字符串内容的 crc32 值，然后回显输出到终端上，我们先尝试下直接执行 `xmake` 命令去编译它。

![2](002CCpp依赖包的集成和使用.assets/c54ce38d76f600527b21fc69b7aa6b7f-0)

通过上图，我们看到编译链接阶段报错了，缺少 `crc32` 符号，这是因为我们还没集成链接 zlib 库，继续修改 xmake.lua 文件手动加上 zlib 库以及它系统库搜索路径。

```lua
target("crc32")
    set_kind("binary")
    add_files("src/*.c")
    add_links("z")
    add_linkdirs("/usr/lib/x86_64-linux-gnu")
```

然后重新执行 `xmake` 编译这个工程，这回看到编译通过了。

![3](002CC++依赖包的集成和使用.assets/753ae6438fecb35e2aa6790b3d760361-0)

如果我们执行 `xmake -rv` 命令，是可以看到详细编译输出里面已经带上了 `-L/usr/lib/x86_64-linux-gnu -lz` 的库链接选项。

注：这里没有额外添加头文件搜索路径，是因为 `zlib.h` 是在 `/usr/include` 目录下，这个目录 xmake 会内置自动添加对应的 `-I` 选项到 gcc。

![4](002CC++依赖包的集成和使用.assets/d601e96ec7b350b3940f5cb360188c87-0)

接下来，我们尝试执行下面的命令运行这个程序。

```bash
xmake run crc32 somedata
```

如果运行正常，会看到类似下面的运行结果，说明我们已经正常集成使用了系统自带的 zlib 库。

![5](002CC++依赖包的集成和使用.assets/99590274624424105b8c30b3b438028f-0)

#### 自动查找系统库

在上节中，我们通过 `add_linkdirs("/usr/lib/x86_64-linux-gnu")` 方式来集成系统依赖库，虽然可以使用但不是非常通用，因为带有 xmake.lua 的 C/C++ 项目有可能会在其它 Linux 环境甚至在其它系统上编译，zlib 库并不一定安装在这个目录下。

为了更加通用化的适配系统库路径，可以使用 `find_packages` 来自动查找 zlib 库所在的位置，通过这种方式集成之前，我们可以先尝试直接执行下面的命令，来探测 zlib 库的位置信息，对这个接口返回的信息有个大概的了解。

```bash
xmake l find_packages zlib
```

如果我们成功找到系统路径下安装的 zlib 库，那么会看到类似下图的结果。

![图片描述](002CC++依赖包的集成和使用.assets/37ae4573514eff6a84187455f8bcc018-0)

里面有链接库名、链接库的搜索路径，有可能还会有头文件的搜索路径等信息，因此我们只需要把这些信息自动探测到后设置到对应的目标程序中，就可以实现自动集成安装在系统环境的依赖库了。

接下来，我们继续修改 xmake.lua，在 target 定义下面新增一个 `on_load()` 脚本块，在里面调用 `find_packages("zlib")` 将探测到的 zlib 库信息直接动态设置到 target 里面去就行了，如下配置代码。

```lua
target("crc32")
    set_kind("binary")
    add_files("src/*.c")
    add_links("z")
    add_linkdirs("/usr/lib/x86_64-linux-gnu")
    on_load(function (target)
        target:add(find_packages("zlib"))
    end)
```

修改完成后执行 `xmake -rv`，其编译输出应该跟之前的输出完全一致才对。

不过需要注意的一点是：xmake 默认会缓存依赖包的检测结果，并不是每次编译都会重新检测，如果之前检测失败，那么结果也会缓存，这个时候我们可以执行 `xmake f -c` 在配置时候，忽略之前的缓存内容，就会自动重新触发各种检测。

![4](002CC++依赖包的集成和使用.assets/d601e96ec7b350b3940f5cb360188c87-0-1626760600064)



## 远程库下载集成

我们现在虽然能够自动检测和集成系统环境的 C/C++ 库，但是如果需要的库在当前环境中还没有安装，那么还是需要手动的去安装它们，如果有些库安装非常复杂容易出错，那么整个过程也是很折腾的。

因此 xmake 提供了依赖包的自动远程下载以及安装集成功能，它不仅支持 conan、vcpkg、homebrew 等第三方包管理仓库，还支持自建的分布式私有包管理仓库，并且 xmake 也提供了官方的 [C/C++ 包仓库](https://github.com/xmake-io/xmake-repo)。

其大致流程可以通过下面这个图来直观的了解。

![7](002CC++依赖包的集成和使用.assets/308009fc6eea85743abe41e97cc60d73-0)

如果暂时看不懂上图的流程，没有关系，我们先来个简单的例子体验下如何远程下载依赖包并把它集成到项目中去，还是拿之前的 zlib 为例，我们先假设当前系统环境中没有这个库存在，想要从网上拉取对应版本进行集成。

还是拿之前的 crc32 工程做下修改，编辑里面的 xmake.lua 文件，修改为下面的内容。

```lua
add_requires("zlib", {system = false})

target("crc32")
    set_kind("binary")
    add_files("src/*.c")
    add_packages("zlib")
```

其中 `add_requires()` 接口指定当前项目需要哪些包，配置这个编译时候会触发一次依赖包的安装，而 `add_packages()` 用来配置对特定 target 目标集成指定的依赖包，这两者需要配合使用，缺一不可。

接下来，执行 `xmake` 重新来编译项目，如果出现下面的提示信息，说明我们配置的 `zlib` 包有在官方仓库中被收录，并且当前平台支持这个依赖包的集成。

如果安装完成并且编译成功会显示下面的信息。

![图片描述](002CC++依赖包的集成和使用.assets/c6d3c0fbcc9427ea787c82c686bccb98-0)

需要注意的一点是，上面的配置中我们额外加上了 `{system = false}` 是因为当前的实验环境中已经存在了 zlib 库，所以 `add_requires("zlib")` 默认会优先自动检测系统环境中的 zlib 库，如果存在就直接使用，也就是内置了 `find_packages("zlib")` 的逻辑。

而这里出于演示远程下载的逻辑，我们通过配置 `{system = false}` 强制触发远程下载，人为忽略了系统库的探测逻辑，在实际项目中，大家可根据自己的需求来决定是否配置这个选项。

另外一个注意事项是，我们的包检测结果都是有本地缓存的，第二次编译并不会再触发依赖包的下载安装，会直接参与集成编译。

所以再次执行 `xmake -rv` 重新编译看看，这次并不会再去安装 zlib 库了，同样可以正常集成 zlib 来重新编译项目，效果如图。

![9](002CC++依赖包的集成和使用.assets/5a098d59fc91cbee44f0a0f04a5f7d0a-0)

上图中红线部分就是我们从远程仓库集成的 zlib 库头文件和链接信息。

#### 卸载和重装依赖包

如果我们想要卸载之前的依赖包，只需要执行卸载命令。

```bash
xmake require --uninstall zlib
```

如果看到下图，说明已经成功卸载了安装包。

![10](002CC++依赖包的集成和使用.assets/a3bb42bbecb4b8c0eb5c5828d0ab184a-0)

不过需要注意的是，虽然卸载了包，但是当前项目的配置缓存还在，如果继续执行 `xmake` 命令，还是会使用缓存的 zlib 包信息。

因此可以添加 `-c` 参数给 `xmake config`，也就是执行下 `xmake f -c` 命令，强制忽略缓存配置，触发依赖包的重新检测逻辑。

由于 xmake.lua 中还是配置了 zlib 包依赖，因此这次会触发重新安装逻辑（不过不会再重新下载了，而是直接解压安装之前缓存的 zlib 包），效果如图。

![图片描述](002CC++依赖包的集成和使用.assets/4d691d5278b4ed5ecf4d1432894e7702-0)

#### 语义版本设置

xmake 的依赖包管理是完全支持语义版本选择的，例如："~1.6.1"，对于语义版本的具体描述见：https://semver.org/。

这里，我们在之前的 xmake.lua 基础上，新增两个依赖库，并且设置上不同的语义版本，例如。

```lua
add_requires("zlib", {system = false})
add_requires("tbox 1.6.*", "pcre2 >10.0")

target("crc32")
    set_kind("binary")
    add_files("src/*.c")
    add_packages("zlib", "tbox", "pcre2")
```

其中 [tbox 库](https://github.com/tboox/tbox) 我们会集成匹配 `1.6` 下面的所有可用子版本，而 pcre2 正则库我们会集成匹配 `>10.0` 的可用版本。配置好后，继续执行 `xmake` 安装依赖库编译，如果一切顺利，会看到类似下面的结果输出。

![图片描述](002CC++依赖包的集成和使用.assets/ec17829315411b65bdf2620a0bc74a2d-0)

再来执行 `xmake -rv` 查看完整命令输出，看看是否真的集成上了这两个库。

![13](002CC++依赖包的集成和使用.assets/180a63868e4e68ec8eab0c57d700e5d5-0)

上图红框部分，就是我们新下载集成的两个依赖库的路径和编译链接选项，说明我们已经顺利集成上了，至于 C 代码里面如何去使用它们，这里就不再过多介绍了，大家在实验后可以自行研究。

当然，如果我们对当前的依赖包的版本没有特殊要求，那么也可以不设置版本，xmake 会默认拉取安装当前可以获得的最新版本包。

如果当前依赖包有 git 仓库，也可以集成 git 仓库的 `master/dev` 分支依赖包，只需要把版本号换成对应的分支名就行了，例如。

```lua
add_requires("tbox master")
add_requires("tbox dev")
```

后续编译操作跟之前的类似，这里就不再重复实验了。

#### 查看依赖库信息

我们可以执行下面的命令，快速查看通过 `add_requires("zlib")` 配置的依赖包信息，比如查看实际的安装路径、缓存目录、可选配置信息等。

```bash
xmake require --info zlib
```

相关的包信息说明可以参考下面的图片。

![14](002CC++依赖包的集成和使用.assets/854df30bf6d0b501080cae667db8dcdb-0)

## Conan 仓库依赖包的集成

虽然 xmake 内置的包仓库功能已经很完善，但毕竟生态还没完全建立起来，收录的包还不是很多，如果用户需要的 C/C++ 依赖包在我们的包仓库中找不到，那么还可以直接集成使用第三方仓库中提供的包，例如：Conan、Vcpkg、Homebrew 等仓库，xmake 都是支持的。

这里我们来重点尝试下集成 Conan 中提供的包，因为相比 Vcpkg、Homebrew 等仓库，Conan 支持更加完善，不仅自身跨平台，提供的包也是支持多个不同平台的，收录的包数量也非常多。

在使用 [Conan](https://conan.io/) 之前，首先需要安装它到系统环境中，由于 conan 是基于 python 的，因此可以通过 pip 工具来安装它，只需要执行下面的命令。

```bash
sudo pip install --upgrade pip
sudo pip install conan
source ~/.profile
```

然后我们可以通过执行 `conan --version` 确认下是否安装成功，如果成功，我们会看到对应的版本信息，如图。

![图片描述](002CC++依赖包的集成和使用.assets/cb007cef65df38649f5f2d6a368e664d-0)

接下来，使用 vim 编辑之前的 crc32/xmake.lua 文件，改成如下配置。

```lua
add_requires("CONAN::zlib/1.2.11@conan/stable", {alias = "zlib"})

target("crc32")
    set_kind("binary")
    add_files("src/*.c")
    add_packages("zlib")
```

这里我们使用 `CONAN::` 作为 conan 包仓库的命名空间，让 xmake 将其作为 conan 依赖包来处理后续的探测、下载和安装逻辑。xmake 内部会自动调用 conan 去安装指定的 zlib 包，并且集成进来，因此这里的 `zlib/1.2.11@conan/stable` 包描述格式也是完全基于 conan 自身的规范，大家可以直接到 [conan 官方文档](https://docs.conan.io/en/latest/using_packages/conanfile_txt.html#requires) 查看如何设置。

而后面的 `{alias = "zlib"}` 部分是 `add_requires` 接口对于每个包的扩展配置，这里 alias 用来设置包的别名，方便之后每个 target 中更加精简的去集成，例如：`add_packages("zlib")`，而不是每次都是用全名。

配置完成后，执行 `xmake` 去编译这个集成了 conan 依赖包的项目，整个依赖包的下载安装流程，跟之前完全一致，如图。

![图片描述](002CC++依赖包的集成和使用.assets/7bf090db28cbba13a5dd35c7fb0117f8-0)

如果安装成功，可以继续执行 `xmake -rv` 来查看 xmake 实际链接的 conan 包路径信息。

![17](002CC++依赖包的集成和使用.assets/cf6f6b2d7930d3782459425fff3bf90c-0)

通过上图可以看到，xmake 直接引用了安装在 conan 自身仓库目录下的 zlib 库，说明整个集成流程已经通过。



## 实验总结



在本节实验中，我们主要学习了如何查找集成依赖库，如何集成使用 xmake 官方包仓库中提供的 C/C++ 依赖包，另外我们也学习了如何接入 Conan 等第三方包仓库中的依赖包。

本实验的参考代码可以使用如下命令下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/2764/code4.zip
```