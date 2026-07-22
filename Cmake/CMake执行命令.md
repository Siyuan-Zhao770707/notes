### CMake执行命令

```zsh

cmake CMakeLists.txt所在路径（当前目录一个点. 上个目录两个点..)

#example
#在当前目录执行
cmake .
#在上一层目录执行
cmake ..

```

#小技巧： 因为在执行cmake命令的时候会创建很多的奇奇怪怪的文件，为了保持我们的目录干净可以创建一个bulid目录，进入这个目录执行cmake命令

#小技巧： cmake执行命令的后面可以直接指定编译使用的标准，见：[[set]]
```zsh
#example
cmake CMakeLists.txt的目录 -DCMAKE_CXX_STANDARD=17


```
