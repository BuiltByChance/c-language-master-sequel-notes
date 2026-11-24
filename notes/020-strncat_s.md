# P20 strncat_s

`strncat_s` 保留目标原有的字符串，从它的结束处开始追加源字符串的前一段。在 Microsoft CRT 中，第二个参数填目标缓冲区的总容量，第四个参数填最多追加多少个数据字节。调用前目标就要含有 `\0`。如果源比上限短，只追加到源的第一个 `\0` 为止。[Microsoft Learn：strncat_s](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/strncat-s-strncat-s-l-wcsncat-s-wcsncat-s-l-mbsncat-s-mbsncat-s-l?view=msvc-170)

讲师先写 `char dest[20] = "Hello";`，编辑器在调用处标出警告。随后改成把数组清零，再用 `strcpy_s` 写入 `"Hello"`。两种方式都能在数组中建立合法的初始字符串。下例保留第一种初始化方式，并沿用课堂的追加上限。

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    char dest[20] = "Hello";
    const char *src = ", World!\n";
    size_t max_append = 7;
    if (strncat_s(dest, sizeof dest, src, max_append) != 0) {
        return 1;
    }
    printf("Concatenated string: %s\n", dest);
    return 0;
}
```

课堂运行显示 `Concatenated string: Hello, World`。新增的 7 个字节是逗号、空格和 `World`，源中的感叹号与换行没有追加。目标原有 5 个字节，再加 7 个和新的 `\0`，合计占用 13 个位置，容量 20 足够。若换成别的源或上限，就用同样的算法判断完整结果能否装下。
