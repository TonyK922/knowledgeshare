# CMake常用命令的一些整理

CMake 是什么我就不用再多说什么了，相信大家都有接触才会看一篇文章。对于不太熟悉的开发人员可以把这篇文章当个查找手册。

## 1. CMake语法    

### 1.1指定cmake的最小版本

```
cmake_minimum_required(version 版本号)
```

例如：

```
cmake_minimum_required(version 2.8)
```



### 1.2 定义工程名称

```
#定义工程名称 
project(项目名称)
```

例如：

```
project(MyTest)
```

### 1.3 显示定义变量

```
set(var [value])
```

例如：

```
# 第一种用法，生成代码文件列表
#先直接设置SRC_LIST的值
set(SRC_LIST add.h add.cpp)
#然后再在SRC_LIST中追加main.cpp 
set(SRC_LIST ${SRC_LIST} main.cpp)

# 第二种用法，设置库生成目录或者可执行文件生成目录
set( LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib/linux) 
set( EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
```



### 1.4 设置编译类型

```
# 编译静态库
add_library(库名称 STATIC 代码文件名称) 

# 编译动态库
add_library(库名称 SHARED 代码文件名称) 

# 编译可执行程序
add_executable(可执行程序名 代码文件名称)
```

例如：

```
# 编译静态库
add_library(add STATIC add.h add.cpp)
add_library(add STATIC ${ADD_SRC} ${ADD_HDR})

# 编译动态库
add_library(add  SHARED add.h add.cpp) 
add_library(add SHARED  ${ADD_SRC} ${ADD_HDR})

# 编译可执行程序
add_executable(main add.h add.cpp mai.cpp)
add_executable(main ${MAIN_SRC} ${MAIN_HDR})
```

### 1.5 指定静态库或者动态库编译输出目录

例如将当前编译的静态库或者动态库输出到当前项目文件夹lib子目录下：

```
set(LIBRARY_OUTPUT_PATH  ${PROJECT_SOURCE_DIR}/lib)
```

### 1.6 指定可执行程序编译输出目录

例如将当前可执行程序输出到当前项目文件夹的bin子目录下：

```
#设定可执行二进制文件的目录
set( EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
```

### 1.7 设置链接库搜索目录

例如将链接库搜索目录设置为当前项目文件夹下lib/linux文件夹：

```
link_directories( ${PROJECT_SOURCE_DIR}/lib/linux)
```

### 1.8 设置包含目录

例如将包含目录设置为当前项目文件夹下include文件夹：

```
include_directories(${PROJECT_SOURCE_DIR}/include)
```



### 1.9 设置宏定义

```
#预定义宏
add_definitions(-D宏名称)
```

例如：

```
add_definitions(-DWINDOWS)
add_definitions(-DLINUX)
```

### 1.10 链接静态库

```
link_libraries(
    静态库1
    静态库2
    静态库3
    ...
)
```

注意，link_libraries中的静态库为全路径，常与1.7 link_directories 搭配使用，例如：

lib1.a lib2.a在目录${PROJECT_SOURCE_DIR}/lib/linux下，则先设置链接目录，再链接相应的库：

```
#设置链接目录
link_directories( ${PROJECT_SOURCE_DIR}/lib/linux)
link_libraries(
        lib1.a
        lib2.a
)
```



### 1.11 链接动态库

```
target_link_libraries(所需生成的文件名称 所需链接的动态库名称)
```



例如：

```
target_link_libraries(main dl)
```



### **1.12 **link_libraries 和 target_link_libraries 区别

在cmake语法中，link_libraries和target_link_libraries是很重要的两个链接库的方式，虽然写法上很相似，但是功能上有很大区别：

```
(1) link_libraries用在add_executable之前，target_link_libraries用在add_executable之后
(2) link_libraries用来链接静态库，target_link_libraries用来链接导入库，即按照header file + .lib + .dll方式隐式调用动态库的.lib库
```



### 1.13 file语法

#### 1.13.1 将文件夹所有的类型的文件添加到文件列表

例如将当前文件夹下所有.cpp文件的文件名加入到MAIN_SRC中，将当前文件夹下所有.h加入到MAIN_HDR中。

```
file(GLOB MAIN_SRC ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
file(GLOB MAIN_HDR ${CMAKE_CURRENT_SOURCE_DIR}/*.h)
```



