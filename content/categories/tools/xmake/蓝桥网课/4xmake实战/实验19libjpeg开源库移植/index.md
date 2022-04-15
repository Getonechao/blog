+++

title= "实验19libjpeg 开源库移植"
description= "蓝桥网课xmake笔记"
date= 2021-07-15T10:37:10+08:00
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

# libjpeg 开源库移植

在本节实验中，我们会学习到如何使用 xmake 去移植编译第三方库 libjpeg 源码，以及如何将移植好的库快速集成进自己的 C/C++ 工程中去使用。

#### 知识点

- autotools 介绍和使用
- xmake 的自动扫描和生成
- libjpeg 库源码移植编译的详细流程
- libjpeg 库的快速集成



## 第三方代码库的移植编译

通过之前的实验，我们已经基本学习了解了 xmake.lua 的配置语法，也学习了如何通过这个文件配置构建 C/C++ 项目。因此，在本节实验中，我们通过实战的方式，以一个实际的第三方开源基础库 libjpeg 为例，讲解如何通过使用 xmake 去对它进行移植和编译。

libjpeg 是一个用于 jpeg 图片文件解码的开源基础库，其代码本身是完全跨平台的，但是其内部的构建系统却非常的混乱，有提供对于 Linux 系统的 autotools，也有对于 msvc 编译器的 bat 构建脚本。不同的平台，构建方式差异很大，大家如果想编译不同平台的库，都需要花上一定的时间去研究各种平台下对 libjpeg 的编译过程。

而使用 xmake 就可以进一步简化对于这个库的编译，并且还提供了如何去快速集成的方法。不过，在实际的讲解前，我们还需要先从官网下载 libjpeg 的源码，并通过 tar 命令解压到 jpeg-9d 目录下。

整个下载和解压过程如下。

```bash
cd ~/Code
wget http://www.ijg.org/files/jpegsrc.v9d.tar.gz
tar -xvf jpegsrc.v9d.tar.gz
```

#### 使用 xmake 调用 autotools 直接编译

解压完成后，执行 `ls` 先看下这个项目的根目录下有哪些跟构建系统相关的文件，我们发现，libjpeg 其实存在 `./configure` 配置脚本，这是 autotools 构建工具的主要配置工具。这就说明 libjpeg 其实可以直接用 autotools 来编译的，只不过 autotools 并不支持 windows 以及 msvc 编译器。

既然存在 `./configure`，那么 xmake 也可以直接调用它来完成整个项目的构建，例如执行下面的命令。

```bash
$ cd jpeg-9d
$ xmake
note: configure found, try building it or you can run `xmake f --trybuild=` to set buildsystem (pass -y or --confirm=y/n/d to skip confirm)?
please input: y (y/n)
```

只需要执行一个 xmake 命令，xmake 就能自动探测当前的项目支持哪些构建系统，如果有 xmake.lua 文件，那就直接按 xmake 的配置编译，如果有 `./configure` 那就提示用户是否使用 autotools 构建系统去常识性编译它（另外 xmake 还支持 cmake、meson 等构建系统文件的探测和尝试编译）。

看到下图提示后，我们可以按 `y` 或者默认回车，来告诉 xmake 请尝试直接调用 autotools 来编译它。

![1](实验19libjpeg开源库移植.assets/01010555512bf567774ce3ebb89b585a-0)

编译成功完成后，xmake 会把构建生成产物放置在 `build/artifacts` 目录下，实际输出内容，如下图。

![2](实验19libjpeg开源库移植.assets/c12927be5b16faae966a47dbb346656d-0)

#### 自动生成 xmake.lua 移植编译 libjpeg

刚刚我们介绍了使用 xmake 直接去调用 autotools 来编译，但这种方式毕竟依赖第三方的构建系统，有很多局限性，比如 autotools 不支持 windows，那就无法做到跨平台编译了。

这个时候，我们可以考虑将 libjpeg 的所有源码移植到 xmake.lua 的配置文件中，然后利用 xmake 内部的构建系统直接去编译它，这样就能做到一次移植，多平台编译支持。

不过从现有的构建系统移植到 xmake.lua 还是比较费体力的，为此 xmake 提供了一个辅助功能，它可以扫描当前目录下的所有源代码，然后自动帮我们生成一个基本可用的 xmake.lua 出来，当然自动生成的配置文件，多少会存在一些小问题，但既然有个范本，我们在上面做些小改动调整，就能完成整个构建移植工作。相比起完全手动配置还是省事不少的。

