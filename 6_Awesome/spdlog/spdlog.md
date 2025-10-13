# spdlog.h 文件

## 前提概要

- **spdlog log API** —— 是建立在 logger 之上的，只是对 logger 使用的封装，目的只是为了能够像官网给的示例代码 spdlog::info ("Welcome to spdlog!"); 那样，让用户能够以最简单的方式使用 spdlog 打印出 log。这是一种从用户使用维度出发的程序设计思想。
- **logger** —— 是 spdlog 开始处理日志的入口。sync-logger 主要负责日志信息的整理，将格式化（通过第三方库 fmt）后的日志内容、日志等级、日志时间等信息“整理”到一个名为 log_msg 结构体的对象中，然后再交给下游的 sink 进行处理。而对于 async-logger，则是在将整理后的 log_msg 对象交给线程池，让线程池去处理后续的工作。
- **sink** —— 接收 log_msg 对象，并通过 formatter 将对象中所含有的信息转换成字符串，最后将字符串输出到指定的地方，例如控制台、文件等，甚至通过 tcp/udp 将字符串发送到指定的地方。sink 译为“下沉”，扩展一下可以理解为“落笔”，做的是把日志真正记录下来的事情。
- **formatter** —— 负责将 log_msg 对象中的信息转换成字符串，例如将等级、时间、实际内容等。时间的格式和精度、等级输出显示的颜色等都是由 formatter 决定的。支持用户自动以格式。
- **registry** —— 负责管理所有的 logger，包括创建、销毁、获取等。通过 registry 用户还可以对所有的 logger 进行一些全局设置，例如设置日志等级。

---

raw：未经加工的

spdlog::info ("Some info to spdlog!");

来自于下面这个代码：

```c++
template <typename T>
inline void info(const T &msg) {
    default_logger_raw()->info(msg);
}
```

```c++
spdlog::set_level(spdlog::level:: debug); 

SPDLOG_INLINE void set_level(level:: level_enum log_level) {
    details::registry:: instance().set_level(log_level);
}
```

logger 类，没错是用来负责日志信息的整理，那这样的话，logger 的构造是需要传入日志名与 sinks，sinks 是用来将日志信息输出到本地的。

--

``` c++
namespace sinks {
class SPDLOG_API sink {
public:
    virtual ~sink() = default;
    virtual void log(const details:: log_msg &msg) = 0;
    virtual void flush() = 0;
    virtual void set_pattern(const std:: string &pattern) = 0;
    virtual void set_formatter(std:: unique_ptr <spdlog::formatter> sink_formatter) = 0;

    void set_level(level:: level_enum log_level);
    level:: level_enum level() const;
    bool should_log(level:: level_enum msg_level) const;

protected:
    // sink log level - default is all
    level_t level_{level:: trace};
};

} 
```

它的方法不多，来一一分析：

- 析构函数为虚函数是默认的虚构函数，启用默认的析构函数有什么用么？
- log()：虚函数，用于传入日志消息
- flush()：刷新缓冲区，将日志消息记录于本地
- set_pattern()：设置日志模式内容，规范日志消息内容
- set_formatter()：设置格式器，用于日志消息的格式化
- set_level()：设置日志等级
- level()：获取日志等级
- should_log(): 判断 log 的日志等级与传入的是否一致

看样子，前几个函数都是虚函数，应该能联想到是因为有不同的 sinks 来实现不同的功能特殊性。而非虚函数，则是适用于所有的 sinks，相当于抽象工厂设计模式，通过一个统一的接口创建一个抽象工厂。

