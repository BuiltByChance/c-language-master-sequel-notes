# P15 strlen

`strlen` 从字符串开头数到第一个 `\0`，返回它之前的字节数。数组的容量、结尾的 `\0` 以及后面的空余位置都不计入长度。传入前须能确认可读范围内有 `\0`。例如 UTF-8 汉字通常占多个字节，`strlen` 的返回值不能直接作为汉字个数。[Microsoft Learn：strlen](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/strlen-wcslen-mbslen-mbslen-l-mbstrlen-mbstrlen-l?view=msvc-170)

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    char dest[50] = "Hello";
    printf("%zu\n", strlen(dest));
    return 0;
}
```

`strlen(dest)` 为 `5`，`sizeof dest` 为 `50`。在 `<string.h>` 的声明中，`strlen` 返回无符号整数类型 `size_t`。课堂打印使用 `%zd`，上面按返回类型改用 `%zu`。[WG14 C11 草案 §7.21.6.1、§7.24.6.3](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf)

课堂第一次运行输出 `5`。Visual Studio 的错误列表同时显示 `C6001`，提示 `dest` 可能未经初始化。检查画面中的声明可见，`"Hello"` 已经初始化了数组，`dest[5]` 是 `\0`，剩余元素也都为零。讲师改成 `char dest[50] = {0};`，再用 `strcpy_s(dest, 50, "Hello");` 填入内容，编辑器随后显示 `No issues found`。片尾打开头文件查看 `strlen` 的声明，未展示修改后的第二次运行。直接用字符串字面量初始化字符数组是标准 C 允许的写法。遇到类似警告，先沿着赋值路径查明 `\0` 在哪里写入；若数据确实来自复制，应检查复制成功，再调用 `strlen`。[WG14 C11 草案 §6.7.9](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=158)