为了启用 xmake 的自动扫描特性，首先得确保当前目录下没有其它的第三方构建系统文件，否则 xmake 检测到后，会优先走尝试调用第三方构建系统来编译的逻辑。

所以我们先执行下面的命令，删除 libjpeg 自带的 autotools 相关配置文件。

```bash
rm ./config*
```

删除后执行 xmake 命令，触发源文件的扫描提示，我们只需要按 `y` 后回车，确认生成即可。

```bash
$ xmake
note: xmake.lua not found, try generating it (pass -y or --confirm=y/n/d to skip confirm)?
please input: n (y/n)
```

提示内容如下图所示。

![3](实验19libjpeg开源库移植.assets/d0b6284a8c3a1604e0083419bb9201cd-0)

我们根据提示，按下 `y` 和回车后，xmake 就会开始自动扫描 libjpeg 库下的所有源文件，如图。

![4](实验19libjpeg开源库移植.assets/9051da171b874a23d52e39d03e81fe5b-0)

还会扫描用到 libjpeg 库的一些可执行程序，如下图。

![5](实验19libjpeg开源库移植.assets/f814800fc432321e5102f7ec9cb69a7e-0)

然后继续编译，编译到一半的时候，触发了编译错误，看下图的错误信息，应该是编译 jmemmac.c 文件失败了。

![6](实验19libjpeg开源库移植.assets/e78c3854de7716293dba898a220249ad-0)

毕竟，自动扫描再怎么智能，也不可能完全可靠，这里只是为了提高一些移植效率，如果编译发生错误，只需要分析下错误原因，对 xmake.lua 里面的配置稍作修改就行了。

看这里的错误，既然是 jmemmac.c 失败了，看文件名，应该是 mac 平台相关的，而我们现在是在 Linux 下，那说明这个源文件不应该参与编译。

因此，我们只需要在 xmake.lua 中删除对这个文件的配置就行了。

![7](实验19libjpeg开源库移植.assets/4189cab357ac9f99b7071bbe58f8f48b-0)

删除后，我们继续执行 xmake 编译 libjpeg 项目，这个时候我们又发现了其它类似的编译错误，这回是 jmemdos.c 文件，似乎也是跟 libjpeg 的内存分配相关代码。

![8](实验19libjpeg开源库移植.assets/04fccaeef5a71581d4e2469f8da5db61-0)

连续碰到 jmemmac.c 和 jmemdos.c 的编译失败，我们大体能知道 libjpeg 提供了不同平台对应的 jmemxxx.c 内存分配实现。既然如此，我们再找下其它跟内存相关的平台代码文件，发现还有一个 jmemdosa.asm 也是，那么我们就一并注释掉这两个文件。

![9](实验19libjpeg开源库移植.assets/73da6000ba821cb20a3d8c8f310c1c4f-0)

注释掉这三个不相关的文件后，继续执行 xmake 命令就能顺利通过编译了，如下图。

![10](实验19libjpeg开源库移植.assets/b357cbbcff6b7773efb115a1bffa291a-0)

#### 手动配置 xmake.lua 移植编译

刚刚我们通过自动生成 xmake.lua 文件的方式，也已经完成了 libjpeg 库的移植编译，但是生成的 xmake.lua 文件内容还比较冗余和粗糙，不够精简。

既然我们现在已经大概了解 libjpeg 库的源码结构，其 Linux 下实际仅仅使用了 jmemansi.c 文件参与编译，其它 `jmem*.c` 代码文件我们可以手动配置去排除它。

因此修改并精简 xmake.lua 配置如下。

```lua
add_rules("mode.debug", "mode.release")

target("jpeg")
    set_kind("static")
    add_files("*.c|jmem*|wrjpgcom.c|cjpeg.c|rdjpgcom.c|ckconfig.c|djpeg.c|jpegtran.c")
    add_files("jmemansi.c", "jmemmgr.c")
```

通过 `add_files("*.c|jmem*")` 的方式，我们就可以在添加匹配大部分代码文件的同时，忽略掉其它内存分配代码文件的编译。另外，我们也知道 wrjpgcom.c、cjpeg.c 和 rdjpgcom.c 这些文件，其实只是可执行程序的主文件，并不是 libjpeg 库的内部文件，编译静态库不需要参与进来，那么也可以一并排除掉，就如上面的配置所示。