例如将当前文件夹子目录src文件夹下所有.cpp文件的文件名加入到MAIN_SRC中，将当前文件夹子目录src文件夹下所有.h加入到MAIN_HDR中。

```
file(GLOB MAIN_SRC ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
file(GLOB MAIN_HDR ${CMAKE_CURRENT_SOURCE_DIR}/src/*.h)
```



#### 1.13.2 递归搜索该文件夹，将文件夹下（包含子目录）符合类型的文件添加到文件列表

例如将当前文件夹下（包括子目录下）所有.cpp文件的文件名加入到MAIN_SRC中，所有.h加入到MAIN_HDR中：

```
file(GLOB_RECURSE MAIN_SRC ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
file(GLOB_RECURSE MAIN_HDR ${CMAKE_CURRENT_SOURCE_DIR}/*.h)
```



### 1.14 List操作

常见的List操作包括：

```
list(LENGTH <list> <output variable>) 
list(GET <list> <element index> [<element index> ...]
   <output variable>)
list(APPEND <list> [<element> ...])
list(FIND <list> <value> <output variable>)
list(INSERT <list> <element_index> <element> [<element> ...])
list(REMOVE_ITEM <list> <value> [<value> ...])
list(REMOVE_AT <list> <index> [<index> ...])
list(REMOVE_DUPLICATES <list>)
list(REVERSE <list>)
list(SORT <list>)
```



#### 1.14.1 List移除指定项

例如从MAIN_SRC移除指定项：

```
list(REMOVE_ITEM MAIN_SRC ${CMAKE_CURRENT_SOURCE_DIR}/add.cpp)
```



#### 1.14.2 将两个List链接起来

```
# 搜索当前目录
file(GLOB  MAIN_SRC ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
file(GLOB  MAIN_HDR ${CMAKE_CURRENT_SOURCE_DIR}/*.h)

# 递归搜索当前目录下src子目录
file(GLOB_RECURSE MAIN_SRC_ELSE  ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
file(GLOB_RECURSE MAIN_HDR_ELSE  ${CMAKE_CURRENT_SOURCE_DIR}/src/*.h)

# 将MAIN_SRC_ELSE中的值添加到MAIN_SRC 
# 将MAIN_HDR_ELSE中的值添加到MAIN_HDR 
list(APPEND MAIN_SRC ${MAIN_SRC_ELSE})
list(APPEND MAIN_HDR ${MAIN_HDR_ELSE})
```



### 1.15 添加子文件夹

例如：

```
add_subdirectory(src)
```



该语句会在执行完当前文件夹CMakeLists.txt之后执行src子目录下的CMakeLists.txt



### 1.16 message输出消息机制

输出正常：

```
message(STATUS "Enter cmake ${CMAKE_CURRENT_LIST_DIR}")
```



输出警告：

```
message(WARNING "Enter cmake ${CMAKE_CURRENT_LIST_DIR}")
```



输出错误：

```
message(FATAL_ERROR "Enter cmake ${CMAKE_CURRENT_LIST_DIR}")
```



### 1.17 安装

install 指令用于定义安装规则，安装的内容包括二进制可执行文件、动态库、静态库以及文件、目录、脚本等。

#### 1.17.1 目标文件安装

例如：

```
install(TARGETS util
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib)
```



ARCHIVE指静态库，LIBRARY指动态库，RUNTIME指可执行目标二进制，上述示例的意思是：

如果目标util是可执行二进制目标，则安装到${CMAKE_INSTALL_PREFIX}/bin目录 如果目标util是静态库，则安装到安装到${CMAKE_INSTALL_PREFIX}/lib目录 如果目标util是动态库，则安装到安装到${CMAKE_INSTALL_PREFIX}/lib目录。



#### 1.17.2 文件夹安装

```
install(DIRECTORY include/ DESTINATION include/util)
```



这个语句的意思是将include/目录安装到include/util目录。



### 1.18 设置编译选项

设置编译选项可以通过add_compile_options命令，也可以通过set命令修改CMAKE_CXX_FLAGS或CMAKE_C_FLAGS。方式1：

```
set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -march=native -O3 -frtti -fpermissive -fexceptions -pthread")
```



方式2：

```
add_compile_options(-march=native -O3 -fexceptions -pthread -fPIC)
```



这两种方式的区别在于：

```
add_compile_options命令添加的编译选项是针对所有编译器的(包括c和c++编译器)，而set命令设置CMAKE_C_FLAGS或CMAKE_CXX_FLAGS变量则是分别只针对c和c++编译器的。
```



