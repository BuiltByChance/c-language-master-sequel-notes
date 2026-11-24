# P24 strncmp

只需判断两个字符串的前缀是否一致时，在 `<string.h>` 中使用 `strncmp(a, b, n)`。它最多比较前 `n` 个字节，遇到不同字节或字符串终止符就能确定结果。`== 0` 表示本次检查范围一致，`< 0` 或 `> 0` 表示首个不同字节决定的顺序。要判断两串全文是否相同，应使用 `strcmp`。[Microsoft Learn：strncmp](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/strncmp-wcsncmp-mbsncmp-mbsncmp-l?view=msvc-170)

课堂中 `str1`、`str2` 都是 `"Hello, World!"`，`str3` 是 `"Hello, Guys!"`。第一次将上限设为 7，控制台却报告 `str1` 和 `str3` 不等。回看第二处调用，实际拼写仍为 `strcmp`，还多传了一个 `num`。这个调用不符合 `strcmp` 的原型，当次运行不能验证前 7 个字节是否相同。根据字符串本身，前 7 个字节都是 `Hello, `（含逗号与空格），第 8 个字节 `W` 和 `G` 才有差异。

讲师随后把上限改为 5，并把两处函数名都改成 `strncmp`。01:13 控制台报告两组都相等，进程退出码为 `0`。画面的旧注释仍写着“比较前七个字符”，实际以 `size_t num = 5;` 为准。这次验证的是三串都以 `Hello` 开头。