手动精简后的配置仅仅只有简单的几行，就能做到相同的编译效果，我们重新执行下 `xmake -r` 命令，看看是否能够正常完成整个库的编译，如果一切正常，就会如下图所示。

![11](实验19libjpeg开源库移植.assets/08d90a94e405340ae395c28a497260d5-0)



## 远程包的依赖集成

通过手动移植的方式，我们虽然能够很好的完成第三方库的编译，但是集成起来还是比较繁琐的，如果把 libjpeg 库代码内置到项目中去，就会非常的臃肿。而如果仅仅提供编译后的库给其它项目链接使用，编译和版本维护又非常的麻烦。

然后我们已经完成了编译，就可以把整个编译移植过程记录下来，通过 `package("libjpeg")` 方式定义一个第三方包，内部描述整个库的编译过程。

而大家只需要在自己的项目中通过 `add_requires("libjpeg")` 引用进来，就可以实现远程库的自动下载、自动编译以及无缝集成。

下面，我们先来创建一个新的空工程，用于测试 jpeg 库的自动移植和集成。

```bash
cd ~/Code
xmake create jpegtest
```

创建完成后进入 jpegtest 目录，我们先来修改 `src/main.cpp` 加上对 jpeglib.h 库头文件的引用，以及对应 c 接口的调用。

```c++
#include "stdio.h"
#include "jpeglib.h"

int main(int argc, char** argv)
{
    jpeg_create_compress(0);
    return 0;
}
```

然后开始编辑 xmake.lua 文件，在里面定义一个 `package("myjpeg")` 的第三方包，内部使用 autotools 构建系统去构建安装 jpeg 库，然后通过 `add_requires("myjpeg")` 引入到工程中来，并且通过 `add_packages("myjpeg")` 的方式添加到对应的 jpegtest 目标程序中去进行集成和使用。

相关配置内容如下。

```lua
add_rules("mode.debug", "mode.release")

package("myjpeg")
    set_urls("http://www.ijg.org/files/jpegsrc.$(version).tar.gz")
    add_versions("v9d", "99cb50e48a4556bc571dadd27931955ff458aae32f68c4d9c39d624693f69c32")
    on_install(function (package)
        import("package.tools.autoconf").install(package)
    end)
package_end()

add_requires("myjpeg")

target("jpegtest")
    set_kind("binary")
    add_files("src/*.cpp")
    add_packages("myjpeg")
```

整个配置非常的简单，我们通过 `add_versions` 将 jpeg v9d 版本的对应源码包的 sha256 哈希值配置上，为了确保下载文件的完整性。

只需 sha256 的计算和获取，可以通过 xmake 提供的 `xmake l hash.sha256 jpegsrc.v9d.tar.gz` 命令来获取。

