
### 添加依赖和链接CMakeList（C++）

#### 添加依赖

package.xml
```xml
<!-- 上面的部分如果你不发表就没有必要修改 -->
<?xml version="1.0"?>
<?xml-model href="http://download.ros.org/schema/package_format3.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>
<package format="3">
  <name>learning_topic_C++</name>
  <version>0.0.0</version>
  <!--description里面放对于这个包的功能的描述-->
  <description>TODO: Package description</description>
  <!--作者的邮件名称和作者名字-->
  <maintainer email="2285518665@qq.com">zhaosiyuan</maintainer>
  <!--Apache License 2.0 开源许可证-->
  <license>TODO: License declaration</license>

  <buildtool_depend>ament_cmake</buildtool_depend>
  
  <!-- 重头戏！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！！--> 
  <!-- 头文件依赖必须写上，只用写ros2带的库 -->
  <depend>rclcpp</depend> 
  <depend>std_msgs</depend>

  <test_depend>ament_lint_auto</test_depend>
  <test_depend>ament_lint_common</test_depend>

  <export>
    <build_type>ament_cmake</build_type>
  </export>
</package>

```

#### 添加链接

CMakeList.txt
```cmake
cmake_minimum_required(VERSION 3.8)
project(learning_topic_C++)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# 添加在头文件里面额外引用的包
# find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)
# uncomment the following section in order to fill in
# further dependencies manually.
# find_package(<dependency> REQUIRED)

# 编译成可执行文件（可执行文件名称 源代码文件名称）
add_executable(learning_topic_C++_pub src/learning_topic_C++_pub.cpp)
add_executable(learning_topic_C++_sub src/learning_topic_C++_sub.cpp)
# 编译时的依赖（目标文件的可执行文件名 依赖1 依赖2 ……）
ament_target_dependencies(learning_topic_C++_pub rclcpp std_msgs)
ament_target_dependencies(learning_topic_C++_sub rclcpp std_msgs)

# 让命令行能够识别包，方便直接ros2 run 
install(TARGETS
  learning_topic_C++_pub
  learning_topic_C++_sub
  DESTINATION lib/${PROJECT_NAME})

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  # the following line skips the linter which checks for copyrights
  # comment the line when a copyright and license is added to all source files
  set(ament_cmake_copyright_FOUND TRUE)
  # the following line skips cpplint (only works in a git repo)
  # comment the line when this package is in a git repo and when
  # a copyright and license is added to all source files
  set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()

```

---
### 话题（C++）

#### publisher

