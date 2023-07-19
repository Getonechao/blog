+++

title= "cmake：c/c++的工程构建工具"
description= "cmake模板"
date= 2022-04-19T15:57:41+08:00
author= "chao"
draft= false
image= "" 
math= true
categories= [
    "app"
]

tags=  [
    " project","cmake"
]

+++

# 一、cmake模板

~~~shell
|--CMakeLists.txt
|--extern
|--src
|--|--subsrc1
|--|--|--CMakeLists.txt
|--|--subsrc2
|--|--|--CMakeLists.txt
|--|--main.cc
|--|--CMakeLists.txt
|--test
|--|--CMakeLists.txt
|--vcpkg.json
~~~





根目录的CMakeLists.txt

~~~cmake

cmake_minimum_required(VERSION 3.15)

project(PROJECT_XXX VERSION 0.0.0 )

#C/C++标准
set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 14)


#设置编译器
set (CMAKE_C_COMPILER "/usr/bin/gcc")
set (CMAKE_CXX_COMPILER "/usr/bin/g++")

######### build 变量 #####
set(CMAKE_BUILD_TYPE Debug#[[Release | Debug| RelWithDebInfo |MinSizeRel]])
set(CMAKE_BUILD_PARALLEL_LEVEL 4)#编译处理器数量
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)#clang
set(CMAKE_GENERATOR "Unix Makefiles")#“Ninja”、“Unix Makefiles”、“Visual Studio”
#set(CMAKE_TOOLCHAIN_FILE )
#add_compile_options()#等同CMAKE_CXXFLAGS_RELESE,前者可以对所有的编译器设置，后者只能是C++编译器


#lib&&bin输出目录
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/build/bin)#可执行文件
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/build/lib)#动态库
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/build/lib/static)#静态库


###### sub directory #####
add_subdirectory(src)
#add_subdirectory(external)

########## TEST ##########
if(FALSE)
	enable_testing()
	add_subdirectory(test)
	add_test(NAME test COMMAND ${PROJECT_NAME} -arg1 -arg2)
endif()

########## istanll #######
set(CMAKE_INSTALL_PREFIX ${PROJECT_SOURCE_DIR}/install)
# 1.target  放到 DESTINATION 指定的目录
#install(TARGETS ... RUNTIME DESTINATION bin)#exe
#install(TARGETS ... LIBRARY DESTINATION lib)#*.so
#install(TARGETS ... ARCHIVE DESTINATION lib/static)#*.lib
#install(TARGETS ... PUBLIC_HEADER DESTINATION include)#公共头文件的安装路径
#install(TARGETS ... RESOURCE 	   DESTINATION <dir>)#私有头文件的安装路径
# 2.普通文件 放置放到 DESTINATION 指定的目录,eg:readme.md config.ini
#install(FILES ... DESTINATION etc)
# 3.目录
#install(DIRECTORY ... DESTINATION ...)
# 4.脚本    放置到 DESTINATION 指定的目录,eg:install.sh
#install(PROGRAMS ... DESTINATION ...)
# 5.target集合
#install(TARGETS ...  EXPORT export_name RUNTIME DESTINATION bin)#exe
#install(EXPORT export_name NAMESPACE namespace DESTINATION <dir>)

########## PACK ##########
if(FALSE)
# 安装包名称
set(CPACK_PACKAGE_NAME ${PROJECT_NAME})
# 版本号                       
set(CPACK_PACKAGE_VERSION "1.0.0") 
# 描述信息
set(CPACK_PACKAGE_DESCRIPTION "My awesome application")
# 许可证                                  
set(CPACK_RPM_PACKAGE_LICENSE "Apache 2.0 + Common Clause 1.0")
# vendor                             
set(CPACK_PACKAGE_VENDOR "vesoft")  
# 安装包图标
#set(CPACK_PACKAGE_ICON )

#配置软件包类型和生成器ZIP、TGZ、RPM、NSIS
set(CPACK_GENERATOR ZIP)#二进制包
#set(CPACK_SOURCE_GENERATOR ZIP)#源码包

#安装系统依赖库
include(InstallRequiredSystemLibraries)

#安装安装包时的依赖关系
#set(CPACK_DEBIAN_PACKAGE_DEPENDS "")#Debian自动安装依赖
#set(CPACK_RPM_PACKAGE_REQUIRES "")

#添加脚本和配置
#set(CPACK_PRE_INSTALL_SCRIPTS "${CMAKE_CURRENT_SOURCE_DIR}/pre_install_script.sh")#安装前,目录权限
#set(CPACK_POST_INSTALL_SCRIPTS "${CMAKE_CURRENT_SOURCE_DIR}/post_install_script.sh")#安装后，systemctl自启动脚本