注：这里我们使用 `add_requires("myjpeg")` 而不是 libjpeg 或者 jpeg，是为了确保 xmake 能够使用我们定义的 `package("myjpeg")` 包，因为 libjpeg 包已经被收录到 [xmake 的官方仓库](https://github.com/xmake-io/xmake-repo) 中，这会导致命名冲突。

而不用 jpeg 这个名称，是因为系统的 pkg-config 默认会检测到 jpeg 库，所以 `add_requires("jpeg")` 会优先使用系统安装的 jpeg 库。

最后，执行 xmake 的编译效果如下图，我们完全复用了整个 jpeg 库的移植过程，不用每次都去手动移植一遍，也可以快速集成。

![12](实验19libjpeg开源库移植.assets/f5eb664813f191a62b35156754e1b158-0)

另外，我们还可以改进下 `package("mytest")` 包定义，内部不使用 autotools 而是直接使用刚刚我们移植的 xmake.lua 来构建它，这样就能在不同平台下集成 jpeg 库了，修改 xmake.lua 如下。

```lua
add_rules("mode.debug", "mode.release")

package("myjpeg")
    set_urls("http://www.ijg.org/files/jpegsrc.$(version).tar.gz")
    add_versions("v9d", "99cb50e48a4556bc571dadd27931955ff458aae32f68c4d9c39d624693f69c32")
    on_install(function (package)
        io.writefile("xmake.lua", [[
            add_rules("mode.debug", "mode.release")

            target("jpeg")
                set_kind("static")
                set_kind("static")
                add_files("*.c|jmem*|wrjpgcom.c|cjpeg.c|rdjpgcom.c|ckconfig.c|djpeg.c|jpegtran.c")
                add_files("jmemansi.c", "jmemmgr.c")
                add_configfiles("jconfig.txt", {filename = "jconfig.h"})
                add_headerfiles("jerror.h", "jmorecfg.h", "jpeglib.h", "$(buildir)/jconfig.h")
        ]])
        import("package.tools.xmake").install(package)
    end)
    on_test(function (package)
        assert(package:has_cfuncs("jpeg_create_compress(0)", {includes = {"stdio.h", "jpeglib.h"}}))
    end)
package_end()

add_requires("myjpeg")

target("jpegtest")
    set_kind("binary")
    add_files("src/*.cpp")
    add_packages("myjpeg")
```

这次我们在 `package("myjpeg")` 的 on_install 里面，写入移植后的 xmake.lua 文件，然后通过 xmake 来编译它。

在重新执行 xmake 编译前，我们需要先执行下面的命令，移除之前已经安装的 myjpeg 包，这样才能重新发出 myjpeg 库的编译安装逻辑。

```bash
xmake require --uninstall myjpeg
```

移除掉之前安装的 myjpeg 包后，执行下面的命令重新触发 myjpeg 包的安装和编译集成。

```bash
xmake f -c
xmake -r
```

整个执行过程如下图，如果一切顺利，则会完成整个工程的编译。

![13](实验19libjpeg开源库移植.assets/84c2225050963e79de5547d6826e0f08-0)

#### 配置文件的自动生成

在上面的 `package("myjpeg")` 包移植中，我们还额外加入了一行配置。

```lua
add_configfiles("jconfig.txt", {filename = "jconfig.h"})
```

这是由于 jpeg 库对外提供的头文件，需要有一个 jconfig.h 的配置头文件，但是其源码中默认是没有的，只有一个 jconfig.vc 的模板配置文件，在使用 autotools 编译时候，它会自动通过这个文件生成对应的 jconfig.h，而在我们移植后的 xmake.lua 中也需要自动去生成它。

这个时候，我们可以使用 add_configfiles 接口，配置上它就可以将其自动生成到 `build/jconfig.h`。

然后，我们又通过下面的配置，将自动生成的 jconfig.h 以及其它头文件一起作为安装文件，在安装集成的时候导出到了 include 安装目录下。

```lua
add_headerfiles("jerror.h", "jmorecfg.h", "jpeglib.h", "$(buildir)/jconfig.h")
```

这样，我们在自己的工程里面就可以正常通过 `#include "jpeglib.h"` 使用 libjpeg 的头文件了。

#### 直接集成使用第三方库

通过远程依赖集成的好处是第三方库的源码不需要内置到项目中去，当然如果像 libjpeg 这种小库，代码量不大，那么直接集成进去使用，也许会更加方便。

要直接进行代码内置集成，我们需要先将刚刚的 jpeg-9d 源码目录放置进来。

```bash
cd jpegtest
mv ../jpeg-9d .
```

然后修改 xmake.lua 文件，通过 includes 的方式引入刚刚在 jpeg-9d 下移植完成的 xmake.lua，例如。

```lua
add_rules("mode.debug", "mode.release")

target("jpegtest")
    set_kind("binary")
    add_files("src/*.cpp")
    add_deps("jpeg")

includes("jpeg-9d")
```

在上面的配置中，我们看到只需要通过 `includes("jpeg-9d")` 引入刚刚移植好的 jpeg-9d/xmake.lua 后，将其作为子工程，然后通过 `add_deps("jpeg")` 将 jpeg 库关联到指定目标程序即可。

执行 `xmake -r` 进行编译，如果一切顺利，那么整个项目都会完成编译，结果如下图。

![14](实验19libjpeg开源库移植.assets/dc9a7d51a6727ac8f8d861ae7aa2e2d5-0)

## 实验总结



在本节实验中，我们通过一个完整的例子学习了如何使用 xmake 来移植编译 libjpeg，并且也学习了如何将移植后的 libjpeg 集成到项目工程中去。整个过程中，我们还学习到了 xmake 的自动扫描和生成模式以及 autotools 的基本概念和使用。

本实验的参考代码可以使用如下命令下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/2764/code19.zip
```