```cpp
#include <chrono>
#include <memory>
#include <string>

#include "rclcpp/rclcpp.hpp" //标准ros库
#include "std_msgs/msg/string.hpp" //ros2标准通信接口的字符串数据类型

// 使用chrono的命名空间
// 可以将原来的时间表达式从std::chrono::milliseconds(500)简化为500ms
using namespace std::chrono_literals;

// 从ros自带的node父类继承并创建类
class MinimalPublisher : public rclcpp::Node
{
// 编写代码从外向内，先写可以看到的public接口，再写private的内部细节
public:
  // MinimalPublisher的构造函数，初始化类的名称和序号（让发布的消息序号从0递增）
  MinimalPublisher()
  : Node("minimal_publisher"), count_(0)
  {
    // 使用父类封装的函数来创建发布消息数据类型为字符串的，名称为topic的publisher
    // 10的意思是创建一个长度为10的队列，出现网络问题消息堵车的时候能够暂存10条消息
    publisher_ = this->create_publisher<std_msgs::msg::String>("topic", 10);
    
    // 定义回调函数，使用了lamda函数简化
    // [this]捕获列表，能够传入指针，能够在在函数体里面直接使用this指针
    // ()参数列表，可以传参数进去
    // ->void返回类型表示不返回任何值
    // time_callback这变量代表的就是这个函数本身，不是函数返回值
    auto timer_callback =
      [this]() -> void {
        auto message = std_msgs::msg::String();
        message.data = "Hello, world! " + std::to_string(this->count_++);
        
	    // RCLCPP_INFO: ROS2 info日志宏，日志格式化打印工具
		// get_logger(): 获取节点日志器
		// "Publishing: '%s'": 格式化字符串
		// c_str(): 转C风格字符串，一定要转换不然会报错
        RCLCPP_INFO(this->get_logger(), "Publishing: '%s'", message.data.c_str());
        // 真正地发布话题
        this->publisher_->publish(message);
      };
    // 每500ms启动一下timer_callback函数
    timer_ = this->create_wall_timer(500ms, timer_callback);
  }

private:
  // 私有变量声明，在上面使用了这些变量
  // Shared_ptr 智能指针可以自己回收内存
  rclcpp::TimerBase::SharedPtr timer_;
  rclcpp::Publisher<std_msgs::msg::String>::SharedPtr publisher_;
  size_t count_;
};

// argc 和 * argv[]这两个参数不能简化，下面第一行就使用了这两个参数
int main(int argc, char * argv[])
{
  rclcpp::init(argc, argv);// 初始化ros2程序
  // 使用智能指针创建MinimalPublisher的类对象并且传入spin函数里面
  // spin函数让程序进入阻塞状态，进入无限循环并循环执行类里面的回调函数，ctrl+c终止
  // 这一部分非常容易被遗忘也非常容易给写成普通的智能指针(std::shared_ptr)
  // 千万要注意，很多时候如果你编译成功但是运行没反应都是这个问题
  auto node = std::make_shared<MinimalPublisher>();
  rclcpp::spin(node);
  rclcpp::shutdown();
  return 0;
}
```

#### subscriber

```cpp
#include <memory>

#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/string.hpp"

class MinimalSubscriber : public rclcpp::Node
{
public:
  MinimalSubscriber()
  : Node("minimal_subscriber")
  {
    auto topic_callback =
      [this](std_msgs::msg::String::UniquePtr msg) -> void {
        RCLCPP_INFO(this->get_logger(), "I heard: '%s'", msg->data.c_str());
      };
		/* 虽然我们看不到传参数，但是在ros2的底层系统里面，当收到参数的时候他会自动将参数填入msg这个参数里面然后自动帮你调用topic_callback(msg)*/
    subscription_ =
      this->create_subscription<std_msgs::msg::String>("topic", 10, topic_callback);
  }

private:
  rclcpp::Subscription<std_msgs::msg::String>::SharedPtr subscription_;
};

int main(int argc, char * argv[])
{
  rclcpp::init(argc, argv);
  auto node = std::make_shared<MinimalSubscriber>();
  rclcpp::spin(node);
  rclcpp::shutdown();
  return 0;
}
```

