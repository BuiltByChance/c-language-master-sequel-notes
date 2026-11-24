# P61 time.h 与时间戳的使用

同一时刻可以记为一个时间戳，也可以显示为本地时间或协调世界时（UTC）。在课堂使用的微软 C 运行库里，`time(NULL)` 返回自 `1970-01-01 00:00:00 UTC` 起经过的秒数，类型为 `time_t`。C 语言没有把所有实现的 `time_t` 都规定为普通 `int`；符号 `%d` 不能直接套用到任何 `time_t` 上。[Microsoft Learn 的 time 说明](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/time-time32-time64?view=msvc-170)

1. 用 `time_t now = time(NULL);` 只取一次系统时钟。失败时返回 `(time_t)-1`。
2. 准备两个分解后的日期时间对象：`struct tm local_time, utc_time;`。课堂 MSVC 的 `localtime_s(&local_time, &now)` 按本地时区分解，`gmtime_s(&utc_time, &now)` 按 UTC 分解。两者均先传目标结构体地址，再传时间戳地址，返回 `0` 表示成功。[微软的 localtime_s 说明](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/localtime-s-localtime32-s-localtime64-s?view=msvc-170)
3. 给两个结构体各准备一个 `char` 数组。课堂用了各 80 字节。调用 `strftime(local_text, sizeof local_text, "%Y-%m-%d %H:%M:%S", &local_time)` 后，用 `%s` 打印 `local_text`。UTC 一行换成对应的目标数组、容量和 `&utc_time`。格式中的 `%m` 是月份，`%M` 是分钟；`%H` 是 24 小时制。对此非空格式，返回大于 0 才使用生成的字符串。若还要输出课堂环境里的数字时间戳，可包含 `<inttypes.h>`，写 `printf("%" PRIdMAX "\n", (intmax_t)now);`。[微软的 strftime 说明](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/strftime-wcsftime-strftime-l-wcsftime-l?view=msvc-170)

课堂构建第一次失败时，`localtime_s` 和 `strftime` 两处误传 `&localtime`。少写下划线后，名字指向了库函数；真正声明的结构体变量 `local_time` 反而未使用。两处都改成 `&local_time` 后成功运行。讲师没有在程序中检查各次调用的返回值，上述检查为整理者补充。

| 现场输出 | 值 |
| --- | --- |
| 时间戳 | `1708473860` |
| 本地时间 | `2024-02-20 16:04:20` |
| UTC | `2024-02-21 00:04:20` |

这组输出相差 8 小时且跨了日期，表明课堂机器在这个时刻的本地偏移为 UTC−08:00。自己运行时，应核对同一个 `now` 是否依本机时区换算；时间戳和本地偏移都可能与课堂不同。
