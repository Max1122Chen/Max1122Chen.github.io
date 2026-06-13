# CMake

[CMake 保姆级教程（上） | 爱编程的大丙](https://subingwen.cn/cmake/CMake-primer/#1-CMake概述)

## C/C++程序构建流程

tool chain -> 预处理 -> 编译 -> 汇编 -> 链接 -> 打包



## makefile

通过各种指令指导构建各种文件组成的复杂系统



## CMake的工作方式

### 常用命令

编写CMakeLists.txt指导生成makefile，makefile再指导构建具体的项目，实现大型项目的自动化构建



简单的CMake命令

```cmake
# 注释

# 最低版本要求（可写可不写）
cmake_minimum_required(VERSION x.x.x)

# 定义项目的名称，还可以增加更多的信息
project(<projectName>)

# 定义工程会生成一个可执行程序
add_executable(exe_name src_files_list)
```



```bash
# 使用cmake产生makefile
cmake CMakeLists.txt_Path
```



```cmake
# 设置变量
set(VARNAME [value])

# 常用宏变量设置
set(CMAKE_CXX_STANDARD XX)
set(EXECUTABLE_OUTPUT_PATH path)

```



搜索文件

```cmake
aux_source_directory(path varName)

file(GLOB/GLOB_RECURSIVE varName pathAndFilePostfix)
```



添加头文件目录

```cmake
include_directories()
```



### CMake构建与使用库文件

#### 构建库

```cmake
# 制作静态库需要使用STATIC，动态库使用SHARED
add_library(<libName> STATIC/SHARED <src_files>)
```

打包好库文件之后还要给别人头文件才行



```cmake
# 同时适用于静态库和动态库的构建
set(LIBRARY_OUTPUT_PATH path)
```

#### 链接库

```cmake
# 链接静态库
link_libraries(<static lib> [<static lib...>])

# 指定搜索库的路径 
link_directories(<lib path>)

# 把库链接到target中
target_link_libraries(<target>
					  <AUTHORITY> <Item>
						)
```



### CMake嵌套

一个大项目下其实也可以划分出各种小项目小模块，那么每个小项目小模块其实也可以用单独的CMakeLists.txt进行管理，这些小模块中的CMakeLists.txt需要和整个大项目的CMakeList.txt建立父子关系

父节点中定义的变量可以给子节点用

```cmake
# 添加子节点所在的（CMakeLists.txt）所在的目录
add_subdirectory(<source_dir>)
```



在VSCode中使用CMmake插件时，可能因为在setting.json中错误配置了CMake源文件位置导致从子节点开始执行















