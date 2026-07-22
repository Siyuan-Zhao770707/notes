## 几个简单的命令：

1. \# 注释
2. \#\[\[     ]]多行注释
3. ***cmake_minimum_required：指定使用的Cmake版本（可选，不使用的话可能会警告）***
```cmake

cmake_minimum_required(VERSION ***)

#example

cmake_minimum_required(VERSION 3.15)

```
1. **project：定义工程名称***
```cmake

project(**name**
[VERSION ***]
[DESCRIPTION ***] 
[HOMEPAGE_URL ***]
[LANGUAGES ***])

#example
#最简单，最经常使用的
project(test)

```
5. ***add_excutable：生成一个可执行程序***
```cmake

add_executable(可执行程序 源文件1 源文件2 ……)

#example

add_executable(test main.c add.c sub.c)

```
因为add_excutable里面需要手动添加所有的源文件，所以需要更加高级的技巧
见[[set]]
见[[搜索文件]]

