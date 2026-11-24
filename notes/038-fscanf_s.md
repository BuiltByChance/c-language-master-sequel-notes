# P38 用 fscanf_s 从文件读取格式化数据

`fscanf_s` 从指定流的当前位置取出文本，按格式转换，写入程序变量。反方向是 `fprintf_s`，把变量格式化后写进文件。讲师先运行微软示例，显示 `a-string`、`65000`、`3.141590`、`x`。口述一度怀疑它没有打开文件，但代码以 `"w+"` 打开 `fscanf.out`，先写数据，调用 `fseek` 回到开头，然后才读回。输入确实来自文件，只是文件内容由同一程序预先生成。[Microsoft Learn：fscanf_s 及示例](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/fscanf-s-fscanf-s-l-fwscanf-s-fwscanf-s-l?view=msvc-170)

## 从已有的文本读一条记录

课堂第二个例子把 `Hello 1234 5.67 Z` 存入桌面的 `frank.txt`，用 `fopen_s(&stream, path, "r")` 打开。以下是整理者按相同字段合并的写法，放在打开成功的分支内。它要求四项全部成功才使用变量。

```c
char word[81], letter;
long number;
float value;
int fields = fscanf_s(stream, "%80s %ld %f %c",
                      word, (unsigned)sizeof word,
                      &number, &value, &letter, 1u);
if (fields == 4) {
    printf("%s\n%ld\n%f\n%c\n", word, number, value, letter);
} else {
    fputs("Incomplete record\n", stderr);
}
fclose(stream);
```

`%80s` 最多取 80 个非空白字符，`word` 有 81 格，最后一格可放 `\0`。`%ld` 对应 `long *`，`%f` 对应 `float *`，这两项不传容量。`%c` 接收单个字符，微软接口还要求传目标容量 `1u`。如果业务要求连空格也原样读取，格式应只用 `%c`；本例需要跳过分隔符，故在 `%c` 前写空格，消费零个或多个空白字符。[Microsoft Learn：scanf_s 的大小参数](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/scanf-s-scanf-s-l-wscanf-s-wscanf-s-l?view=msvc-170)

## 为什么返回成功却没看到 Z

讲师分四次读取，每次只检查一个字段是否返回 `1`。初次运行前三行正常，第四行没有可见字母。检查文件可见 `5.67` 和 `Z` 中间有空格；单独调用 `fscanf_s(stream, "%c", &c, 1u)` 时读进的正是这个空格。约 08:56，改成 `" %c"` 后，控制台依次显示 `Hello`、`1234`、`5.670000`、`Z`，退出码为 0。

画面还保留了一处未改的 `"%lds"`。`%ld` 已经读出 `1234`，多余的字面 `s` 与接下来的空格不符时，本次调用仍返回已赋值的项数 `1`。后面的浮点读取可跳过该空格，所以最终输出看似正常。只读长整数时应写 `%ld`。各字段读取失败时，课堂代码只打印提示然后继续；上面的合并写法避免了在出错后打印尚未赋值的字段。
