[[cmake]] 使用：
###1.`PROJECT_SOURCE_DIR`：
此CMakeLists.txt中使用`PROJECT_SOURCE_DIR`表示：所有包含`PROJECT函数`的CMakeLists.txt文件中，里自己最近的那个文件所在的文件夹路径;即离这个CMakList文件的最近的一个包含有project函数的文件的路径。
###2.`PROJECT_BINARY_DIR`：
即项目生成目录的完整路径,是最近的project函数的二进制路径。

###3.cmake关于链接库的一些用法以及区别：
1）`add_library`:
该指令的主要作用就是将指定的源文件生成链接文件，然后添加到工程中去.
```
add_library(<name> [STATIC | SHARED | MODULE]
            [EXCLUDE_FROM_ALL]
            [source1] [source2] [...])
```
[name]是要生成的链接库的名字，有[STATIC | SHARED | MODULE]三种生成方式，[sourcei]表示要生成库的源cpp或者c文件。
2)`link_directories`:
该指令的作用主要是指定要链接的库文件的路径，该指令有时候不一定需要。因为find_package和find_library指令可以得到库文件的绝对路径。不过你自己写的动态库文件放在自己新建的目录下时，可以用该指令指定该目录的路径以便工程能够找到。
```
link_directories(lib)
```
不过官网不推荐使用link_directoris，而是推荐使用find_package和find_library寻找共享库的绝对路径，再传给target_link_libraries使用。
3)`LINK_LIBRARIES`:
添加需要链接的库文件路径，注意这里是全路径
4)`target_link_libraries`:
该指令的作用为将目标文件与库文件进行链接。该指令的语法如下：
```
target_link_libraries(<target> [item1] [item2] [...]
                      [[debug|optimized|general] <item>] ...)
```
上述指令中的`<target>`是指通过`add_executable()`和`add_library()`指令生成已经创建的目标文件。而[item]表示库文件没有后缀的名字。默认情况下，库依赖项是传递的。当这个目标链接到另一个目标时，链接到这个目标的库也会出现在另一个目标的连接线上。这个传递的接口存储在interface_link_libraries的目标属性中，可以通过设置该属性直接重写传递接口。

**注意：**
1）link_libraries用在add_executable之前，target_link_libraries用在add_executable之后

2）link_libraries用来链接静态库，target_link_libraries用来链接导入库。

###4.`LD_LIBRARY_PATH`
`LD_LIBRARY_PATH`是Linux环境变量名，该环境变量主要用于指定查找共享库（动态链接库）时除了默认路径之外的其他路径

当执行函数动态链接`.so`时，如果此文件不在缺省目录`/lib`，`/usr/local/lib`和`/usr/lib`下，那么就需要指定环境变量`LD_LIBRARY_PATH`。
假如现在需要在已有的环境变量上添加新的路径名，则采用如下方式：`export LD_LIBRARY_PATH=new dirs:$LD_LIBRARY_PATH` （newdirs是新的路径串）。

**为什么需要修改LD_LIBRARY_PATH：**
因为运行时动态库的搜索路径的先后顺序是：
1.编译目标代码时指定的动态库搜索路径；
2.环境变量LD_LIBRARY_PATH指定的动态库搜索路径；
3.配置文件/etc/ld.so.conf中指定的动态库搜索路径；
4.默认的动态库搜索路径/lib和/usr/lib；

这个顺序是compile gcc时写在程序内的，通常软件源代码自带的动态库不会太多，而我们的/lib和/usr/lib只有root权限才可以修改，而且配置文件/etc/ld.so.conf也是root的事情，我们只好对LD_LIBRARY_PATH进行操作<font color=red>(当我们没有root权限的时候)</font>

###5.cmake 编译静库 动态库 指定输出路径
```
define_source_files（）

Project(CmakeTest)
aux_source_directory(. src)
[[add_executable]](project1 ${src})                        [[编译为可执行程序]]
[[add_library]](project1 ${src})                           [[编译为静态库]]
[[add_library]](project1 SHARED ${src})                    [[编译为动态链接库]]

[[add_executable]](project1 MACOSX_BUNDLE ${src})          [[编译为可执行程序]] *.app

[[add_library]](project1 MODULE ${src})                    [[编译为程序资源包]] *.bundle
[[set_target_properties]](project1 PROPERTIES BUNDLE TRUE)

[[add_library]](project1 SHARED ${src})                     [[编译为程序资源包]] *.framework
[[set_target_properties]](project1 PROPERTIES FRAMEWORK TRUE)

SET(CMAKE_CXX_FLAGS_DEBUG "$ENV{CXXFLAGS} -O0 -Wall -g -ggdb")  # Debug模式下的编译指令
SET(CMAKE_CXX_FLAGS_RELEASE "$ENV{CXXFLAGS} -O3 -Wall")         # Release模式下的编译指令

[[SET]](EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/../bin)       [[设置可执行文件的输出目录]]

[[SET]](LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/../lib)           [[设置库文件的输出目录]]


[[set]](CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG ${PROJECT_SOURCE_DIR}/bin)

[[set]](CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE ${PROJECT_SOURCE_DIR}/../bin)

[[上面两条语句分别设置了Debug版本和Release版本可执行文件的输出目录]],

[[一旦设置上面的属性]],在任何环境下生成的可执行文件都将直接放在你所设置的目录.

#    set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_DEBUG ${PROJECT_SOURCE_DIR}/../lib)
#    set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_RELEASE ${PROJECT_SOURCE_DIR}/../lib)

[[上面两条语句分别设置了Debug版本和Release版本库文件的输出目录]],

[[一旦设置上面的属性]],在任何环境下生成的库文件都将直接放在你所设置的目录.
```
