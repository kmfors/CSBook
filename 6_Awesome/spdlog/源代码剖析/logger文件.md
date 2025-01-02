# logger.h 文件

线程安全日志器（除 set_error_handler()外），有名字、日志级别、流向器指针容器和格式器。在每个日志上写入记录器：

1. 检查器日志级别是否足以记录消息，如果是：
2. 调用底层的下沉器（sinks）来完这个工作
3. 每个下沉器使用自己的格式化器私有副本来格式化消息，然后送到目的地。

对每个下沉器使用私有格式器提供了缓存一些格式化数据的机会，并支持每个下沉器不同的格式。

---

看到 logger 类型的定义：

logger() 构造函数：多个构造函数，用于指定名字的日志器和下沉器的集合

log() 函数：设置源位置、日志等级和输出信息。发现这两个函数听特殊的

```c++
void log(log_clock::time_point log_time,
         source_loc loc,
         level::level_enum lvl,
         string_view_t msg) {
    bool log_enabled = should_log(lvl);
    bool traceback_enabled = tracer_.enabled();
    if (!log_enabled && !traceback_enabled) {
        return;
    }

    details::log_msg log_msg(log_time, loc, name_, lvl, msg);
    log_it_(log_msg, log_enabled, traceback_enabled);
}
```

# logger-inl.h 文件


