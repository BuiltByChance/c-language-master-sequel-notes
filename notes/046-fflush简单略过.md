# P46 用 fflush 刷新输出流

程序刚向日志写了一条关键消息，接下来还要继续运行，这时可以用 `fflush(log_file)` 把 C 运行库中尚未输出的数据交给底层文件，流保持打开。成功返回 0，失败返回 `EOF`。这能降低进程意外退出时丢失缓冲日志的风险；操作系统或设备仍可能缓存数据，断电后的持久性需要另外保证。[Microsoft Learn 的 fflush 文档](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/fflush?view=msvc-170)

课堂用 Visual Studio 中编辑文件、按 Ctrl+S 保存来作类比。约 02:58，讲师把打开 `frank.txt` 的模式从 `"r"` 临时改为追加模式 `"a"`，并写下 `fprintf(file_ptr, "Start");` 和 `fflush(file_ptr);`。片尾撤掉示例，打开模式又恢复为 `"r"`。没有实际运行写入或模拟崩溃，因而没有现场观察到文件内容变化。

整理者补充，实际使用时先确认已用允许写入的模式成功打开文件，再依次检查 `fprintf` 的返回值是否非负、`fflush` 的返回值是否为 0。程序还能继续向该流写下一条消息，最终用完时再检查 `fclose` 的结果。