# 设置支持指定安装目录的控制为 ON;设置安装到的目录路径                                   
#set(CPACK_SET_DESTDIR ON)
#set(CPACK_INSTALL_PREFIX )   
include(CPack)
endif()

~~~


LIB
~~~cmake

######### Target LIB #########
aux_source_directory(目录 LIBS_SRC_LISTS)

set(LIBS_NAME )
add_library(${LIBS_NAME } "")#默认是STATIC
target_include_directories(${LIBS_NAME} PRIVATE )
target_sources(${LIBS_NAME} PRIVATE ${LIBS_SRC_LISTS})
#target_link_libraries(${LIBS_NAME} )
target_compile_options(${LIBS_NAME} PRIVATE -Wall
                                          -O3 -std=c++11 )
target_compile_definitions(${LIBS_NAME} PRIVATE
                                          CMAKE_BUILD_TYPE=Release
                                           CMAKE_EXPORT_COMPILE_COMMANDS=ON)

~~~


EXE

~~~cmake
######### Target EXE #########
aux_source_directory(目录 EXE_SRC_LISTS)

add_executable(${PROJECT_NAME} )
#target_include_directories(${PROJECT_NAME} RIVATE )
target_sources(${PROJECT_NAME} PRIVATE  ${EXE_SRC_LISTS})
#target_link_libraries(${PROJECT_NAME} )
target_compile_options(${PROJECT_NAME} RRIVATE -Wall
                                            -O3 -std=c++11 )
target_compile_definitions(${PROJECT_NAME} PRIVATE
                                          CMAKE_BUILD_TYPE=Release
                                           CMAKE_EXPORT_COMPILE_COMMANDS=ON)
~~~
FIND
~~~cmake
######### FIND FILE   #######
#find_package(Eigen3 REQUIRED)
#find_path (<VAR> name1 [path1 path2 ...])
#find_file (<VAR> name1 [path1 path2 ...])
#find_library (<VAR> name1 [path1 path2 ...])

~~~
vcpkg
~~~cmake
                                           