### 1.19 预定义变量

#### 1.19.1 基本变量

- **PROJECT_SOURCE_DIR**

- 我们使用cmake命令后紧跟的目录，一般是工程的根目录；

- **PROJECT_BINARY_DIR**

  执行cmake命令的目录,通常是${PROJECT_SOURCE_DIR}/build；

- **CMAKE_INCLUDE_PATH**

  系统环境变量,非cmake变量；

- **CMAKE_LIBRARY_PATH**

  系统环境变量,非cmake变量；

- **CMAKE_CURRENT_SOURCE_DIR**

  当前处理的CMakeLists.txt所在的路径；

- **CMAKE_CURRENT_BINARY_DIR**

  target编译目录（使用ADD_SURDIRECTORY(src bin)可以更改此变量的值 ，SET(EXECUTABLE_OUTPUT_PATH <新路径>)并不会对此变量有影响,只是改变了最终目标文件的存储路径）；

- **CMAKE_CURRENT_LIST_FILE**

  输出调用这个变量的CMakeLists.txt的完整路径；

- **CMAKE_CURRENT_LIST_LINE**

  输出这个变量所在的行；

- **CMAKE_MODULE_PATH**

  定义自己的cmake模块所在的路径（这个变量用于定义自己的cmake模块所在的路径，如果你的工程比较复杂，有可能自己编写一些cmake模块，比如SET(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake),然后可以用INCLUDE命令来调用自己的模块）；

- **EXECUTABLE_OUTPUT_PATH**

  重新定义目标二进制可执行文件的存放位置；

- **LIBRARY_OUTPUT_PATH**

  重新定义目标链接库文件的存放位置；

- **PROJECT_NAME**

  返回通过PROJECT指令定义的项目名称；

- **CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS**

  用来控制IF ELSE语句的书写方式；

#### 1.19.2 操作系统变量

- **CMAKE_MAJOR_VERSION**

  cmake主版本号,如3.4.1中的3；

- **CMAKE_MINOR_VERSION**

  cmake次版本号,如3.4.1中的4；

- **CMAKE_PATCH_VERSION**

  cmake补丁等级,如3.4.1中的1；

- **CMAKE_SYSTEM**

  操作系统名称，包括版本名，如Linux-2.6.22；

- **CAMKE_SYSTEM_NAME**

  操作系统名称，不包括版本名，如Linux；

- **CMAKE_SYSTEM_VERSION**

  操作系统版本号,如2.6.22；

- **CMAKE_SYSTEM_PROCESSOR**

  电脑处理器名称，如i686；

- **UNIX**

  在所有的类UNIX平台为TRUE,包括OS X和cygwin，Linux/Unix操作系统；

- **WIN32**

  在所有的win32平台为TRUE,包括cygwin，Windows操作系统；

- **APPLE**

  苹果操作系统；

例如操作系统判断方式一：

```
if(WIN32)
    message(STATUS “This operating system is Windows.”)
elseif(UNIX)
    message(STATUS “This operating system is Linux.”)
elseif(APPLE)
    message(STATUS “This operating system is APPLE.”)
endif(WIN32)
```



操作系统判断方式二：

```
if (CMAKE_SYSTEM_NAME MATCHES "Linux")
    message(STATUS "current platform: Linux ")
elseif (CMAKE_SYSTEM_NAME MATCHES "Windows")
    message(STATUS "current platform: Windows")
elseif (CMAKE_SYSTEM_NAME MATCHES "FreeBSD")
    message(STATUS "current platform: FreeBSD")
else ()
    message(STATUS "other platform: ${CMAKE_SYSTEM_NAME}")
endif (CMAKE_SYSTEM_NAME MATCHES "Linux")
```

1.19.3 开关选项

- **BUILD_SHARED_LIBS**

  控制默认的库编译方式。如果未进行设置,使用ADD_LIBRARY时又没有指定库类型,默认编译生成的库都是静态库；

- **CMAKE_C_FLAGS**

  设置C编译选项，也可以通过指令ADD_DEFINITIONS()添加；

- **CMAKE_CXX_FLAGS**

  设置C++编译选项，也可以通过指令ADD_DEFINITIONS()添加；

- **CMAKE_C_COMPILER**

  指定C编译器；

- **CMAKE_CXX_COMPILER**

  指定C++编译器；

