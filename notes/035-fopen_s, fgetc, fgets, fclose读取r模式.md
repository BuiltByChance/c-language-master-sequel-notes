# P35 用 fopen_s 打开文件，用 fgets 和 fgetc 读取

课堂把同一个 `123.txt` 读了两遍。第一遍把文本分段放入数组，第二遍逐个字节输出。开始时屏幕只出现一份文本，因为两个循环共用同一个流，第一遍已经读到文件末尾。

## 先确定文件，再打开流

1. 在记事本新建一个测试文件，按行写入 `Hello`、`Hi`、`Exit` 并保存为 `123.txt`。讲师在桌面文件的右键“属性”中复制“位置”，在目录末尾补上文件名。Windows 路径放进 C 字符串时，一个反斜杠写成 `\\`，例如 `"C:\\work\\123.txt"`。若只写 `"123.txt"`，运行时从程序的当前工作目录查找。
2. 声明 `FILE *file_stream = NULL;`，再调用 `fopen_s(&file_stream, path, "r")`。`"r"` 请求以只读方式打开已存在的文件。`&file_stream` 让函数能把打开结果写回这个指针变量；另一种接口 `fopen(path, "r")` 直接返回 `FILE *`。两者都由本次调用的路径和模式确定要打开的文件。[Microsoft Learn：fopen_s](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/fopen-s-wfopen-s?view=msvc-170)
3. 先检查返回值，再读。讲师写的是 `if (err != 0 || file_stream == NULL)`，任一条件成立便报错退出。约 13:15，旧代码在错误路径下仍试图读取，Debug 运行库触发 `stream.valid()` 断言；约 17:31，补上判断后，相同问题得到 `Error opening file: No such file or directory`，退出码为 1。权限受限是讲师举的另一种可能，现场验证的是路径不存在。

## 选择读法并验证结果

下面是按课堂调用顺序整理的两段循环，假设文件已经成功打开。`fgets` 适合需要处理文本片段的场合，`fgetc` 适合逐个字节判断的场合。

```c
char buffer[256];
while (fgets(buffer, sizeof buffer, file_stream) != NULL) {
    printf("%s", buffer);
}

if (ferror(file_stream)) {
    perror("Error reading file");
    fclose(file_stream);
    return EXIT_FAILURE;
}

putchar('\n');
rewind(file_stream);
int ch;
while ((ch = fgetc(file_stream)) != EOF) {
    putchar(ch);
}

int read_failed = ferror(file_stream);
if (read_failed) {
    perror("Error reading file");
}
if (fclose(file_stream) != 0) {
    perror("Error closing file");
    return EXIT_FAILURE;
}
return read_failed ? EXIT_FAILURE : EXIT_SUCCESS;
```

- `fgets` 最多读 255 个字节，给 `\0` 留一格。读到换行时会保留它，所以 `printf("%s", buffer)` 不额外加换行。讲师说超过 256 的一行就读不出来；实际是一次装不下，余下部分可由下一次调用继续读。[Microsoft Learn：fgets](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/fgets-fgetws?view=msvc-170)
- `fgetc` 返回读到的字节值或 `EOF`，用 `int` 接收才能保留二者的区别。讲师单步读取时输出开头的 `H`。整理示例在循环后补查 `ferror`，用来区分读取错误和正常结束；所需头文件为 `<stdio.h>`、`<stdlib.h>`。[Microsoft Learn：fgetc](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/fgetc-fgetwc?view=msvc-170)
- `rewind` 把这个普通文件的流位置移回开头并清除流状态。约 26:30，加入它后两种读法都产生输出。原文件最后一行没有换行，第一份的 `Exit` 紧接第二份的 `Hello`，屏幕出现 `ExitHello`。讲师随后在两遍之间打印换行，以便分开看。[Microsoft Learn：rewind](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/rewind?view=msvc-170)

讲师还用 `memset(buffer, 0, sizeof buffer)` 清零数组。清零会改写数组字节，数组仍占原来的空间，文件位置也不受影响；第二遍 `fgetc` 根本不使用这个数组。课堂最后一次运行退出码为 0，但构建仍报告 `memset` 缺少声明的 C4013 警告。上例省掉了清零；确实要用 `memset` 时应包含 `<string.h>`。课堂写了 `fclose` 失败的处理分支，现场没有触发关闭失败。
