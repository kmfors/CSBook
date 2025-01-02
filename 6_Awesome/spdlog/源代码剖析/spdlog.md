# spdlog.h 文件

## spdlog.h

### 创建日志器 logger

创建日志器logger的默认API为 `spdlog::create` 函数：

```c++
template <typename Sink, typename... SinkArgs>
inline std::shared_ptr<spdlog::logger> create(std::string logger_name, SinkArgs &&...sink_args) {
    return default_factory::create<Sink>(std::move(logger_name),
                                         std::forward<SinkArgs>(sink_args)...);
}
```

创建成功后返回一个logger智能指针对象。

### 日志器 logger 操作

这里的指针对象通常指的是智能指针对象，其他称作原始指针。

```c++
// 初始化日志器
SPDLOG_API void initialize_logger(std::shared_ptr<logger> logger);

// 通过日志器名称获取日志器指针对象
SPDLOG_API std::shared_ptr<logger> get(const std::string &name);

// 设置全局格式化器 formatter
SPDLOG_API void set_formatter(std::unique_ptr<spdlog::formatter> formatter);

// 设置全局字符串模式输出
SPDLOG_API void set_pattern(std::string pattern,
                            pattern_time_type time_type = pattern_time_type::local);
// 启用回滚追踪
SPDLOG_API void enable_backtrace(size_t n_messages);
// 关闭回滚追踪
SPDLOG_API void disable_backtrace();
// 默认日志器启用dump追踪
SPDLOG_API void dump_backtrace();

// 获取全局日志等级
SPDLOG_API level::level_enum get_level();

```





使用默认记录器(stdout_color_mt)的 API，例如：`spdlog::info("Message {}"， 1);`

可以使用 spdlog::default_logger()访问默认的记录器对象：例如，添加另一个 `sink：spdlog:: default_logger()->sinks().push_back (some_sink);`

可以使用 spdlog::set_default_logger(new_logger)替换默认的日志记录器。例如，将其替换为文件记录器。

重要：默认 API 是线程安全的(对于 `_mt loggers`)，但是：`set_default_logger()` 不应该与默认 API 并发使用。例如，不要在一个线程中调用 set_default_logger()，而在另一个线程中调用 `spdlog::info()`。

```c++
SPDLOG_API std::shared_ptr<spdlog::logger> default_logger();

SPDLOG_API spdlog::logger *default_logger_raw();

SPDLOG_API void set_default_logger(std::shared_ptr<spdlog::logger> default_logger);
```

default_logger 与 default_logger_raw 什么区别呢？

default_logger 是 shared_ptr 指针，是安全指针。default_logger_raw 是原生指针。那接下来为什么无论是 API 接口还是 API 宏，其内部都是用的 default_logger_raw()去调用函数呢。因为它足够快。

因为能保证在使用这些原生指针时，不会发生内存泄漏等问题。那什么样的情况下使用智能指针呢？智能指针的出现是为了管理申请的内存。因此当新创建了一个对象时，为了使其用起来更安全必须用智能指针。因此在使用内存对象这个整体时要用智能指针，而对这个对象的操作频率有要求时，可是使用其原生指针操作。



---

根据环境配置初始化日志器级别。将 SPDLOG_LEVEL 应用于手动创建的日志器是非常有用的。例如：

```c++
auto mylogger = std::make_shared<spdlog::logger>("mylogger", …);
spdlog: apply_logger_env_levels (mylogger);
```

函数原型：

```c++
SPDLOG_API void apply_logger_env_levels(std::shared_ptr<logger> logger);
```

---

默认日志器原始的不同级别的设置。

```c++
template <typename... Args>
inline void log(source_loc source,
                level::level_enum lvl,
                format_string_t<Args...> fmt,
                Args &&...args) {
    default_logger_raw()->log(source, lvl, fmt, std::forward<Args>(args)...);
}

template <typename... Args>
inline void log(level::level_enum lvl, format_string_t<Args...> fmt, Args &&...args) {
    default_logger_raw()->log(source_loc{}, lvl, fmt, std::forward<Args>(args)...);
}

template <typename... Args>
inline void trace(format_string_t<Args...> fmt, Args &&...args) {
    default_logger_raw()->trace(fmt, std::forward<Args>(args)...);
}

template <typename... Args>
inline void debug(format_string_t<Args...> fmt, Args &&...args) {
    default_logger_raw()->debug(fmt, std::forward<Args>(args)...);
}

template <typename... Args>
inline void info(format_string_t<Args...> fmt, Args &&...args) {
    default_logger_raw()->info(fmt, std::forward<Args>(args)...);
}

template <typename... Args>
inline void warn(format_string_t<Args...> fmt, Args &&...args) {
    default_logger_raw()->warn(fmt, std::forward<Args>(args)...);
}

template <typename... Args>
inline void error(format_string_t<Args...> fmt, Args &&...args) {
    default_logger_raw()->error(fmt, std::forward<Args>(args)...);
}

template <typename... Args>
inline void critical(format_string_t<Args...> fmt, Args &&...args) {
    default_logger_raw()->critical(fmt, std::forward<Args>(args)...);
}

template <typename T>
inline void log(source_loc source, level::level_enum lvl, const T &msg) {
    default_logger_raw()->log(source, lvl, msg);
}

template <typename T>
inline void log(level::level_enum lvl, const T &msg) {
    default_logger_raw()->log(lvl, msg);
}
```

