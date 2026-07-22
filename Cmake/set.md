
```cmake
add_excutable(可执行程序 源文件1 源文件2 ……)

#example

add_executable(test main.c add.c sub.c)
```
[[CMakeLists.txt]]
#小技巧： 这个命令里面源文件如果一多不好管理——**可以创建变量来将源文件存储起来**

### set的使用:

1. 给变量初始化：
```cmake

set(VAR [VALUE])
#[[VAR：变量名(大小写都可以) VALUE 变量的数值 
默认情况下里面的数据类型都是字符串
意思就是如果要给变量赋值一个int，就必须要对变量进行转换]]

#example
set(SRC_LIST add.c sub.c main.c)

```
2. 使用变量：(类似于宏，直接将字符串替代)
```cmake

${VAR}

#example
add_excutable(test ${SCR_LIST})

```
### 使用场景：

1. 用一个变量名代替多个源文件名（如上)
2. 指定使用的C++标准
```cmake

#指定为C++17版本
#方式1:使用set改变CMAKE_CXX_STANDAED宏
set(CMAKE_CXX_STANDARD 17)

#方式2:在cmake执行命令之后直接指定 
cmake CMakeLists.txt的目录 -DCMAKE_CXX_STANDARD=17

```
[[CMake执行命令]]
3. 指定可执行文件生成路径(对应一个宏EXCUTABLE_OUTPUT_PATH)
```cmake

#example
set(HOME zhaosiyuan/C_prctice/CMake_practice/)
set(EXCUTABLE_OUT_PUT ${HOME}/build)

```
#小技巧： 这个宏建议使用**绝对路径**，这样生成路径不会跟着项目路径的变化而变化。如果这个路径不存在，cmake会直接创建一个路径