# P25 strchr 与 strrchr

在 `<string.h>` 中，`strchr(str, c)` 返回字符 `c` **第一次**出现处的指针，`strrchr(str, c)` 返回**最后一次**出现处的指针。两者都在以 `\0` 终止的字符串中搜索，找不到就返回 `NULL`。要取下标，先判断查找成功，再用命中指针减去串首地址。这个差是从零开始的下标，类型为 `ptrdiff_t`；加 `1` 才是“第几个字符”。[Microsoft Learn：查找首个字符](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/strchr-wcschr-mbschr-mbschr-l?view=msvc-170) · [WG14 C11 草案 §6.5.6、§7.24.5.5](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf)

课堂特意在 `world` 里插入额外的 `w`。下面保留最终字符串，把打印内容简化为从一开始数的位置。

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    const char *str = "Hello, worwlwd!";
    const char *first = strchr(str, 'w');
    const char *last = strrchr(str, 'w');
    if (first == NULL || last == NULL) {
        puts("No match");
        return 0;
    }
    printf("First: %td, last: %td\n", first - str + 1, last - str + 1);
    return 0;
}
```

第一个 `w` 的下标为 `7`，位置为 `8`。讲师原用 `%ld` 打印指针差，在本次 x64 配置中收到类型不匹配的警告，后来改用适配 `ptrdiff_t` 的 `%td`。此处 `str` 本身声明为 `const char *`，保存的是字符串首字符的地址。

加入末次查找后，两处一度都显示 `8`。讲师先将第二处误留的 `strchr` 改为 `strrchr`，又把计算位置时误用的首个命中指针换成末个命中指针，最后得到 `8` 和 `13`。若只改函数名，打印的表达式仍可能读出旧结果。