对不同级别的设置再加一层包装。

```c++
template <typename T>
inline void trace(const T &msg) {
    default_logger_raw()->trace(msg);
}

template <typename T>
inline void debug(const T &msg) {
    default_logger_raw()->debug(msg);
}

template <typename T>
inline void info(const T &msg) {
    default_logger_raw()->info(msg);
}

template <typename T>
inline void warn(const T &msg) {
    default_logger_raw()->warn(msg);
}

template <typename T>
inline void error(const T &msg) {
    default_logger_raw()->error(msg);
}

template <typename T>
inline void critical(const T &msg) {
    default_logger_raw()->critical(msg);
}
```

---

在编译时根据全局级别，启用/禁用日志调用。

在包含 spdlog.h 之前，先定义 SPDLOG_ACTIVE_LEVEL 为以下各个级别之一。

- SPDLOG_LEVEL_TRACE
- SPDLOG_LEVEL_DEBUG
- SPDLOG_LEVEL_INFO
- SPDLOG_LEVEL_WARN
- SPDLOG_LEVEL_ERROR
- SPDLOG_LEVEL_CRITICAL
- SPDLOG_LEVEL_OFF


```c++
#ifndef SPDLOG_NO_SOURCE_LOC
    #define SPDLOG_LOGGER_CALL(logger, level, ...) \
        (logger)->log(spdlog::source_loc{__FILE__, __LINE__, SPDLOG_FUNCTION}, level, __VA_ARGS__)
#else
    #define SPDLOG_LOGGER_CALL(logger, level, ...) \
        (logger)->log(spdlog::source_loc{}, level, __VA_ARGS__)
#endif

#if SPDLOG_ACTIVE_LEVEL <= SPDLOG_LEVEL_TRACE
    #define SPDLOG_LOGGER_TRACE(logger, ...) \
        SPDLOG_LOGGER_CALL(logger, spdlog::level::trace, __VA_ARGS__)
    #define SPDLOG_TRACE(...) SPDLOG_LOGGER_TRACE(spdlog::default_logger_raw(), __VA_ARGS__)
#else
    #define SPDLOG_LOGGER_TRACE(logger, ...) (void)0
    #define SPDLOG_TRACE(...) (void)0
#endif

#if SPDLOG_ACTIVE_LEVEL <= SPDLOG_LEVEL_DEBUG
    #define SPDLOG_LOGGER_DEBUG(logger, ...) \
        SPDLOG_LOGGER_CALL(logger, spdlog::level::debug, __VA_ARGS__)
    #define SPDLOG_DEBUG(...) SPDLOG_LOGGER_DEBUG(spdlog::default_logger_raw(), __VA_ARGS__)
#else
    #define SPDLOG_LOGGER_DEBUG(logger, ...) (void)0
    #define SPDLOG_DEBUG(...) (void)0
#endif

#if SPDLOG_ACTIVE_LEVEL <= SPDLOG_LEVEL_INFO
    #define SPDLOG_LOGGER_INFO(logger, ...) \
        SPDLOG_LOGGER_CALL(logger, spdlog::level::info, __VA_ARGS__)
    #define SPDLOG_INFO(...) SPDLOG_LOGGER_INFO(spdlog::default_logger_raw(), __VA_ARGS__)
#else
    #define SPDLOG_LOGGER_INFO(logger, ...) (void)0
    #define SPDLOG_INFO(...) (void)0
#endif

#if SPDLOG_ACTIVE_LEVEL <= SPDLOG_LEVEL_WARN
    #define SPDLOG_LOGGER_WARN(logger, ...) \
        SPDLOG_LOGGER_CALL(logger, spdlog::level::warn, __VA_ARGS__)
    #define SPDLOG_WARN(...) SPDLOG_LOGGER_WARN(spdlog::default_logger_raw(), __VA_ARGS__)
#else
    #define SPDLOG_LOGGER_WARN(logger, ...) (void)0
    #define SPDLOG_WARN(...) (void)0
#endif

#if SPDLOG_ACTIVE_LEVEL <= SPDLOG_LEVEL_ERROR
    #define SPDLOG_LOGGER_ERROR(logger, ...) \
        SPDLOG_LOGGER_CALL(logger, spdlog::level::err, __VA_ARGS__)
    #define SPDLOG_ERROR(...) SPDLOG_LOGGER_ERROR(spdlog::default_logger_raw(), __VA_ARGS__)
#else
    #define SPDLOG_LOGGER_ERROR(logger, ...) (void)0
    #define SPDLOG_ERROR(...) (void)0
#endif

#if SPDLOG_ACTIVE_LEVEL <= SPDLOG_LEVEL_CRITICAL
    #define SPDLOG_LOGGER_CRITICAL(logger, ...) \
        SPDLOG_LOGGER_CALL(logger, spdlog::level::critical, __VA_ARGS__)
    #define SPDLOG_CRITICAL(...) SPDLOG_LOGGER_CRITICAL(spdlog::default_logger_raw(), __VA_ARGS__)
#else
    #define SPDLOG_LOGGER_CRITICAL(logger, ...) (void)0
    #define SPDLOG_CRITICAL(...) (void)0
#endif

#ifdef SPDLOG_HEADER_ONLY
    #include "spdlog-inl.h"
#endif
```