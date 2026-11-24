# P37 用 ftell、fseek 和 rewind 查询、改变文件位置

`FILE *` 标识一次打开的流，流中的**文件位置**决定下一次从哪里读写。`long position = ftell(file_ptr);` 取得调用那一刻的位置。之后移动流，变量 `position` 仍保存旧值，必须再次查询才能观察移动结果。[Microsoft Learn：ftell](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/ftell-ftelli64?view=msvc-170)

讲师用 `"r+"` 打开已有的 `frank.txt`。这个模式要求文件存在，允许读写并保留原有内容。现场做了读取和定位，未执行写入。修正显示代码后，控制台给出以下顺序。

| 步骤 | 操作 | 现场读数 |
| --- | --- | --- |
| 1 | 打开文件，用 `ftell(file_ptr)` 查询 | `0` |
| 2 | `fgets(buffer, sizeof buffer, file_ptr)` 读取首行 `Hi`，再次查询 | `4` |
| 3 | `fseek(file_ptr, 0L, SEEK_SET)` 移到相对文件开头偏移 0 的位置，再查询 | `0` |
| 4 | `rewind(file_ptr)` 后再查询 | `0`。调用前已在开头 |

第一次运行的最后两个数字都是 `4`。画面显示 `fseek` 和 `rewind` 都已经调用，两处 `printf` 却仍然使用第二步保存的 `position`。约 08:26，把两处输出改为重新调用 `ftell(file_ptr)` 后，得到 `0 → 4 → 0 → 0`，程序正常结束。

`Hi` 可见的两个字母后面还有行结束符。记事本显示这份文件使用 Windows 的 CRLF，`H`、`i`、回车、换行在这里对应开头四个字节。文本模式可以转换换行，不能把所有文本流的 `ftell` 差值当成字符数。要回到读过的位置，可保存同一流的有效 `ftell` 值 `p`，再用 `fseek(file_ptr, p, SEEK_SET)` 恢复。[Microsoft Learn：文本模式中的 fseek](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/fseek-fseeki64?view=msvc-170)

**如何选函数**　只想知道位置，用 `ftell`，失败时返回 `-1L`。需要定位并检查是否成功，用 `fseek`，返回 0 表示成功；`SEEK_SET`、`SEEK_CUR`、`SEEK_END` 分别以开头、当前位置、末尾为参照。只想从头重读，可以用无返回值的 `rewind`，它还会清除流的错误和文件尾标志。[Microsoft Learn：rewind](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/rewind?view=msvc-170)
