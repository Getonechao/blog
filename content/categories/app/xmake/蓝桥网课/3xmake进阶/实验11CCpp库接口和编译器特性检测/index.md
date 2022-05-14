+++

title= "实验11C/C++ 库接口和编译器特性检测"
description= "蓝桥网课xmake笔记"
date= 2021-07-15T10:36:50+08:00
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

# C/C++ 库接口和编译器特性检测

在本节实验中，我们会学习到 C/C++ 跨平台开发中一些基础知识和注意点，以及如何使用 xmake 去检测不同编译环境中的 C/C++ 库头文件、库接口是否存在，以及编译器特性相关的检测方式。

#### 知识点

- 跨平台编译开发相关基础知识，不同编译器之间的差异性
- 头文件、库接口的存在性检测
- C/C++ 代码片段的检测
- 编译器特性的支持力度检测



## 跨平台开发介绍

如果 C/C++ 项目在开发和构建中要考虑跨平台问题，需要同时支持 Linux、macOS 和 Windows 系统，甚至还要支持 Android 和 IOS 等移动端系统，那么我们需要考虑很多跟平台相关的一些差异化因素，才能支持跨平台，其涉及并需要解决的一些平台差异有如下这些。

1. 代码差异，比如依赖的系统库 API 不同，不同编译器对 C++ 标准的支持力度不同。
2. 编译工具链的差异，以及对应的编译器特性、编译选项差异。
3. 运行时环境的差异。

抛开运行环境不谈，如果要在不同平台提供的编译工具链上通过编译，首先要解决代码自身支持跨平台的问题，然后不同的工具链、系统提供的 API 接口各不相同，很难能保证写一份代码，随处编译。

即使有 posix 接口、libc 库接口等保证一定程度上调用的 API 是跨平台的，但实际上不同平台对这些接口的支持力度都各不相同，就拿 `strncasecmp` 接口来说，gcc 和 clang 编译器通常都会提供，但是对于 msvc 编译器，是没有这个接口，只提供了等价的 `_strnicmp` 接口。

这个时候，如果我们要支持跨平台编译和运行，那么就需要在代码中判断当前的编译器和工具链是否提供了这个接口，如果有就调用，没有就用其它解决方案，例如。

```c
#ifdef _MSC_VER
    _strnicmp(s1, s2, n);
#else
    strncasecmp(s1, s2, n);
#endif
```

但是这样还是不可靠的，因为我们不能保证 msvc 的所有版本都提供了 `_strnicmp` 接口，有可能老版本里面也没这个接口，同时也不能保证所有 gcc 和 clang 的工具链都提供了 `strncasecmp`，有些嵌入式系统的交叉编译工具链为了精简，可能会去掉部分 libc 接口也是有可能的。

因此，单纯的判断编译器是不行的，我们应该直接检测当前使用的编译工具链里面有没有这个接口，如果有就用，没有就不去用它。但是，这种检测在代码中通过宏是完成不了的，这时候就需要构建工具去提供支持才行。

例如，我们在编译的时候，构建工具先自动检测当前的编译工具链里面是否能够正常使用 `_strnicmp` 接口，如果检测通过，那么可以自动给编译器传递一个宏定义，例如 `HAVE_STRNICMP`，这样就可以在代码中进行更准确的判断处理。

```c
#if defined(HAVE_STRNICMP)
    _strnicmp(s1, s2, n);
#elif defined(HAVE_STRNCASECMP)
    strncasecmp(s1, s2, n);
#else
#    error "not supported"
#endif
```

上述的代码判断逻辑已经相对比较可靠，也已经能很好的处理跨平台了。如果当前编译器没提供对应接口，我们要么直接报错、提示不支持，要么可以自己实现这个接口把它支持上。

因此，为了让大家更方便的处理跨平台，xmake 内置提供了各种便于检测的配置接口，方便用户定制化配置检测编译器特性、头文件、接口、库、类型等支持力度，下面我们会逐一详细讲解如何去配置 xmake.lua 来实现跨平台特性检测。

#### 配置库头文件检测

先从最简单的例子开始，假设我们现在需要使用 pthread 库，但不确定当前环境和工具链是否已经提供了 `pthread.h` 头文件，那么可以通过配置检测下这个头文件是否在。

在配置检测前，执行如下命令创建一个空工程。

```bash
cd Code
xmake create check
```

然后修改其中的 xmake.lua 文件配置，通过 `includes` 接口引入 xmake 内置提供的 `check_cincludes.lua` 辅助接口文件，引入后就可以使用里面提供的 `check_cincludes` 辅助接口去检测头文件是否存在。

```lua
includes("check_cincludes.lua")

target("check")
    set_kind("binary")
    add_files("src/*.cpp")
    check_cincludes("HAS_PTHREAD_H", "pthread.h")
```

