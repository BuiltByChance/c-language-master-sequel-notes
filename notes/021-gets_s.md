# P21 gets_s

从标准输入读取一整行时，`gets_s(buffer, capacity)` 会保留行内空格，读到换行后把换行换成 `\0`。在 Microsoft CRT 中，`char buffer[100]` 最多装 99 个数据字节和一个结束符。读取成功时函数返回 `buffer`，读取出错或未读到字符便到达文件末尾时返回 `NULL`。只按回车会成功读入空字符串。输入行过长会触发参数处理，程序不能将其当作正常截短的一行继续使用。[Microsoft Learn：gets_s](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/gets-s-getws-s?view=msvc-170)

```c
#include <stdio.h>

int main(void) {
    char buffer[100];
    printf("Enter a string: ");
    if (gets_s(buffer, sizeof buffer) == NULL) {
        puts("Error or end of file encountered.");
        return 1;
    }
    printf("you entered: %s\n", buffer);
    return 0;
}
```

课堂约 06:21 在提示后输入一串字符，下一行 `you entered:` 后显示相同内容。讲师后来暂时去掉返回值判断，普通输入也回显成功，片尾恢复了有判断的版本。本节没有实测空行、文件末尾或超长行。若要区别正常的文件末尾和读取错误，可在收到 `NULL` 后分别查看 `feof(stdin)` 和 `ferror(stdin)`。

只输出字符串并另起一行时，`puts(buffer)` 就足够；要加前缀时，用固定格式 `printf("you entered: %s\n", buffer)`。讲师先用 `puts` 打印 `Hello!`，又用同一固定文字作 `printf` 的格式串。这段文字恰好不含格式转换符；处理用户输入时应把它作为 `%s` 对应的数据参数。[Microsoft Learn：puts](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/puts-putws?view=msvc-170)

旧函数 `gets` 不接收目标容量，C11 已将它移除。03:24 画面里实际拼写为 `get()`，链接器报的未解析符号也是 `get`，这次构建没有测试旧 `gets`。`gets_s` 属于可选的 C11 附录 K；在缺少它的运行库中可用标准 `fgets`，成功读入的换行可能仍留在数组里。[WG14 C11 草案前言、§7.21.7.2 和附录 K](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf)
