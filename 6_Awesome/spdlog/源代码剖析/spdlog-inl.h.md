
会发现一个挺有意思的点：就是为什么要使用 std::move 呢？

std:move 的作用是将左值转换为右值，这样的好处的就是方便转移对这个内存资源的所有权。不然如果我们一直用这个引用的话，虽然也可以继续操作，但是要求我们不能删除或脱离作用域销毁的该内存资源的所有权拥有者（一开始创建申请所赋予的变量对象）。这样很不方便。而如果现在可以通过转移内存资源所有权的形式，我们就可以牢牢地把握这个内存资源为我们已经规范化的代码所使用，而不再受限于客户对无脑行为。

```c++
// Copyright(c) 2015-present, Gabi Melman & spdlog contributors.
// Distributed under the MIT License (http://opensource.org/licenses/MIT)

#pragma once

#ifndef SPDLOG_HEADER_ONLY
    #include <spdlog/spdlog.h>
#endif

#include <spdlog/common.h>
#include <spdlog/pattern_formatter.h>

namespace spdlog {

SPDLOG_INLINE void initialize_logger(std::shared_ptr<logger> logger) {
    details::registry::instance().initialize_logger(std::move(logger));
}

SPDLOG_INLINE std::shared_ptr<logger> get(const std::string &name) {
    return details::registry::instance().get(name);
}

SPDLOG_INLINE void set_formatter(std::unique_ptr<spdlog::formatter> formatter) {
    details::registry::instance().set_formatter(std::move(formatter));
}

SPDLOG_INLINE void set_pattern(std::string pattern, pattern_time_type time_type) {
    set_formatter(
        std::unique_ptr<spdlog::formatter>(new pattern_formatter(std::move(pattern), time_type)));
}

SPDLOG_INLINE void enable_backtrace(size_t n_messages) {
    details::registry::instance().enable_backtrace(n_messages);
}

SPDLOG_INLINE void disable_backtrace() { details::registry::instance().disable_backtrace(); }

SPDLOG_INLINE void dump_backtrace() { default_logger_raw()->dump_backtrace(); }

SPDLOG_INLINE level::level_enum get_level() { return default_logger_raw()->level(); }

SPDLOG_INLINE bool should_log(level::level_enum log_level) {
    return default_logger_raw()->should_log(log_level);
}

SPDLOG_INLINE void set_level(level::level_enum log_level) {
    details::registry::instance().set_level(log_level);
}

SPDLOG_INLINE void flush_on(level::level_enum log_level) {
    details::registry::instance().flush_on(log_level);
}

SPDLOG_INLINE void set_error_handler(void (*handler)(const std::string &msg)) {
    details::registry::instance().set_error_handler(handler);
}

SPDLOG_INLINE void register_logger(std::shared_ptr<logger> logger) {
    details::registry::instance().register_logger(std::move(logger));
}

SPDLOG_INLINE void apply_all(const std::function<void(std::shared_ptr<logger>)> &fun) {
    details::registry::instance().apply_all(fun);
}

SPDLOG_INLINE void drop(const std::string &name) { details::registry::instance().drop(name); }

SPDLOG_INLINE void drop_all() { details::registry::instance().drop_all(); }

SPDLOG_INLINE void shutdown() { details::registry::instance().shutdown(); }

SPDLOG_INLINE void set_automatic_registration(bool automatic_registration) {
    details::registry::instance().set_automatic_registration(automatic_registration);
}

SPDLOG_INLINE std::shared_ptr<spdlog::logger> default_logger() {
    return details::registry::instance().default_logger();
}

SPDLOG_INLINE spdlog::logger *default_logger_raw() {
    return details::registry::instance().get_default_raw();
}

SPDLOG_INLINE void set_default_logger(std::shared_ptr<spdlog::logger> default_logger) {
    details::registry::instance().set_default_logger(std::move(default_logger));
}

SPDLOG_INLINE void apply_logger_env_levels(std::shared_ptr<logger> logger) {
    details::registry::instance().apply_logger_env_levels(std::move(logger));
}

}  // namespace spdlog

```