上面的配置中，我们通过 check_cincludes 配置了 `pthread.h` 头文件的检测，如果检测通过，那么编译时，`-DHAS_PTHREAD_H` 宏定义会自动传递给 gcc 编译器。这样就可以在代码中通过这个宏定义判断出当前的编译环境是否存在这个头文件了。

修改 `src/main.cpp` 中的代码，加上对 `HAS_PTHREAD_H` 的判断检测，例如。

```c++
#ifdef HAS_PTHREAD_H
#   include <pthread.h>
#else
#   error "pthread.h not found!"
#endif

int main(int argc, char** argv)
{
    return 0;
}
```

这里仅仅判断 `pthread.h` 是否存在，不存在就直接报错。当然如果要真正的支持跨平台，我们应该处理的更完善些，比如判断 `pthread.h` 不存在，那就继续尝试其它的线程接口，比如 Windows 下的线程 API 等等。

一切准备就绪后，执行 `xmake` 去编译这个工程，xmake 会在编译的前期自动检测 `pthread.h` 是否存在，如果存在，就会看到下图红线位置的检测结果，并且正常通过编译。

![1](实验11CCpp库接口和编译器特性检测.assets/066c69d7e77884a8d6afea2fff22f0e0-0)

只要编译没报错，就说明检测通过并且 `HAS_PTHREAD_H` 已经被定义上了，我们也可以再执行 `xmake -rv` 重新编译查看完整的编译输出，继续确认下 `-DHAS_PTHREAD_H` 是否真的被传递到了 gcc 编译器，如图所示。

![2](实验11CCpp库接口和编译器特性检测.assets/7e93c1b0ddee79d82822177ffeb2a34c-0)

#### 配置自动生成 config.h

也可以将检测结果永久保存下来，并将对应检测后的宏定义放置在特定的 config.h 头文件中，这便于我们将自己项目的库头文件导出给其它独立工程使用，只需要附带提供这个 config.h 就行了。

为了支持自动生成 config.h 文件，只需要修改 xmake.lua，例如。

```lua
includes("check_cincludes.lua")

target("check")
    set_kind("binary")
    add_files("src/*.cpp")
    add_configfiles("config.h.in")
    set_configdir("$(buildir)")
    add_includedirs("$(buildir)")
    configvar_check_cincludes("HAS_PTHREAD_H", "pthread.h")
```

上面的配置相比之前稍微复杂些，但还是很好理解的，其中 `add_configfiles` 用于设置 `config.h.in` 作为配置模板文件，一会检测完成后，会把结果生成到对应的 `config.h` 中，具体生成到哪里，由 `set_configdir` 指定，这里我们配置将其生成到 `build` 目录下。

而 `configvar_check_cincludes` 也就是跟之前的 `check_cincludes` 检测接口类似，差别在于，前者检测完会把结果存储到 `config.h`，而后者检测完仅仅只会直接传递给编译器，不会永久保存。

配置完成后，我们继续在当前项目根目录下创建并编辑 `config.h.in` 文件，这是一个模板文件，xmake 会基于这个模板文件提供的内容来生成实际的 `config.h`。

`config.h.in` 模板文件的内容如下。

```c
${define HAS_PTHREAD_H}
```

当前只需要加一行代码，定义了一个模板变量，如果 xmake 检测 `pthread.h` 通过，那么这行会被自动展开成下面的代码，然后存储到 `config.h` 文件中去。

```c
#define HAS_PTHREAD_H 1
```

如果检测没通过，那么这行会被自动展开为下面的定义，也就是不定义 `HAS_PTHREAD_H`。

```c
/*#define HAS_PTHREAD_H 0*/
```

另外，除了配置 xmake.lua 和 config.h.in 文件，我们还需要改下 `src/main.cpp` 将自动生成的 `config.h` 引入进来获取检测后的宏定义结果，例如。

```c++
#include "config.h"

#ifdef HAS_PTHREAD_H
#   include <pthread.h>
#else
#   error "pthread.h not found!"
#endif

int main(int argc, char** argv)
{
    return 0;
}
```

相比之前的代码，这里还多了一行 `#include "config.h"` 来引入 `config.h` 头文件获取检测结果，也就是 `HAS_PTHREAD_H`。

到这里为止，我们整个配置过程就算完成了。现在执行 `xmake` 命令，确认是否能够通过编译，如果编译通过，说明检测到了 pthread.h 文件，并且获取到了 `HAS_PTHREAD_H` 定义，因此也就不会报 `pthread.h not found!` 的编译错误。

编译结果如下图。

![3](实验11CCpp库接口和编译器特性检测.assets/7188588f22b0266c32400e7e8b4eb477-0)

