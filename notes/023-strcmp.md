# P23 strcmp

判断两个以 `\0` 结尾的字符串内容是否相等，包含 `<string.h>` 后写 `strcmp(a, b) == 0`。若 `a`、`b` 是字符串指针，直接写 `a == b` 比较的是地址。`strcmp` 区分大小写，从两串开头依次对照字节，把它们解释为 `unsigned char`。首对不同字节决定排序；一串恰好是另一串的前缀时，先遇到 `\0` 的短串排在前面。[WG14 C11 草案 §7.24.4、§7.24.4.2](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf)

| 判断 | 含义 |
| --- | --- |
| `strcmp(a, b) == 0` | 内容相同，直到两边同时到达终止符。 |
| `strcmp(a, b) < 0` | `a` 在该字节顺序中排在 `b` 前面。 |
| `strcmp(a, b) > 0` | `a` 排在 `b` 后面。 |

课堂把 `str1` 和 `str2` 都设为 `"Hello, World!"`，把 `str3` 设为 `"Hello, Guys!"`。03:02 控制台先报告前两串相等，再报告 `str1` 和 `str3` 不相等，进程退出码为 `0`。这里程序只判断返回值是否为零，没有打印具体差值。

讲师随后在画面手写 `hello` 与 `hey` 来说明排序。前两字节相同，第三位才遇到 `l` 和 `y`；在课堂使用的 ASCII 编码中 `l` 更小，所以 `strcmp("hello", "hey") < 0`。把字节值相减有助于理解符号，标准只保证负、零、正这三类结果。排序时直接判断符号，不依赖恰好返回 `-1`、`1` 或某个具体差值。这种顺序也不会按中文拼音等本地语言规则排列。[Microsoft Learn：strcmp](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/strcmp-wcscmp-mbscmp?view=msvc-170)
