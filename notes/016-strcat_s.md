# P16 strcat_s

要在现有字符串末尾追加文字，先确认目标是可写数组，里面已有以 `\0` 结尾的字符串。Microsoft CRT 的 `strcat_s(dest, capacity, src)` 从 `dest` 原先的 `\0` 处开始写，追加 `src` 的内容，再写入新的 `\0`。源和目标不能重叠。**第二个参数填写从 `dest` 开始的目标缓冲区总容量**。[Microsoft Learn：strcat_s 的参数和返回值](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/strcat-s-wcscat-s-mbscat-s?view=msvc-170)

课堂在 `char dest[50]` 中放入 `"Hello"`，再追加 `", World!\n"`。这里要区分三个数：

| 数值 | 如何得到 | 用在哪里 |
| --- | --- | --- |
| 50 | 实际数组的 `sizeof dest` | 传给 `strcat_s` 的容量参数 |
| 44 | `50 - 5 - 1` | 保留末尾 `\0` 后，最多还能追加的字节数，可与 `strlen(src)` 比较 |
| 15 | `5 + 9 + 1` | 本次结果实际需要的空间，包含换行和末尾的 `\0` |

讲师口头把第二个参数解释为剩余容量，画面也确实算出了 `44` 并传给函数。07:02 控制台显示 `Hello, World!`，进程退出码为 `0`。这次完整结果只需 15 个字节，传入 44 也够用。换成一个完整结果需要 45 到 50 字节的例子，数组本来放得下，错误的参数却会让运行库判为容量不足。

按接口约定修正的写法如下。源文字已带换行，打印时无需再加一次。

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    char dest[50] = "Hello";
    const char *src = ", World!\n";

    if (strlen(src) > sizeof dest - strlen(dest) - 1) {
        return 1;
    }
    if (strcat_s(dest, sizeof dest, src) != 0) {
        return 1;
    }
    printf("%s", dest);
    return 0;
}
```
