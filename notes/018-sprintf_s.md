# P18 sprintf_s

`sprintf_s` 把数字等数据按格式写入字符数组，适合先生成一段文本，稍后再显示或保存。第一项是可写数组，第二项是数组容量（包括末尾 `\0` 的位置），然后依次给出格式字符串和其中各占位符对应的值。下面沿用课堂的数值，改用能识别空字符串成功情形的判断。[Microsoft Learn：sprintf_s](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/sprintf-s-sprintf-s-l-swprintf-s-swprintf-s-l?view=msvc-170)

```c
#include <stdio.h>

int main(void) {
    char buffer[50];
    int number = 3;
    double pi = 3.14159;
    int ret = sprintf_s(buffer, sizeof buffer, "Integer: %d, Double: %.2f", number, pi);
    if (ret < 0) {
        return 1;
    }
    printf("Formatted string: %s\n", buffer);
    return 0;
}
```

`%d` 把整数转换为十进制文本，`%.2f` 把浮点数保留两位小数。函数返回写入的字节数，不含 `\0`。课堂用 `ret > 0` 判断，此例确有内容，但空结果也可能成功，此时返回 `0`。`printf` 才负责把数组里的文字送到控制台。08:24 的实际输出为 `Formatted string: Integer: 3, Double: 3.14`，编辑器里用于讲解的注释曾将小数写成 `1.14`。

讲师随后加长了格式字符串，使完整结果超过数组容量。Debug 弹出 `Buffer too small` 断言；Release 没有打印程序自写的错误提示，进程以 `-1073740791` 退出。微软运行库在这种情况下会清空有效目标并调用无效参数处理函数，只有程序获准继续时才有机会检查返回值。课末恢复了短格式，本节仅回顾自定义回调，没有重新注册或验证。 [P14 回调与退出状态](014-选修(难、非入门)：VS编译器捕获字符串异常.md)

讲师把 `_s` 接口概括为 C11 以后通用。C11 将这组接口列为可选的附录 K，同时仍保留 `sprintf`。换到另一套 C 运行库时，应查它是否提供 `sprintf_s`。若用标准 `snprintf`，检查返回值是否非负并小于目标容量，才能确认完整结果已写入；返回值达到或超过容量表示发生截断。[WG14 C11 草案 §7.21.6.5、附录 K](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf)
