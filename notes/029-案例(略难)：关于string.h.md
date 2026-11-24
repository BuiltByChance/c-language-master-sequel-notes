# P29 案例（略难）——用 string.h 替换和统计文本

原文是 `This is a simple test. This text is for testing.`。程序先把其中的 `test` 全部替换成 `example`，随后三项统计都读取替换后的 `replacedText`。约 30:52，控制台给出了这四项结果，进程退出码为 0。

| 操作 | 本次运行的结果 |
| --- | --- |
| 替换子串 | `This is a simple example. This text is for exampleing.` |
| 统计小写 `s` | 5 个。两个 `This`、两个 `is` 和一个 `simple` 各贡献一个 |
| 统计单词 | 10 个，重复出现也计数 |
| 按首次出现顺序去重 | `This`、`is`、`a`、`simple`、`example`、`text`、`for`、`exampleing`，共 8 个 |

## 怎样替换并保留原文其余部分

`replaceWord` 用 `pos` 指向尚未处理的文本，从空的结果缓冲区开始。每次用 `temp = strstr(pos, oldWord)` 寻找下一处匹配。若找到，先用 `strncat_s` 追加 `pos` 到 `temp` 之前的 `temp - pos` 个字节，再用 `strcat_s` 追加新词，把 `pos` 推进到 `temp + strlen(oldWord)`。查找返回 `NULL` 时，最后追加 `pos` 起的剩余文本，才得到完整结果。第一次写成了 `temp == strstr(...)`，仅做比较，没有把匹配位置存入 `temp`；现场改为赋值后才得到预期的替换。

`strstr` 查找区分大小写的连续子串，所以 `testing` 也命中，留下尾部 `ing`。如果需求是“只替换独立单词”，需要额外判断命中位置的左右边界。整理者补充，复用这套循环时应先排除空的旧词，否则每次都在原位置命中，游标不会前进。[Microsoft Learn：strstr](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/strstr-wcsstr-mbsstr-mbsstr-l?view=msvc-170)

## 怎样计数和去重

- `countChar` 逐字节扫描，遇到小写 `'s'` 加一，遇到 `\0` 结束。这里所有输入都是英文 ASCII 文本。
- `countWords` 先把 `replacedText` 复制到可写数组，用微软三参数 `strtok_s` 拆词。画面的 `DELIMS` 是 `" ,.!?\n"`，包含空格、逗号、句点、感叹号、问号和换行。用 `char *context = NULL;` 保存进度，首次调用 `strtok_s(copy, DELIMS, &context)`，后续传 `strtok_s(NULL, DELIMS, &context)`，每取到一个非空指针便加一。分词会改写副本，原来的替换结果仍能交给下一步。课堂一度把 `&context` 当成字符数组交给复制函数，出现参数类型警告，那次只打印了替换文本与字符数，退出码为 3。改为直接初始化指针后完成了统计。[Microsoft Learn：strtok_s](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/strtok-s-strtok-s-l-wcstok-s-wcstok-s-l-mbstok-s-mbstok-s-l?view=msvc-170)
- `extractUniqueWords` 用另一份可写副本重新分词。对每个词遍历已收录的行，`strcmp(token, uniqueWords[j]) == 0` 表示重复，可结束本轮查找。全部行都不相等才把词复制到下一行，并递增实际词数。这样第二个 `This` 被跳过，`testing` 替换产生的 `exampleing` 则单列一项。

**更换输入时的容量判断（整理者补充）**　本例最终把 `TEXT_SIZE` 定为 100、`WORD_SIZE` 定为 50。替换结果含结尾的 `\0` 必须放进 100 字节；替换词比原词长时，每次匹配都会增加所需空间。词表有 100 行，每行只能保存 49 个数据字节和一个终止符。写入新行前核对剩余行数和词长，检查复制、追加函数的返回值，成功后再把新行计入词数。
