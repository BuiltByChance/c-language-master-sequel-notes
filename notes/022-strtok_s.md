# P22 strtok_s

课堂用 Microsoft CRT 的三参数 `strtok_s` 把一句话按空格分成单词。每次调用只取下一段，并把该段结尾的分隔符换成 `\0`。返回的指针指向原数组中的字符，所以输入必须可写；若还要保留完整原句，应先复制到另一个可写数组，再拆副本。[Microsoft Learn：strtok_s](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/strtok-s-strtok-s-l-wcstok-s-wcstok-s-l-mbstok-s-mbstok-s-l?view=msvc-170)

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    char str[] = "This is a sample string";
    const char *delim = " ";
    char *context = NULL;
    char *token = strtok_s(str, delim, &context);

    while (token != NULL) {
        printf("Token: %s\n", token);
        token = strtok_s(NULL, delim, &context);
    }
    return 0;
}
```

- 第一次传 `str`，从数组开头找单词。得到 `This` 后，`context` 保存了下次应当从哪里继续。传 `&context` 让函数能修改调用者的指针变量。
- 后续第一个参数传 `NULL`，第三个参数仍是同一个 `&context`。这次调用会取 `is`。当返回值本身为 `NULL` 时，所有单词才算取完。
- `delim` 表示分隔字符的集合。本例只有一个普通空格，连续的分隔符会被跳过，不会返回空单词。若一份数据要求保留空字段，就需要能保留它们的解析方式。

讲师口头读作 `simple` 的词，源码与控制台都拼作 `sample`。05:48 课堂运行依次输出五行 `Token:`，后面分别是 `This`、`is`、`a`、`sample`、`string`。讲师提到同时拆多个字符串时保存位置的问题，本节只运行了一次单串拆分。若需交替拆两串，每串各用一份可写数组和一个 `context`。移植时查目标库的声明，C11 可选附录 K 的同名函数另有一项长度参数，与课堂三参数接口不同。[WG14 C11 草案 K.3.7.3.1](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf)