- CMAKE_BUILD_TYPE==::build 类型(Debug, Release, …)

  CMAKE_BUILD_TYPE=Debug

#### 1.19.4 环境变量

设置环境变量：

```
set(env{name} value)
```

调用环境变量：

```
$env{name}
```

例如：

```
message(STATUS "$env{name}")
```

#### 1.19.5 CMAKE_INCLUDE_CURRENT_DIR

自动添加CMAKE_CURRENT_BINARY_DIR和CMAKE_CURRENT_SOURCE_DIR到当前处理 的CMakeLists.txt。相当于在每个CMakeLists.txt加入：

```
include_directories(${CMAKE_CURRENT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR})
```

### 1.20 条件判断

#### 1.20.1 逻辑判断和比较

- **if (expression)**：expression 不为空时为真，false的值包括（0,N,NO,OFF,FALSE,NOTFOUND）；
- **if (not exp)**：与上面相反；
- **if (var1 AND var2)**：如果两个变量都为真时为真；
- **if (var1 OR var2)**：如果两个变量有一个为真时为真；
- **if (COMMAND cmd)**：如果 cmd 确实是命令并可调用为真；
- **if (EXISTS dir) if (EXISTS file)**：如果目录或文件存在为真；
- **if (file1 IS_NEWER_THAN file2)**：当 file1 比 file2 新，或 file1/file2 中有一个不存在时为真，文件名需使用全路径；
- **if (IS_DIRECTORY dir)**：当 dir 是目录时为真；
- **if (DEFINED var)**：如果变量被定义为真；
- **if (var MATCHES regex)**：给定的变量或者字符串能够匹配正则表达式regex 时为真，此处 var 可以用 var 名，也可以用 ${var}；
- **if (string MATCHES regex)**：给定的字符串能够匹配正则表达式regex时为真。

#### 1.20.2 数字比较

- **if (variable LESS number)**：

  如果variable小于number时为真；

- **if (string LESS number)**：

  如果string小于number时为真；

- **if (variable GREATER number)**：

  如果variable大于number时为真；

- **if (string GREATER number)**：

  如果string大于number时为真；

- **if (variable EQUAL number)**：

  如果variable等于number时为真；

- **if (string EQUAL number)**：

  如果string等于number时为真。

#### 1.20.3 字母表顺序比较

- **if (variable STRLESS string)**
- **if (string STRLESS string)**
- **if (variable STRGREATER string)**
- **if (string STRGREATER string)**
- **if (variable STREQUAL string)**
- **if (string STREQUAL string)**

### 1.21 循环

#### 1.21.1  foreach

start 表示起始数，stop 表示终止数，step 表示步长

```
foreach(loop_var RANGE start stop [step])
    ...
endforeach(loop_var)
```

#### 1.21.2  while

```
while(condition)
    ...
endwhile()
```

### 1.22 自动检测编译器是否支持C++11

```
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
elseif(COMPILER_SUPPORTS_CXX0X)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else()
    message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()
```

### 1.23 CMake生成VS解决方案将项目放置在设定文件夹下

例如，我们在工程中引用了许多的第三方开源库，这些库的源码与自己所写的代码需要进行区分和隔离，通常情况下会单独开一个third筛选器存储这些第三方库的项目，怎么做？



第一步：

在第三方库的CMakeLists.txt中cmake_minimum_required(VERSION 2.6)中加上set_property(GLOBAL PROPERTY USE_FOLDERS On)

第二步：在生成编译目标的语法之后，如：

```
add_executable(demo demo.cpp) # 生成可执行文件
add_library(common STATIC util.cpp) # 生成静态库
add_library(common SHARED util.cpp) # 生成动态库或共享库
```

加入一句：

```
set_target_properties(${第三方库项目名称} PROPERTIES FOLDER “目标文件夹名称”)
```

## 2. 错误解决方案   

 **2.1 Cannot specify link libraries for target “/…/…/lib/linux/libMyDll.a” which** 

这个问题要将生成执行文件、静态库、动态库的声明：

```
add_executable(demo demo.cpp) # 生成可执行文件
add_library(common STATIC util.cpp) # 生成静态库
add_library(common SHARED util.cpp) # 生成动态库或共享库
```

放在

```
target_link_libraries()
```

之前。

**2.2 警告：检测到时钟错误，您的创建可能是不完整的**

在项目根目录下执行命令：

```
touch *
```

更新所有文件时间。  

