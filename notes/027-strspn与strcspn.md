# P27 strspn 与 strcspn

两个函数都从第一个字符串的开头连续计数，返回类型是 `size_t`，用 `%zu` 打印。选哪个取决于“什么时候停”：

| 函数 | 计入的字节 | 何时停止 |
| --- | --- | --- |
| `strspn(s, accept)` | 属于 `accept` 集合的字节 | 首次遇到集合外的字节，或走到串尾 |
| `strcspn(s, reject)` | 不属于 `reject` 集合的字节 | 首次遇到集合内的字节，或走到串尾 |

课堂用一个数字集合检查数字开头的字符串，`strspn` 先后得到 `5`、`6`，换成首字符不在集合中的输入后得到 `0`。讲师把它解释成比较两个串开头相同的部分，但第二个参数是字符集合，字符出现的顺序、次数均不影响成员资格。整理者例：`strspn("221x", "12")` 得到 `3`，因为两个 `2` 和一个 `1` 都在集合内。[Microsoft Learn：strspn](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/strspn-wcsspn-mbsspn-mbsspn-l?view=msvc-170)

课堂的 `strcspn("abcdef123456", "1234567890")` 返回 `6`，因为遇到第一个数字前有六个字母。把检查对象换成 `Hello, world`、拒绝集合换成逗号，结果为 `5`。若从开头到结尾都没有命中，结果便等于 `strlen(s)`。[Microsoft Learn：strcspn](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/strcspn-wcscspn-mbscspn-mbscspn-l?view=msvc-170)

## 检查文件名中是否出现指定字符

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    const char *input = "filename.txt";
    const char *invalid_chars = "/\\:*?\"<>|";
    size_t index = strcspn(input, invalid_chars);
    if (index < strlen(input)) {
        printf("Invalid character '%c' at index %zu\n", input[index], index);
    } else {
        puts("No listed character found.");
    }
    return 0;
}
```

课堂先把函数误拼成 `strscpn`，链接器报告找不到这个符号。改为 `strcspn` 后，`filename.txt` 得到 `Input is valid.`，改成含问号的名字又得到 `Input contains invalid characters.`。上例在命中时额外打印具体字符和下标，便于定位。这里验证的只是文件名中有没有列举的符号，Windows 还另有限制，例如保留设备名 `NUL.txt` 和结尾空格或句点。[Microsoft Learn：Windows 文件命名约定](https://learn.microsoft.com/zh-cn/windows/win32/fileio/naming-a-file)
