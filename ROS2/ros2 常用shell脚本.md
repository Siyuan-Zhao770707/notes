都放在工作空间的根目录里面。我使用的是zsh

## build.sh
这个脚本能够编译你的文件，并且能够产生compile_commands.json，就能启用你的C++的LSP了
```zsh
  source /opt/ros/jazzy/setup.zsh
  # 第一次 build 前 install/ 还不存在，用 || true 跳过
  source install/setup.bash 2>/dev/null || true
  colcon build --symlink-install --cmake-args -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
  ln -sf build/compile_commands.json .
```
使用方式
```zsh
zsh build.sh
```

## clean.sh
这个脚本你可以随时清理编译文件，不至于让你的磁盘爆炸
```shell
rm -rf build
rm -rf install
rm -rf log
rm -f complie_command.json

```
使用方式
```zsh
zsh clearn.sh
```

## source.sh
这个脚本能够减轻你环境配置的负担
```shell
source /opt/ros/jazzy/setup.zsh
source ~/ros2_ws/install/setup.zsh

```
使用方式
**注意！！！：不能够使用zsh source.sh**
zsh会创建一个一个子shell来运行里面的程序，运行玩就消失了，你当前的配置根本没动
```zsh
source ./source.sh
```

## release_build.sh
到正式比赛或者要发布的时候使用，平常使用的少
这种编译方式不带debug文件，编译出来的文件体积要比普通方法小得多
```shell
colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release
```
使用方式
```zsh
zsh release_build.sh
```