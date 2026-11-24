# P36 用 fputs、fputc 和 w 模式写文件

先决定是否保留文件原有内容。`fopen_s(&file_ptr, path, "w")` 成功打开现有文件时，立即把其长度截为零；文件不存在时会尝试创建。想保留旧内容并把新数据放在末尾，应选 `"a"`。课堂实际用了 `"w"`，只口头提到追加模式和“仅打开再关闭即可清空文件”。[Microsoft Learn：fopen_s 的打开模式](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/fopen-s-wfopen-s?view=msvc-170)

讲师先在桌面的练习文件 `frank.txt` 中保存了一段文字。程序声明 `FILE *file_ptr = NULL;`，以 `"w"` 打开该文件，若返回值非零或 `file_ptr == NULL`，用 `perror` 报告打开错误并退出。成功后，以下三个函数在同一次打开期间依次写入，前一个函数写完的位置就是后一个的起点。

| 要写的内容 | 课堂调用 | 怎样控制换行 |
| --- | --- | --- |
| 单个字符 | `fputc('H', file_ptr)`、`fputc('i', file_ptr)` | 再调用 `fputc('\n', file_ptr)`，第一行才结束 |
| 已经准备好的字符串 | `fputs("This is a line written by fputs.\n", file_ptr)` | `fputs` 写入字符串内容，末尾的 `\0` 不写入。这里的换行来自字符串中的 `\n` |
| 多个值按格式排列 | `fprintf_s(file_ptr, "Numbers: %d, %f, %s\n", 10, pi, "End of example.")` | 格式字符串决定字段的顺序、标点与换行，`%f` 默认保留六位小数 |

约 05:38，程序关闭文件后，讲师用记事本重新打开，看到三行，原来的文字已经消失。

```text
Hi
This is a line written by fputs.
Numbers: 10, 3.140000, End of example.
```

第一次编译出现 C4305，原因是 `float pi = 3.14;` 把 `double` 字面量转换成 `float`。讲师随后同时改为 `double pi = 3.14;` 和 `%lf`。对于这里的格式化**输出**，`%f` 也接收 `double`；改变变量的类型消除了初始化处的窄化转换。约 06:40，讲师在 `fclose` 前另加一次不带换行的 `fputs`，重跑并打开文件，看到原先三行各一份，第四行是新文本。[WG14 C11 草案 §7.21.6.1](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=330)

**怎样判断写入成功**　课堂的成功提示由程序无条件打印，证据来自随后重开文件。整理者补充，程序本身可检查 `fputc`、`fputs` 是否返回 `EOF`，`fprintf_s` 是否返回负值，最后检查 `fclose(file_ptr)` 是否返回 0。缓冲的数据有可能到关闭时才报告写入失败。[Microsoft Learn：fputs](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/fputs-fputws?view=msvc-170) · [fprintf_s](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/fprintf-s-fprintf-s-l-fwprintf-s-fwprintf-s-l?view=msvc-170)
