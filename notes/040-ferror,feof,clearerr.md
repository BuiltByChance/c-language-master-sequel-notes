# P40 用 ferror、feof 和 clearerr 判断文件读取状态

`fgets` 返回 `NULL` 时，先问“为什么读不下去”。同一个返回值可能表示文件正常读完，也可能表示读取出错。打开阶段要单独判断，`fopen_s` 失败便没有可供检查的流，应立即处理并退出。

| 在已打开的流上调用 | 可以判断或改变什么 |
| --- | --- |
| `ferror(fp)` | 非零表示此前的读写操作设置了错误标志。它不会给出具体错误代码 |
| `feof(fp)` | 非零表示读取时已经遇到文件末尾。如果刚好取出最后一个字节，还未试图读取更多内容，标志可能仍为零 |
| `clearerr(fp)` | 清除该流的错误和文件尾标志，文件位置不变。再次尝试读取前仍需处理原来的错误原因 |

课堂以 `"r"` 打开 `frank.txt`，准备 `char buffer[100]`。循环条件写成 `while (fgets(buffer, sizeof buffer, fp) != NULL)`，只有本次确实读到内容，才用 `printf("%s", buffer)` 输出。这样不会在末尾额外处理一次旧缓冲区。用于控制循环的是实际读取结果，`feof` 用来解释循环为何结束。[Microsoft Learn：feof](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/feof?view=msvc-170)

约 03:45，程序把 `printf` 错拼为 `pirntf`，链接失败。改正后，屏幕依次出现 `John 25`、`Jane 30` 和“已经到达文件的末尾”，退出码为 0。原文最后没有换行，所以末尾提示紧接 `Jane 30`。现场验证了正常读完这一条路径；发生读取错误以及随后调用 `clearerr` 的分支只写在代码中。

整理者补充，若要保留这次读取停止的原因，先检查两个标志，再决定是否需要清除。课堂代码在 `ferror` 分支先调用 `clearerr`，随后才检查 `feof`；如果进入过那个分支，第二次查询看到的已经是清除后的状态。直接关闭文件时不必先清标志。[Microsoft Learn：ferror](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/ferror?view=msvc-170) · [clearerr](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/clearerr?view=msvc-170)