---
### 服务（C++）
#### server
在写这个之前要先添加add_tow_ints.hpp接口
详细见[[通信接口#定义接口（以.msg为例子）]]
```cpp
#include "rclcpp/rclcpp.hpp"
// 自定义的通信接口
/* 这里和python不一样，.serv等等接口文件会被编译成可以被C++识别的.hpp文件，直接引用的是.hpp文件 */
#include "example_interfaces/srv/add_two_ints.hpp"

#include <memory>

void add(
    const std::shared_ptr<example_interfaces::srv::AddTwoInts::Request> request,
    std::shared_ptr<example_interfaces::srv::AddTwoInts::Response>      response)
{
    response->sum = request->a + request->b;
    RCLCPP_INFO(
        rclcpp::get_logger("rclcpp"),
        "Incoming request\na: %ld" " b: %ld",
        request->a,
        request->b);
    RCLCPP_INFO(
        rclcpp::get_logger("rclcpp"),
        "sending back response: [%ld]",
        (long int)response->sum);
}

int main(int argc, char **argv)
{
    rclcpp::init(argc, argv);

    std::shared_ptr<rclcpp::Node> node = rclcpp::Node::make_shared("add_two_ints_server");

    rclcpp::Service<example_interfaces::srv::AddTwoInts>::SharedPtr service =
        node->create_service<example_interfaces::srv::AddTwoInts>("add_two_ints", &add);

    RCLCPP_INFO(rclcpp::get_logger("rclcpp"), "Ready to add two ints.");

    rclcpp::spin(node);
    rclcpp::shutdown();
}
```

#### client
```cpp
#include "rclcpp/rclcpp.hpp"
#include "example_interfaces/srv/add_two_ints.hpp"

#include <chrono>
#include <cstdlib>
#include <memory>

using namespace std::chrono_literals;

int main(int argc, char **argv)
{
  /* argc是命令行传入的参数个数，包含程序本身。argv是参数的字符串数组，argv[0]是程序名，argv[1]、argv[2]等等是后续的参数 */
  rclcpp::init(argc, argv);

  /* 如果输入的参数数量不足（两个加数+程序自身）就执行 */
  if (argc != 3) {
      RCLCPP_INFO(rclcpp::get_logger("rclcpp"), "usage: add_two_ints_client X Y");
      // 退出主函数并返回非0数值，程序因为错误而终止
      return 1;
  }

  // 创建节点
  std::shared_ptr<rclcpp::Node> node = rclcpp::Node::make_shared("add_two_ints_client");
  rclcpp::Client<example_interfaces::srv::AddTwoInts>::SharedPtr client =
    node->create_client<example_interfaces::srv::AddTwoInts>("add_two_ints");

  auto request = std::make_shared<example_interfaces::srv::AddTwoInts::Request>();
  // atoll()将参数1和参数2的字符串转化成longlong类型
  request->a = atoll(argv[1]);
  request->b = atoll(argv[2]);

  // 阻塞等待服务端上线，如果没有上线，没一秒钟执行一次循环
  while (!client->wait_for_service(1s)) {
    // rclcpp::ok() = true 当前程序正常运行 = false 为当前程序被终止
    if (!rclcpp::ok()) {
      RCLCPP_ERROR(rclcpp::get_logger("rclcpp"), "Interrupted while waiting for the service. Exiting.");
      return 0;
    }
    RCLCPP_INFO(rclcpp::get_logger("rclcpp"), "service not available, waiting again...");
  }
  // 异步接受服务端的response
  auto result = client->async_send_request(request);
  // Wait for the result.
  /* 让程序在这里卡死，直到result的结果出现
  rclcpp::FutureReturnCode::SUCCESS 成功接收到了服务
  rclcpp::FutureReturnCode::TIMEOUT 超时了没有接收打服务
  rclcpp::FutureReturnCode::INTERUPTED 被打断了没有接收到服务
  如果是类的写法的话，第一个node要填写的是this->shared_from_this()
  将普通的类指针转换成给ros2节点专用的节点智能共享指针*/
  if (rclcpp::spin_until_future_complete(node, result) ==
    rclcpp::FutureReturnCode::SUCCESS)
  {
    RCLCPP_INFO(rclcpp::get_logger("rclcpp"), "Sum: %ld", result.get()->sum);
  } else {
    RCLCPP_ERROR(rclcpp::get_logger("rclcpp"), "Failed to call service add_two_ints");
  }

  rclcpp::shutdown();
  return 0;
}
```

类版本，只保留骨架
```cpp
#include "rclcpp/rclcpp.hpp"
#include "example_interfaces/srv/add_two_ints.hpp"

#include <chrono>
#include <cstdlib>
#include <memory>

using namespace std::chrono_literals;

class AddTwoIntsClient : public rclcpp::Node
{
public:
  // 构造函数：传入两个加数，并初始化节点名称和客户端
  AddTwoIntsClient(int a, int b)
    : Node("add_two_ints_client"), a_(a), b_(b)
  {
    client_ = this->create_client<example_interfaces::srv::AddTwoInts>("add_two_ints");
  }

  // 执行请求的主逻辑，返回是否成功
  // 注意！！：在创建函数的时候不要忘记函数名称后面的那个()
  bool send_request()
  {
    // 1. 等待服务可用
    while (!client_->wait_for_service(1s)) {
      if (!rclcpp::ok()) {
        RCLCPP_ERROR(this->get_logger(), "Interrupted while waiting for the service. Exiting.");
        return false;
      }
      RCLCPP_INFO(this->get_logger(), "service not available, waiting again...");
    }

    // 2. 构造请求
    // 注意！！！！！！auto这个自动检测数据类型只能够在类的成员函数里面使用！！！
    // 如果在外面使用会报错
    auto request = std::make_shared<example_interfaces::srv::AddTwoInts::Request>();
    request->a = a_;
    request->b = b_;

    // 3. 异步发送请求
    auto result = client_->async_send_request(request);

    // 4. 等待响应（使用 this->shared_from_this() 传入节点智能指针）
    if (rclcpp::spin_until_future_complete(this->shared_from_this(), result) ==
        rclcpp::FutureReturnCode::SUCCESS)
    {
      RCLCPP_INFO(this->get_logger(), "Sum: %ld", result.get()->sum);
      return true;
    } else {
      RCLCPP_ERROR(this->get_logger(), "Failed to call service add_two_ints");
      return false;
    }
  }

private:
  int a_;
  int b_;
  //千万不要忘记下面的最后的SharedPtr!!!!!!!
  rclcpp::Client<example_interfaces::srv::AddTwoInts>::SharedPtr client_;
};

int main(int argc, char **argv)
{
  rclcpp::init(argc, argv);

  if (argc != 3) {
    RCLCPP_INFO(rclcpp::get_logger("rclcpp"), "usage: add_two_ints_client X Y");
    return 1;
  }

  int a = std::stoll(argv[1]);  // 更安全的转换
  int b = std::stoll(argv[2]);

  // 创建节点实例（智能指针）
  auto node = std::make_shared<AddTwoIntsClient>(a, b);
  
  // 执行请求
  bool success = node->send_request();

  rclcpp::shutdown();
  return success ? 0 : 1;
}
```
---
### 参数
```cpp
#include <chrono>
#include <functional>
#include <string>

#include <rclcpp/rclcpp.hpp>

using namespace std::chrono_literals;

class MinimalParam : public rclcpp::Node
{
public:
  MinimalParam()
  : Node("minimal_param_node")
  {
    /* 
    或者可以使用下面这个来给参数增加注释
    auto param_desc = rcl_interfaces::msg::ParameterDescriptor{};
    param_desc.description = "This parameter is mine!";

    this->declare_parameter("my_parameter", "world", param_desc); 
    */
    this->declare_parameter("my_parameter", "world");

    auto timer_callback = [this](){
      std::string my_param = this->get_parameter("my_parameter").as_string();

      RCLCPP_INFO(this->get_logger(), "Hello %s!", my_param.c_str());

	  //因为set_parameters()函数接收vector数组，所以需要转变成数组再传入函数
	  /*
	  如果想要重制多个参数可以这样写
		std::vector<rclcpp::Parameter>all_new_parameters{
		    rclcpp::Parameter("speed", 1.0),
		    rclcpp::Parameter("name", "robot"),
		    rclcpp::Parameter("verbose", true)
		};
	  */
      std::vector<rclcpp::Parameter> all_new_parameters{rclcpp::Parameter("my_parameter", "world")};
      this->set_parameters(all_new_parameters);
    };
    timer_ = this->create_wall_timer(1000ms, timer_callback);
  }

private:
  rclcpp::TimerBase::SharedPtr timer_;
};

int main(int argc, char ** argv)
{
  rclcpp::init(argc, argv);
  rclcpp::spin(std::make_shared<MinimalParam>());
  rclcpp::shutdown();
  return 0;
}
```

