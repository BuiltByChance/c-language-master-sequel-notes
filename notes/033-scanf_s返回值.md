# P33 scanf_s 返回值

`int result = scanf_s("%d", &number);` 涉及两个值。`number` 接收读到的整数，`result` 告诉程序成功转换并赋值了几个字段。本例只要求一个字段，因此只有 `result == 1` 时才输出 `number`。

| `result` | 处理方式 | 现场证据 |
| --- | --- | --- |
| `1` | 一个整数已写入 `number` | 输入 `123123123` 后，程序显示 `You entered the integer: 123123123` |
| `0` | 首个输入项无法匹配 `%d`，提示用户需要整数 | 输入以字母开头的文本，程序显示 `Invalid input for integer.`。随后进程退出码为 1，来自 `main` 的 `return 1` |
| `EOF` | 第一次转换完成前输入已结束或读取失败。需要进一步区分时，分别检查 `feof(stdin)` 和 `ferror(stdin)` | 讲师写了分支，课堂程序没有展示这一条提示的运行结果 |

如果格式要求两个值，就比较返回值与 `2`；这时 `1` 表示只完成了第一项。也不要把返回值当成整行内容的验证。整理者举例，`%d` 可以在 `12abc` 中成功读出 `12`，剩下的字符留在输入流。[Microsoft Learn：scanf_s 返回值](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/scanf-s-scanf-s-l-wscanf-s-wscanf-s-l?view=msvc-170)

**两处容易混淆的现场结果**

- 讲师输入远超 `int` 范围的数，控制台显示 `You entered the integer: -1`，进程退出码为 0。这里打印的是 `number`，走的是 `result == 1` 分支。口述的“返回 −1”容易把两者混在一起。不能用该数判断溢出；需要验证整数范围时，可以先读取整行，再用 `strtol` 检查转换终点、`ERANGE` 及目标类型的上下限。[WG14 C11 草案 §7.21.6.2](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=337)
- `EOF` 是 `<stdio.h>` 定义的负 `int` 常量，代码直接与 `EOF` 比较即可。字幕漏掉了口述“负一”的负号。讲师尝试快捷键时所在的窗口是 `cmd` 命令提示符，尚未运行这个输入程序；Windows 控制台默认把 Ctrl+C 当作中断信号。它不能验证 `scanf_s` 的 EOF 分支。[Microsoft Learn：Windows 控制台的 Ctrl+C 行为](https://learn.microsoft.com/en-us/windows/console/ctrl-c-and-ctrl-break-signals)
