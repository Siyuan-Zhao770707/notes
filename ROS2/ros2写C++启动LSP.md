使用C++写ros2的代码最大的一个问题就是LSP没有办法正常工作，自动补全使用不了，把代码复制粘贴过来“假报错”满天飞，看着心烦意乱。下面是解决方法

最开始的时候，给你的项目的.cpp文件填上一个最简单的代码：
```cpp
int main(){}
```

然后按照[[ros2的C++版本#添加依赖和链接CMakeList（C++）]]来添加你需要的包，例如rclcpp

最后编译，加上设定，这样会自动添加compile_commands.json
```zsh
colcon build -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
```

然后就可以正确地使用自动补全和语法纠正了
如果你创建一个新包，每个包都需要从头这样操作（挺烦人的）