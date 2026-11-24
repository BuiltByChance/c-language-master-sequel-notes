# P32 复习 scanf_s

在微软 C 运行库中，`scanf_s` 从 `stdin` 按格式读取数据。决定参数的关键是转换说明符。下表每一行都是一次独立读取，均需要包含 `<stdio.h>`。

| 目标 | 写法 | 读取规则与现场结果 |
| --- | --- | --- |
| `char name[50]` | `scanf_s("%49s", name, (unsigned)sizeof name)` | `%s` 跳过开头空白，读一个词。`49` 是最大数据宽度，容量 `50` 包含结尾 `\0`。课堂输入 `Frank` 后显示 `Hello, Frank!` |
| `char ch` | `scanf_s("%c", &ch, 1u)` | 接收一个字符，空格和换行也能被读入，不追加 `\0`。课堂输入字符 `3`，回显 `3` |
| `int age` | `scanf_s("%d", &age)` | 目标是一个整数，只传地址。课堂输入 `1231231`，回显同一数值 |

在 `name` 的声明处，`sizeof name` 正好给出整个 `char` 数组的容量，微软接口的容量参数类型为 `unsigned`。课堂把数组改大到 123 时，`sizeof name` 随之变化，写在格式里的 `%49s` 仍最多读取 49 个非空白字符。若改为在函数中接收 `char *name`，应由调用者传进数组容量。整理者补充，单项输入可以用 `scanf_s(...) == 1` 判断是否成功赋值，再使用目标变量；本节将返回值留待后讲。[Microsoft Learn：scanf_s 的参数与返回值](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/scanf-s-scanf-s-l-wscanf-s-wscanf-s-l?view=msvc-170)

## 两次构建诊断怎样解读

- 约 03:25，讲师只包含了 `<stdio.h>` 就尝试 `_countof(name)`，项目提示未定义并链接失败，由此判断新版 Visual Studio 已删除这个宏。截至 2026-11-24，微软文档仍把 `_countof` 列在 `<stdlib.h>` 中。包含该头文件后可在数组声明可见处用 `(unsigned)_countof(name)` 传元素数。[Microsoft Learn：_countof 宏](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/countof-macro?view=msvc-170)
- 讲师演示用 `#define scanf scanf_s` 让预处理器替换后面的函数名。整数调用的参数恰好相同，项目能够构建；删除宏、保留 `scanf` 时，该项目以 C4996 构建失败，最后改回直接调用 `scanf_s`。这种宏仅替换名称，遇到 `%c`、`%s` 等需要额外容量参数的格式，仍须按各自的接口写全参数。
