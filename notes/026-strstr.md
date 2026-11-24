# P26 strstr

`strstr(text, sub)` 在原文中按大小写查找一段连续文字。两个参数都应指向以 `\0` 结尾的有效字符串。找到时，函数返回第一处匹配的首字节在原文中的地址；找不到时返回 `NULL`。因此先判空，再用 `result - text` 计算从零开始的字节下标。返回的指针仍指向原文，如果直接用 `%s` 打印它，会显示从匹配处一直到原文结尾的内容。[WG14 C11 草案 §7.24.5.7](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf)

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    const char *text = "This is a simple test simple string.";
    const char *sub = "simple";
    const char *result = strstr(text, sub);
    if (result != NULL) {
        printf("Found '%s' at position %td\n", sub, result - text + 1);
    } else {
        printf("'%s' not found\n", sub);
    }
    return 0;
}
```

课堂原文有两个 `simple`，实际输出第一个的位置为 `11`，即字节下标 `10`。讲师口头认为空的搜索串可能返回 `NULL`，没有现场运行该情形。接口规定 `strstr(text, "")` 返回 `text`，因此空搜索串在下标 `0` 处就匹配。片尾讲师在编辑器里把搜索串拉长，随后恢复 `simple`；视频没有展示这个未命中分支的运行结果。[Microsoft Learn：strstr 的返回值](https://learn.microsoft.com/en-us/cpp/c-runtime-library/reference/strstr-wcsstr-mbsstr-mbsstr-l?view=msvc-170)
