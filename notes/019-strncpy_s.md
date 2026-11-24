# P19 strncpy_s

需要复制整串时可以用 `strcpy_s`；只需要开头一段时，Microsoft CRT 的 `strncpy_s` 可用第四个参数限制复制的数据字节数。它的四项参数依次是目标数组、目标的总容量、源字符串、最多要取的字节数。用普通数值作为上限时，实际复制量是源串长度和上限中的较小值，目标还要为 `\0` 多留一个位置。[Microsoft Learn：strncpy_s](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/strncpy-s-strncpy-s-l-wcsncpy-s-wcsncpy-s-l-mbsncpy-s-mbsncpy-s-l?view=msvc-170)

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    char dest[20];
    const char *src = "Hello, World!\n";
    size_t max_copy = 10;
    errno_t result = strncpy_s(dest, sizeof dest, src, max_copy);
    if (result == 0) {
        printf("Copied string: %s\n", dest);
    } else {
        printf("Error copying string.\n");
    }
    return 0;
}
```

04:57 控制台实际输出 `Copied string: Hello, Wor`。`Hello` 占 5 个字节，逗号和空格各占 1 个，`Wor` 再占 3 个，合计 10 个。终止符另占 `dest[10]`，所以这次实际需要 11 个位置。源里的换行没有被复制，控制台的换行来自 `printf`。本节只运行了成功分支。`result == 0` 可直接写在条件判断里，单独用变量接收返回值是为了查看和处理结果。