上面红线部分提示检测正常通过，并且还自动生成了 config.h 文件，至于它的输出位置，就是之前 xmake.lua 配置中指定的 build 目录，如下图所示。

![4](实验11CCpp库接口和编译器特性检测.assets/0db8d8c0a2616978363b1e1f2b3c3203-0)

执行 `cat ./build/config.h` 命令，查看实际生成到 config.h 文件里面的内容，说明 `HAS_PTHREAD_H` 确实被定义进去了。

![5](实验11CCpp库接口和编译器特性检测.assets/a1ccf23930a7c68b8df97101ce2dab6e-0)

注：xmake 会严格按照配置在 config.h.in 里面的内容格式生成和替换模板变量，不会自己额外添加不存在的配置，关于这个配置模板，我们还可以参考下 tbox 库里面的 [tbox.config.h.in](https://github.com/tboox/tbox/blob/master/src/tbox/tbox.config.h.in) 文件，来更直观地了解。

#### 配置库接口检测

虽然刚刚完成了头文件的自动检测，但是即使检测通过，也不代表这个头文件里面有我们需要的接口，毕竟随着平台和编译环境的差异，即使相同的头文件，也会随着一些宏定义的不同，接口也会随之变动。

为了更好地处理跨平台，我们应该检测指定头文件里面是否确实存在需要的接口 API 才行，而 xmake 也提供了接口检测相关的辅助接口。继续修改 xmake.lua 文件，在之前的配置基础上，新引入一个 `check_cfuncs` 的辅助接口，去尝试从 `signal.h` 和 `setjmp.h` 头文件中检测是否存在 setjmp 接口。

```lua
includes("check_cfuncs.lua")
includes("check_cincludes.lua")

target("check")
    set_kind("binary")
    add_files("src/*.cpp")
    add_configfiles("config.h.in")
    set_configdir("$(buildir)")
    add_includedirs("$(buildir)")
    configvar_check_cincludes("HAS_PTHREAD_H", "pthread.h")
    configvar_check_cfuncs("HAS_SETJMP", "setjmp", {includes = {"signal.h", "setjmp.h"}})
```

这里为了检测 C 函数接口，需要通过 `{includes = {}}` 指定从哪些头文件中去检测（还可以传递 `defines`、`links` 等参数来扩展检测），而第二个参数就是指定需要检测的函数名 `setjmp`。

除了指定函数名，我们还可以写传递参数来扩展函数检测，具体看自己的检测需求，因为有时候定义在头文件的接口其实不是函数，是宏定义过的，这时单纯的指定函数名，是检测不过的，我们通过传递参数调用来通过检测。

具体对于设置 `check_cfuncs` 的函数名描述规则如下。

| 函数描述                                        | 说明          |
| ----------------------------------------------- | ------------- |
| `sigsetjmp`                                     | 纯函数名      |
| `sigsetjmp((void*)0, 0)`                        | 函数调用      |
| `sigsetjmp{int a = 0; sigsetjmp((void*)a, a);}` | 函数名 + {}块 |

接下来我们在 `config.h.in` 文件中加入这个宏开关。

```c
${define HAS_PTHREAD_H}
${define HAS_SETJMP}
```

在 `src/main.cpp` 文件中，我们也加上对应的宏开关来判断是否检测到，如果检测到就引入 setjmp 接口进来。

```c++
#include "config.h"

#ifdef HAS_PTHREAD_H
#   include <pthread.h>
#else
#   error "pthread.h not found!"
#endif

#ifdef HAS_SETJMP
#   include <signal.h>
#   include <setjmp.h>
#else
#   error "setjmp() not found!"
#endif

int main(int argc, char** argv)
{
    return 0;
}
```

然后执行 `xmake` 命令编译，查看是否能够检测通过、完成编译，因为在 Linux 下通常都是存在这个 setjmp 接口的，因此通常是可以编译通过的，如下图。

![6](实验11CCpp库接口和编译器特性检测.assets/d530f9ac5f1324b2ea099ceeb9797afd-0)

编译通过后，我们再来看下 `build/config.h` 里面的内容，这时 `${define HAS_SETJMP}` 模板变量已经自动变成了 `#define HAS_SETJMP 1`，说明检测顺利通过。

![7](实验11CCpp库接口和编译器特性检测.assets/b4088d819d6c7a3a5d4007a66e705d80-0)



## 配置编译器特性检测

由于不同的编译器对 C++ 标准的支持力度都是不同的，有些编译器可能并不支持 C++ 的一些高版本标准中的特性，这时我们就可以通过 xmake 提供的编译器特性检测接口去检测它们。

这里以 C++11 的 constexpr 为例，在编译代码时，检测判断是否支持 constexpr，如果不支持，我们就不使用这个关键字。

为了简化实验步骤，这里重新修改 xmake.lua 配置，去掉对 config.h 的使用，直接通过 `check_features` 接口去完成 C++ 特性的检测，例如。

```lua
includes("check_features.lua")

target("check")
    set_kind("binary")
    add_files("src/*.cpp")
    check_features("HAS_CONSTEXPR", "cxx_constexpr")
```

其中 `cxx_constexpr` 就是指 C++ 的 constexpr 特性名称，关于完整的 C++ 特性名称列表，我们可以到 [C++ 特性列表名称](https://xmake.io/#/zh-cn/manual/extension_modules?id=compilerfeatures) 查看。

然后修改 `src/main.cpp` 中的代码，改成如下对 constexpr 的使用。

```c++
#ifdef HAS_CONSTEXPR
constexpr int a = 1;
constexpr int c = a * 2 + 1;
#else
#   error "constexpr not supported!"
#endif

int main(int argc, char** argv)
{
    return 0;
}
```

这个时候，如果我们执行 `xmake` 编译后，会发现 constexpr 的特性并没有检测通过，编译也是失败的，如下图。

![8](实验11CCpp库接口和编译器特性检测.assets/6a7103828e959f4360c585ac3943ca1a-0)

那是因为这个关键字仅仅只在 c++11 标准之后才生效，因此需要对 target 启用 c++11 标准，并且对于 `check_features` 检测接口，也需要传递 c++11 的语言标准给它才能通过检测。

因此继续修改完善 xmake.lua 配置，修改如下。

```lua
includes("check_features.lua")

target("check")
    set_kind("binary")
    add_files("src/*.cpp")
    set_languages("c++11")
    check_features("HAS_CONSTEXPR", "cxx_constexpr", {languages = "c++11"})
```

设置上 c++11 语言标准后，再来执行 `xmake` 命令就能顺利通过检测和编译了。

![9](实验11CCpp库接口和编译器特性检测.assets/735bed4af388bfcab65b23b24c2f446a-0)

#### 配置自定义 C/C++ 代码片段检测

虽然，xmake 提供了很多内置的辅助检测接口，可以方便的检测头文件、库接口、类型定义、编译器特性等等，但是对于一些比较复杂的检测，单纯靠这些是无法满足的，这时我们还可以通过 xmake 提供的通用检测接口，也就是 C/C++ 代码片段检测，来实现任意的用户检测需求。

可以在检测接口中，传入一小段 C/C++ 代码片段，来实现我们所希望的检测逻辑，其中 `check_csnippets` 用于检测 C 代码片段，而 `check_cxxsnippets` 用于检测 C++ 代码片段。

继续修改之前的 xmake.lua 文件，加上对这两个接口的配置使用，例如。

```lua
includes("check_csnippets.lua")
includes("check_cxxsnippets.lua")

target("check")
    set_kind("binary")
    add_files("src/*.cpp")
    set_languages("c++11")
    check_csnippets("HAS_STATIC_ASSERT", "_Static_assert(1, \"\");")
    check_cxxsnippets("HAS_CONSTEXPR", "constexpr int a = 1;", {languages = "c++11"})
```

通过上面的配置，我们看到其中加了两个代码片段，一个是 C 代码片段，用来检测能否使用 `_Static_assert` 来实现静态断言，而另外一个是 C++ 代码片段，用来实现跟之前的编译器特性检测配置一样的效果，也就是检测 constexpr 是否可用。

但是跟之前不同的是，我们这里是直接在配置中传入了完整的 C++ 代码片段，在里面去使用 constexpr，然后 xmake 会尝试调用 gcc 等编译器去编译它，如果编译通过，就会定义 `HAS_CONSTEXPR` 宏，也就是通过了检测。

同时也需要修改 `src/main.cpp` 文件，加上对 `HAS_STATIC_ASSERT` 和 `HAS_CONSTEXPR` 的宏定义检测。

```c++
#ifdef HAS_CONSTEXPR
constexpr int a = 1;
constexpr int c = a * 2 + 1;
#else
#   error "constexpr not supported!"
#endif

#ifndef HAS_STATIC_ASSERT
#   error "_Static_assert not supported!"
#endif

int main(int argc, char** argv)
{
    return 0;
}
```

然后执行 `xmake` 编译，如果一切顺利，就会看到下图编译通过的输出信息。

![10](实验11CCpp库接口和编译器特性检测.assets/da380ae16193c8b0fe07f94c0517a002-0)



## 实验总结

在本节实验中，我们了解了怎样才能实现跨平台开发和编译，并且主要学习了如何去使用 xmake 提供的辅助接口来配置检测 C/C++ 的头文件、库接口是否存在，以及检测指定的编译器特性和 C/C++ 代码片段是否能够支持并使用。

本实验的参考代码可以使用如下命令下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/2764/code11.zip
```

