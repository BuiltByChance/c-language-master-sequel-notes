# P96 企业案例：日志系统与指针问题处理的架构设计，第 3 部分 error_handling

`error_handling.h` 声明三个形如 `(const char *format, ...)` 的入口。需要提示但业务可以继续时用 `log_warning`，本次操作发生错误但调用者仍可决定如何恢复时用 `log_error`。课堂为 `log_fatal` 选择了不同的策略：尝试记录 FATAL 后执行 `exit(EXIT_FAILURE)`，结束整个进程。记录一条 ERROR 不会自动回滚操作，调用者仍要处理自己的失败路径。

## 为什么可变参数需要专门的转发入口

`error_handling.c` 中的三个函数都用 `va_start(args, format)` 获取参数列表。课堂接着把 `args` 作为一个实参传给上一节的 `log_message(level, format, ...)`。这一写法**不会展开参数列表**。例如，调用者用 `"%d"` 和整数 42 时，后端实际收到的额外实参是 `args`，不能按原来的整数读取。只有固定文字、没有任何转换项的测试也不足以发现这种错误。

**整理者修订。** 在 `logger.h` 另行声明 `void vlog_message(LogLevel level, const char *format, va_list args);`。把原有日志函数中的阈值过滤、时间和级别输出、`vfprintf(logFile, format, args)`、换行与刷新移入这个入口。原来的 `log_message(...)` 和三个错误函数都负责建立自己的列表，再交给同一后端。下面以警告函数为例。[Microsoft Learn 的 vfprintf 文档](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/vfprintf-vfprintf-l-vfwprintf-vfwprintf-l?view=msvc-170)

```c
void log_warning(const char *format, ...)
{
    va_list args;
    va_start(args, format);
    vlog_message(LOG_WARNING, format, args);
    va_end(args);
}
```

验证时，先初始化日志模块并允许 WARNING 级别，调用 `log_warning("id=%d name=%s", 42, "Frank")`，应看到带 WARNING 标签且正文为 `id=42 name=Frank` 的记录。致命级别应另开一次运行，确认记录之后的语句未执行并检查进程失败状态。这是整理者给出的验证方法；课堂只在编辑器中完成三个函数，没有运行、显示日志或验证退出状态。指针检查留待后续部分。
