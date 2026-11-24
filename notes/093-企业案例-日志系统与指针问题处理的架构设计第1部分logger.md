# P93 企业案例：日志系统与指针问题处理的架构设计，第 1 部分 logger

这一部分实现一个供其他模块调用的日志接口。同一条消息可以送往控制台或文件，调用方只指定级别、格式串和参数。开头还规划了内存分配与释放、统一处理错误、检查指针等工作，本节只写到基础类型和日志模块。

## 项目与文件

在 Visual Studio 的解决方案资源管理器中右击项目，选择“属性 → 配置属性 → 常规 → C 语言标准”。画面显示当前配置为 Debug、平台为 x64、C 语言标准为 ISO C11（`/std:c11`）；不能由此推定其他配置同步改变。课堂在 Header Files 添加 `types.h`、`logger.h`，在 Source Files 添加 `logger.c`，使用 `.c` 扩展名编译 C 源码。[微软对 C11/C17 项目配置的说明](https://learn.microsoft.com/zh-cn/cpp/overview/install-c17-support?view=msvc-170)

- `types.h` 包含 `<stdint.h>`，把 `int32_t`、`uint32_t`、`int64_t`、`uint64_t` 分别命名为 `int32`、`u32`、`int64`、`u64`。要使用这些项目别名，包含 `"types.h"` 即可。
- `logger.h` 定义枚举 `LogLevel`，按从低到高排列 DEBUG、INFO、WARNING、ERROR、FATAL，并声明四个函数。`logger.c` 保存仅本文件可直接访问的当前输出流 `logFile` 和最低级别 `currentLevel`，初值分别为 `NULL` 和 `LOG_INFO`。

## 四个调用如何配合

| 接口 | 调用者应如何判断 |
| --- | --- |
| `logger_init(path)` | `path == NULL` 时借用标准输出；传文件路径时用微软 `fopen_s` 以 `"a"` 打开，成功后追加而保留之前的内容。返回 `false` 表示未得到可写文件，应在此处理初始化失败。 |
| `set_log_level(level)` | 设置允许写出的最低级别。例如设置 `LOG_WARNING`，会记录 WARNING、ERROR 和 FATAL，跳过 INFO 和 DEBUG。 |
| `log_message(level, format, ...)` | 检查是否已有输出流，以及消息级别是否达到阈值。符合条件才写入时间、级别、由 `vfprintf` 填好参数的正文和换行。记录 FATAL 在这个模块里也只是写日志。 |
| `logger_close()` | 输出目标是自己打开的文件才 `fclose`，借用的 `stdout` 保持打开。然后将 `logFile` 置为 `NULL`，后续消息会被跳过。 |

`log_message` 用 `time` 和微软 `localtime_s` 取得本地时间，年取 `tm_year + 1900`，月取 `tm_mon + 1`。加 1900 是因为 `tm_year` 按距 1900 年的年数表示。函数把 `va_start(args, format)` 初始化的列表交给 `vfprintf(logFile, format, args)`，结束后调 `va_end`，最后换行并 `fflush(logFile)`。刷新把 C 流中待写的数据交给下层；如需确认写入是否成功，还要检查输出、刷新和关闭的返回值。课堂没有检查这些结果。[微软 fflush 文档](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/fflush?view=msvc-170)

**整理者的验证方法。** 调用 `logger_init(NULL)` 成功后，把阈值设为 WARNING，依次写一条 INFO、一条 ERROR；预期只出现带时间和 `[ERROR]` 的后一条。关闭后再以文件路径初始化并重复运行，重开文件应看到两次的记录按顺序保留。课中只在编辑器内完成接口和实现，没有构建、运行或检查日志文件。片尾未知级别分支写成 `default:LOG_FATAL:` 并打印 `[UNKNOW]`；多出的普通标签可以删除，若要显示英文“未知”应拼作 `[UNKNOWN]`。