#########  VCPKG #########
set(CMAKE_TOOLCHAIN_FILE $ENV{VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake)
find_package(Boost REQUIRED)
include_directories(${Boost_INCLUDE_DIRS})
target_link_libraries(${PROJECT_NAME} ${Boost_LIBRARIES})
~~~
闭源库

~~~cmake
######### 闭源库 ##########
add_library(${LIBNAME} STATIC IMPORTED)
set_property(TARGET ${LIBNAME} PROPERTY IMPORTED_LOCATION ${CMAKE_CURRENT_LIST_DIR}/extern/{LIBNAME}/lib-vc2019/glfw3.lib)
target_include_directories( ${LIBNAME} INTERFACE ${CMAKE_CURRENT_LIST_DIR}/extern/${LIBNAME}/include)
~~~



test

~~~cmake
           
~~~







# 二、参数设置

> 编译选项
>
> **CMAKE_BUILD_TYPE  Release | Debug| RelWithDebInfo |MinSizeRel**
>
> 



# 三、命令解释

## 3.1 find命令

**find_path：用于找到指定文件或目录路径的命令(安装ini文件)**

```cmake
find_path(<VAR> name1 [path1 path2 ...])

eg：
find_path(STDIO_H_INCLUDE_DIR stdio.h
    /usr/include
    /usr/local/include)
    
   
```

其中，`<VAR>`是用于存储找到的路径的变量名。`name1`是要查找的文件或目录的名称。`path1`，`path2`等是可选的搜索路径。

 find_path命令特别适用于需要在构建过程中动态查找头文件路径的情况

**find_file：用于查找指定文件的路径**

```cmake
find_file(<VAR> name1 [path1 path2 ...])

eg：
find_file(EXAMPLE_FILE example.txt)
执行上述命令后，如果找到了example.txt文件，路径将存储在EXAMPLE_FILE变量中，否则该变量将为空。

find_file(EXAMPLE_FILE example.txt
    /usr/data
    /home/user/data
)
这将在/usr/data和/home/user/data目录下搜索example.txt文件
```

其中，`<VAR>`是一个变量，用于存储找到的文件路径。`name1`是要查找的文件的名称。`path1`，`path2`等是可选的搜索路径。

**find_library：用于查找指定库文件的路径**

```cmake
find_library(<VAR> name1 [path1 path2 ...])
```

其中，`<VAR>`是用于存储找到的库文件路径的变量名。`name1`是要查找的库文件的名称（不包括前缀`lib`和文件扩展名）。

使用案例

~~~cmake
cmake_minimum_required(VERSION 3.12)
project(MyProject)

# 查找名为 mylibrary 的库文件
find_library(MYLIBRARY_LIB mylibrary)

# 如果找到了库文件
if(MYLIBRARY_LIB)
    message("Found mylibrary at: ${MYLIBRARY_LIB}")
    # 添加库文件的路径到链接器
    target_link_libraries(MyExecutable ${MYLIBRARY_LIB})
else()
    message(FATAL_ERROR "mylibrary not found")
endif()
~~~

**find_program：用于查找指定可执行程序的路径**

~~~cmake
find_program(<VAR> name1 [path1 path2 ...])

eg：
find_program(MYPROGRAM_EXECUTABLE myprogram
    /usr/bin
    /usr/local/bin
)
~~~

这将在`/usr/bin`和`/usr/local/bin`目录下搜索`myprogram`可执行程序。

使用案例

~~~cmake
cmake_minimum_required(VERSION 3.12)
project(MyProject)

# 查找名为 myprogram 的可执行程序
find_program(MYPROGRAM_EXECUTABLE myprogram)

# 如果找到了可执行程序
if(MYPROGRAM_EXECUTABLE)
    message("Found myprogram at: ${MYPROGRAM_EXECUTABLE}")
    # 在构建过程中使用可执行程序
    add_custom_target(run_myprogram COMMAND ${MYPROGRAM_EXECUTABLE})
else()
    message(FATAL_ERROR "myprogram not found")
endif()
~~~

在这个案例中，假设我们的项目想要运行名为`myprogram`的可执行程序。

通过`find_program(MYPROGRAM_EXECUTABLE myprogram)`命令，CMake会尝试在系统的默认可执行程序搜索路径中找到名为`myprogram`的可执行程序。

如果成功找到了可执行程序，CMake会将路径存储在变量`MYPROGRAM_EXECUTABLE`中，并通过`add_custom_target`命令定义一个自定义目标，使得我们可以在构建过程中使用该可执行程序。

如果未找到可执行程序，则会输出错误消息并终止构建过程

**find_package: CMake中用于查找和加载第三方库的命令**

使用案例

~~~cmake
cmake_minimum_required(VERSION 3.12)
project(MyProject)

# 查找并加载 OpenCV
find_package(OpenCV 4 REQUIRED)

# 检查是否找到库
if(OpenCV_FOUND)
    message("Found OpenCV version ${OpenCV_VERSION}")
    include_directories(${OpenCV_INCLUDE_DIRS})
    add_executable(MyApp main.cpp)
    target_link_libraries(MyApp ${OpenCV_LIBS})
else()
    message(FATAL_ERROR "OpenCV not found")
endif()

# 添加其他项目配置和构建指令...
~~~

变量

>1. `<PackageName>_FOUND`：
>     表示是否找到了指定的库。它是一个布尔值，如果找到了库，则为 `TRUE`，否则为 `FALSE`。
>2. `<PackageName>_VERSION`：
>     当找到库时，它表示所找到的库的版本号。该值可能是一个字符串或一个列表。
>3. `<PackageName>_INCLUDE_DIRS`：
>     包含着所找到的库的头文件路径的变量。这允许您在项目中包含库的头文件。
>4. `<PackageName>_LIBRARIES`：
>     存储了所找到的库的完整库文件路径的变量。通过这个变量，您可以将所需的库链接到项目的可执行文件或库中。
>
>这些变量的命名约定在不同的 `Find` 模块中可能会有所不同，因此请查阅库的相关文档来获取详细的变量名称和用途。

## 3.2 file 执行与文件和目录相关的操作

它可以用于创建、复制、删除、重命名、读取文件内容等操作。以下是一些 `file()` 命令的常见用法：

1. `file(COPY ...)`：
   将指定的文件或目录复制到指定的目标目录下。该命令可以用于将文件复制到构建目录或安装位置。

   ```
   file(COPY source_file DESTINATION destination_directory)
   ```

   

2. `file(REMOVE ...)`：
   删除指定的文件或目录。

   ```
   file(REMOVE file_path)
   ```

   

3. `file(RENAME ...)`：
   将文件或目录重命名。

   ```
   file(RENAME old_name new_name)
   ```

   

4. `file(READ ...)`：
   读取文件内容到变量中。

   ```
   file(READ file_path variable_name)
   ```

   

5. `file(WRITE ...)`：
   将字符串内容写入文件中。

   ```
   file(WRITE file_path "content")
   ```

   

6. `file(APPEND ...)`：
   将字符串内容追加到文件末尾。

   ```
   file(APPEND file_path "content")
   ```

   

这只是 `file()` 命令的一些常见用法示例，实际上它还有很多其他用法，例如创建目录、查找文件、获取文件属性等。您可以在 CMake 官方文档中查找 `file()` 命令的完整参考以获取更详细的信息和用法示例。

## 3.3 自定义命令

在很多时候，需要在`cmake`中创建一些目标，如`clean`、`copy`等等，这就需要通过`add_custom_target`来指定。同时，`add_custom_command`可以用来完成对`add_custom_target`生成的`target`的补充。

**区别**

在CMake中，"add_custom_command"和"add_custom_target"是两个常用的命令，用于定义自定义编译命令和自定义构建目标。它们之间的区别如下：

1. add_custom_command：
   - add_custom_command用于定义在构建时执行的自定义命令。它可以用来生成文件、生成代码、运行脚本等。add_custom_command通常被用作目标的依赖项。
   - add_custom_command并不会创建真正的构建目标，它仅仅是一个构建过程中执行的命令。因此，默认情况下，add_custom_command不会导致重新构建整个项目。
2. add_custom_target：
   - add_custom_target用于定义一个构建目标，该目标不与实际的文件或输出相关联，而是与一组其他规则相关联。
   - add_custom_target可以用来执行一系列自定义命令或构建步骤。它对于组织和管理构建过程非常有用。
   - add_custom_target通常用作构建系统的入口点，通过依赖其他目标和规则来执行自定义构建任务。

总结来说，add_custom_command用于定义构建过程中的自定义命令，而add_custom_target用于定义自定义构建目标。两者可以结合使用，以实现更复杂的构建逻辑。

**add_custom_target：自定义构建目标**

​~~~cmake
add_custom_target(Name [ALL] [command1 [args1...]]
                  [COMMAND command2 [args2...] ...]
                  …
)
# Name：定义的target的名字
# COMMAND：该target要执行的命令

add_custom_target(
    target_name
    [COMMAND command1 [ARGS] [args1...]]
    [COMMAND command2 [ARGS] [args2...] ...]
    [DEPENDS [depend1...]]
    [WORKING_DIRECTORY dir]
    [COMMENT comment]
    [VERBATIM]
    [USES_TERMINAL]
)

其中，常用的参数和选项包括：

target_name：自定义构建目标的名称。
COMMAND：指定要执行的命令。
DEPENDS：指定自定义目标所依赖的其他目标或文件。
WORKING_DIRECTORY：设置命令的工作目录。
COMMENT：添加对命令或目标的注释。
VERBATIM：保留命令中的转义字符，确保按原样传递给底层的构建工具。
USES_TERMINAL：指示命令是否会使用终端。

通过使用 add_custom_target，可以在构建过程中定义各种自定义的构建目标，例如运行测试、生成文档、执行代码检查等。这些目标可以依赖其他目标或文件，并根据需要执行自定义的命令或操作

add_custom_target 命令不会生成实际的构建产物（如可执行文件或库），它只是定义了一个自定义的构建目标。您可以通过 add_dependencies 命令将这个自定义目标与其他目标关联起来，以确保在构建过程中执行相关的自定义操作。
~~~

使用案例

~~~cmake
add_custom_target(RunTests
    COMMAND run_tests.sh
    DEPENDS test_files
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
    COMMENT "Running tests..."
)

add_dependencies(RunTests MyApp)

在这个示例中，RunTests 是一个自定义目标，通过执行 run_tests.sh 脚本来运行测试。它依赖于 test_files 目标（或文件），并在 ${CMAKE_BINARY_DIR} 目录下执行。通过 COMMENT 可以添加注释，说明正在运行的操作。最后，通过 add_dependencies 将 RunTests 目标与 MyApp 目标关联起来，以确保在构建 RunTests 目标时先构建 MyApp 目标。
~~~

使用教程

1. 创建自定义目标：
   首先，使用 `add_custom_target` 命令创建一个自定义构建目标并指定它的名称。

   ```
   add_custom_target(MyTarget)
   ```

   

   在上述示例中，我们创建了一个名为 `MyTarget` 的自定义目标，这个目标还没有定义任何操作。

2. 添加命令：
   使用 `COMMAND` 选项来添加要执行的命令或操作。

   ```
   add_custom_target(MyTarget
       COMMAND echo "Hello, World!"
   )
   ```

   

   在这个例子中，我们在 `MyTarget` 目标中添加了一个命令，即输出 “Hello, World!”。您可以根据实际需求添加更多的命令或操作。

3. 添加依赖项：
   使用 `DEPENDS` 选项来指定 `MyTarget` 目标所依赖的其他目标或文件。

   ```
   add_custom_target(MyTarget
       COMMAND echo "Hello, World!"
       DEPENDS other_target file.txt
   )
   ```

   

   在上述示例中，我们将 `MyTarget` 目标设置为依赖于 `other_target` 目标和 `file.txt` 文件。这意味着在构建 `MyTarget` 之前，CMake 将确保先构建 `other_target` 目标和检查 `file.txt` 文件的更新。

4. 设置工作目录和注释：
   使用其他选项如 `WORKING_DIRECTORY` 和 `COMMENT` 来设置工作目录和添加注释。

   ```
   add_custom_target(MyTarget
       COMMAND echo "Hello, World!"
       DEPENDS other_target file.txt
       WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
       COMMENT "Running custom commands..."
   )
   ```

   

   在上述示例中，我们设置了 `MyTarget` 的工作目录为 `${CMAKE_BINARY_DIR}`，并添加了一个注释以描述正在执行的操作。

5. 添加到其他目标的依赖项：
   使用 `add_dependencies` 命令将自定义目标与其他目标关联起来。

   ```
   add_executable(MyApp main.cpp)
   add_custom_target(MyTarget
       COMMAND echo "Hello, World!"
       DEPENDS other_target file.txt
       WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
       COMMENT "Running custom commands..."
   )
   
   add_dependencies(MyApp MyTarget)
   ```

   

   在这个例子中，我们将自定义目标 `MyTarget` 添加为可执行文件 `MyApp` 的依赖项。这意味着在构建 `MyApp` 时，CMake 将确保先构建 `MyTarget`。



**add_custom_command：自定义编译命令**

~~~cmake
add_custom_command(OUTPUT output1 [output2 ...]
                   COMMAND command1 [ARGS] [args1...]
                   [COMMAND command2 [ARGS] [args2...] ...]
                   [MAIN_DEPENDENCY depend]
                   [DEPENDS [depends...]]
                   [BYPRODUCTS [files...]]
                   [IMPLICIT_DEPENDS <lang1> depend1
                                    [<lang2> depend2] ...]
                   [WORKING_DIRECTORY dir]
                   [COMMENT comment]
                   [DEPFILE depfile]
                   [JOB_POOL job_pool]
                   [VERBATIM] [APPEND] [USES_TERMINAL]
                   [COMMAND_EXPAND_LISTS]
                   [DEPENDS_EXPLICIT_ONLY])
~~~

其中，常用的参数和选项包括：

- `TARGET`：指定目标（可执行文件或库），表示该自定义命令与构建目标相关联。
- `PRE_BUILD`、`PRE_LINK`、`POST_BUILD`：指定自定义命令在何时执行，分别表示前置构建、前置链接和后置构建。
- `COMMAND`：指定要执行的命令。
- `WORKING_DIRECTORY`：设置命令的工作目录。
- `COMMENT`：添加对命令的注释。
- `MAIN_DEPENDENCY`：指定自定义命令所依赖的主文件。
- `DEPENDS`：指定自定义命令所依赖的其他文件。
- `BYPRODUCTS`：指定由命令生成的副产品文件。
- `IMPLICIT_DEPENDS`：指定命令的隐式依赖关系，这些依赖关系可能是根据源文件的语言推断出来的。
- `USES_TERMINAL`：指示命令是否会使用终端（仅在可执行文件的情况下）。

通过使用 `add_custom_command`，可以在构建过程中执行各种自定义的命令，如生成代码、拷贝文件、运行脚本等。这允许您在构建过程中执行与编译和链接无关的任意操作。

请注意，`add_custom_command` 命令必须与 `add_custom_target` 或 `add_executable` / `add_library` 配合使用，以将其与构建目标关联起来。

使用案例

`add_custom_command` 命令用于在构建过程中执行自定义的命令。下面是一个简单的教程，介绍了如何使用 `add_custom_command`：

1. 添加生成文件的自定义命令：
   首先，使用 `add_custom_command` 命令定义一个用于生成文件的自定义命令。

   ```
   add_custom_command(
       OUTPUT generated_file.txt
       COMMAND generate_file.py
       DEPENDS generate_file.py
   )
   ```

   

   在上述示例中，我们定义了一个自定义命令，它使用 Python 脚本 `generate_file.py` 来生成一个名为 `generated_file.txt` 的文件。`DEPENDS` 参数指定生成文件的依赖项，例如脚本文件本身。

2. 将生成的文件作为构建目标使用：
   可以将生成的文件作为构建目标的一部分使用，例如作为源文件或目标文件。

   ```
   add_executable(MyApp main.cpp generated_file.txt)
   ```

   

   在上述示例中，我们将生成的文件 `generated_file.txt` 作为一个源文件之一与 `main.cpp` 一起使用，构建一个名为 `MyApp` 的可执行文件。

3. 声明生成文件的依赖关系：
   使用 `add_dependencies` 命令将自定义命令与其他目标关联起来。

   ```
   add_custom_command(
       OUTPUT generated_file.txt
       COMMAND generate_file.py
       DEPENDS generate_file.py
   )
   
   add_executable(MyApp main.cpp)
   add_dependencies(MyApp generated_file.txt)
   ```

   

   在这个例子中，我们在自定义命令与目标之间建立了依赖关系。通过 `add_dependencies` 命令，`MyApp` 目标将在构建之前先构建 `generated_file.txt`。

使用 `add_custom_command`，您可以执行各种自定义的命令或操作，并将其与构建目标进行关联。这使得您可以在构建过程中执行额外的操作，例如生成代码、复制文件、预处理资源等。根据自己的项目需求，您可以根据需要实现适合的自定义命令，并根据构建逻辑设置相关的依赖关系。

## 3.4 配置文件

configure_file命令是CMake提供的一个常用命令，用于在构建过程中根据模板文件生成配置文件

# 四、自动化测试

当使用CTest来运行测试时，通常需要按照以下步骤进行配置和执行：

1. 创建测试目录：在项目中创建一个用于存放测试脚本和测试数据的目录。可以将该目录命名为`tests`或者其他合适的名称。

2. 编写测试脚本：在测试目录中创建一个或多个测试脚本文件，用于描述测试用例和测试步骤。测试脚本可以使用CTest的命令和语法来定义测试行为、验证结果等。以下是一个简单的示例：

   ```
   # tests/mytest.cmake
   
   # 定义一个测试用例
   add_test(NAME MyTest COMMAND my_program input_data.txt output_data.txt)
   
   # 验证测试结果
   set_tests_properties(MyTest PROPERTIES PASS_REGULAR_EXPRESSION "Expected output")
   ```

   

3. 创建CTest配置文件：在项目的根目录中创建一个名为`CTestTestfile.cmake`的配置文件，用于配置测试设置、测试套件和测试驱动程序等细节。该文件会被CTest自动加载。以下是一个简单的示例：

   ```
   # CTestTestfile.cmake
   
   # 添加测试目录
   add_subdirectory(tests)
   ```

   

4. 构建项目：使用CMake构建项目，此时确保CTest被启用并已成功集成到项目中。

5. 运行CTest：在项目构建完成后，在构建目录中运行`ctest`命令来执行测试。可以使用`ctest --verbose`来显示详细的测试输出。

   ```
   $ ctest
   ```

   

   或者使用图形界面工具来运行CTest，例如运行`ccmake`或`cmake-gui`命令来查看和运行CTest。

6. 查看测试结果：CTest将测试结果输出到终端或CTest的GUI前端。你将看到每个测试的状态（通过、失败、跳过等），以及相关的错误消息和日志文件。

这只是一个简单的CTest使用示例，你可以根据项目的特定需求和测试要求自定义和扩展CTest的功能。参阅CTest文档以获取更多详细信息和更高级的CTest配置选项。

记住，在编写测试脚本时，应该尽可能涵盖项目的各方面，并验证预期的行为和结果。测试是质量保证过程的重要组成部分，能够提供反馈以确保项目的正确性和可靠性。

# 五、安装

## 5.1 Linux的rpath机制

在 CMake 中，可以通过使用 `CMAKE_INSTALL_RPATH` 或者 `CMAKE_BUILD_RPATH` 属性来设置可执行文件的 rpath。

1. `CMAKE_INSTALL_RPATH`：用于指定在安装过程中可执行文件的 rpath。可执行文件会被安装到目标目录，同时 rpath 会被设置为 `CMAKE_INSTALL_RPATH` 指定的路径。可以通过在 CMakeLists.txt 文件中设置该属性来达到目的。

```
set(CMAKE_INSTALL_RPATH <path>)
```

其中 `<path>` 是要设置的 rpath 的路径。

1. `CMAKE_BUILD_RPATH`：用于指定在构建过程中可执行文件的 rpath。可执行文件在构建过程中会被放置在构建目录，同时 rpath 会被设置为 `CMAKE_BUILD_RPATH` 指定的路径。可以通过在 CMakeLists.txt 文件中设置该属性来达到目的。

```
set(CMAKE_BUILD_RPATH <path>)
```

同样，`<path>` 是要设置的 rpath 的路径。

注意：

- `<path>` 可以是多个路径的列表。可以使用 `;` 分隔路径。
- 在设置 `CMAKE_INSTALL_RPATH` 和 `CMAKE_BUILD_RPATH` 时，可以使用 CMake 的 generator expressions，以便根据不同的配置和平台展开不同的路径。
- `CMAKE_INSTALL_RPATH` 和 `CMAKE_BUILD_RPATH` 可以同时设置，并且它们的优先级会根据具体的情况决定。

设置 `CMAKE_INSTALL_RPATH` 或 `CMAKE_BUILD_RPATH` 后，重新运行 CMake 构建过程，可执行文件的 rpath 会被相应地设置。这样，在运行时可执行文件会使用 rpath 指定的路径来查找动态库，从而确保动态库能够正确加载。

需要注意的是，rpath 机制在不同的操作系统上有所不同，具体的设置和行为可能会有所差异。确保根据目标平台和操作系统的要求进行适当的配置和测试。

## 5.2 `CMAKE_INSTALL_RPATH`的使用案例

以下是一个使用 `CMAKE_INSTALL_RPATH` 的简单示例：

```
cmake_minimum_required(VERSION 3.12)
project(MyApp)

# 设置可执行文件的源文件
set(SOURCES main.cpp)

# 设置生成可执行文件
add_executable(myapp ${SOURCES})

# 设置动态库的搜索路径
set(CMAKE_INSTALL_RPATH "$ORIGIN/lib")

# 安装规则
install(TARGETS myapp
    RUNTIME DESTINATION bin
    DESTINATION "${CMAKE_INSTALL_PREFIX}"
    # 设置 rpath 为 CMAKE_INSTALL_RPATH 变量的值
    INSTALL_RPATH "${CMAKE_INSTALL_RPATH}"
)
```

在这个示例中，假设项目目录结构如下：

```
.
├── CMakeLists.txt
├── main.cpp
└── lib
    └── mylib.so
```

- `main.cpp` 是可执行文件的源文件。
- `lib` 目录下包含一个名为 `mylib.so` 的动态库文件。

在 CMakeLists.txt 中，我们首先定义了可执行文件的源文件，并通过 `add_executable()` 命令添加了一个名为 `myapp` 的可执行目标。

接下来，我们通过 `set()` 命令设置了 `CMAKE_INSTALL_RPATH` 的值为 `$ORIGIN/lib`。这里使用了 `$ORIGIN` 变量，它表示可执行文件所在的目录。

最后，在安装规则中，我们使用 `install()` 命令将可执行文件安装到指定的目录，并通过 `INSTALL_RPATH` 属性将 `CMAKE_INSTALL_RPATH` 的值传递给 rpath。

在构建并运行项目时，可执行文件 `myapp` 将被安装到目标目录（例如安装到 `/usr/local/bin`），同时 rpath 将被设置为 `/usr/local/bin/lib`。这样在运行时，可执行文件就能够找到并加载位于 `lib` 目录下的动态库文件。

请注意，实际的 rpath 设置可能因操作系统、CMake 版本和项目结构而有所不同，上述示例仅为了说明如何使用 `CMAKE_INSTALL_RPATH`。根据具体的需求和情况，可能需要进行适当的调整。

## 5.3 `CMAKE_BUILD_RPATH`的使用案例

以下是一个使用 `CMAKE_BUILD_RPATH` 的简单示例：

```
cmake_minimum_required(VERSION 3.12)
project(MyApp)

# 设置可执行文件的源文件
set(SOURCES main.cpp)

# 设置生成可执行文件
add_executable(myapp ${SOURCES})

# 设置动态库的搜索路径
set(CMAKE_BUILD_RPATH "$ORIGIN/lib")

# 构建规则
set_target_properties(myapp PROPERTIES
    # 设置 rpath 为 CMAKE_BUILD_RPATH 变量的值
    BUILD_RPATH "${CMAKE_BUILD_RPATH}"
)
```

在这个示例中，假设项目目录结构如下：

```
.
├── CMakeLists.txt
├── main.cpp
└── lib
    └── mylib.so
```

- `main.cpp` 是可执行文件的源文件。
- `lib` 目录下包含一个名为 `mylib.so` 的动态库文件。

在 CMakeLists.txt 中，我们首先定义了可执行文件的源文件，并通过 `add_executable()` 命令添加一个名为 `myapp` 的可执行目标。

接下来，我们通过 `set()` 命令设置了 `CMAKE_BUILD_RPATH` 的值为 `$ORIGIN/lib`。这里使用了 `$ORIGIN` 变量，它表示构建目录。

最后，使用 `set_target_properties()` 命令，将 `BUILD_RPATH` 属性设置为 `CMAKE_BUILD_RPATH` 的值。这样，在构建过程中，可执行文件 `myapp` 的 rpath 将被设置为构建目录下的 `lib` 目录。

在构建项目时，生成的可执行文件 `myapp` 将具有指定的 rpath，以便在运行时正确加载位于构建目录下的动态库文件。

请注意，实际的 rpath 设置可能因操作系统、CMake 版本和项目结构而有所不同，上述示例仅为了说明如何使用 `CMAKE_BUILD_RPATH`。根据具体的需求和情况，可能需要进行适当的调整。

# 六、闭源包引用

```
#glfw
add_library(glfw STATIC IMPORTED)
set_property(TARGET glfw PROPERTY IMPORTED_LOCATION ${CMAKE_CURRENT_SOURCE_DIR}/extern/glfw/lib-vc2019/glfw3.lib)
target_include_directories(glfw INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/extern/glfw/include)


#引用
target_link_libraries(main glfw)
```

# 七、vcpkg包管理

## 6.1 安装

[官方链接](https://github.com/microsoft/vcpkg/blob/master/README_zh_CN.md)

1. 打开终端。

2. 克隆 Vcpkg 存储库：

   ```
   git clone https://github.com/microsoft/vcpkg.git
   ```

3. 进入 Vcpkg 目录：

   ```
   cd vcpkg
   ```

4. 运行 `bootstrap-vcpkg.sh` 脚本以初始化和构建 Vcpkg：

   ```
   ./bootstrap-vcpkg.sh
   
   sudo ln -s $HOME/vcpkg/vcpkg /usr/bin
   
   cat "EXPORT VCPKG_ROOT=/home/chao/vcpkg" &>> ~/.bashrc
   ```

5. 运行以下命令将 Vcpkg 安装到系统目录 `/usr/local`：

   ```
   sudo ./vcpkg integrate install
   ```

   输入您的密码以进行身份验证。

6. 现在，Vcpkg 已成功安装到您的 Ubuntu 系统上。

7. 使用 Vcpkg 安装和管理库。

   - 在终端中，使用以下命令安装所需的库：

     ```
     ./vcpkg install <library-name>
     ```

     将 `<library-name>` 替换为您要安装的库的名称。

   - 安装完成后，您可以在代码中使用 Vcpkg 安装的库来进行开发和构建。

请注意，使用 Vcpkg 在 Ubuntu 上安装库可能需要满足一些依赖项和构建工具的要求。在某些情况下，您可能需要在 Ubuntu 上预先安装一些依赖项，以便成功安装和使用特定的库

## 6.2 vcpkg.json

1. 在项目的根目录下创建一个名为 vcpkg.json 的文件。

2. 打开 vcpkg.json 并编辑文件，按照 JSON 格式的语法来定义您的库和其设置。下面是一个示例：

   ```json
   {
     "name": "myproject",
     "version": "0.1",
     "dependencies": [
       {
         "name": "library1",
         "version": "1.2"
       },
       {
         "name": "library2",
         "version": "2.0"
       }
     ]
   }
   ```

   上述示例中，“name” 指定项目名称，“version” 指定项目版本，“dependencies” 下列出了项目所依赖的库。

3. 定义库的依赖项。每个依赖项都需要指定名称 (“name”) 和版本 (“version”)。此外，您还可以指定特定的库功能（如果有）。

4. 保存 vcpkg.json 文件。

5. 在终端中，导航到项目的根目录。

6. 运行以下命令，使用 Vcpkg 安装项目依赖项：

   ```
   vcpkg install
   ```

   Vcpkg 将根据 vcpkg.json 文件中定义的库和版本信息，自动下载、安装和构建所需的库。

7. 安装完成后，您可以在项目中使用已安装的库进行开发和构建。

vcpkg.json 是一个方便的方法，可以在项目级别上配置 Vcpkg。通过使用该文件，可以轻松地与其他人共享项目依赖项和配置，并确保每个人都能够使用相同的库版本。

## 6.3 多包管理器共存

~~~cmake
if(USE_VCPKG)
    find_package(<VCPKG_PACKAGE> REQUIRED)
    include_directories(${<VCPKG_PACKAGE>_INCLUDE_DIRS})
    target_link_libraries(MyProject ${<VCPKG_PACKAGE>_LIBRARIES})
else()
    find_package(<APT_PACKAGE> REQUIRED)
    include_directories(${<APT_PACKAGE>_INCLUDE_DIRS})
    target_link_libraries(MyProject ${<APT_PACKAGE>_LIBRARIES})
endif()
~~~

