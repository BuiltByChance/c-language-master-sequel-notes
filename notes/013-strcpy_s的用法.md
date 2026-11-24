# P13 strcpy_s 的用法

要把 `"hello"` 从源数组复制到另一个可写数组，目标至少需要六个 `char`：五个字母，加一个 `\0`。课堂的目标数组有 50 个元素。`strcpy_s(dest, 50, src)` 的第二项告诉 Microsoft C 运行库目标的可用容量；第三项才是源字符串。成功时连同结束符一起复制，返回 `0`。源和目标存储区须互不重叠。[Microsoft Learn：strcpy_s 的参数和返回值](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/strcpy-s-wcscpy-s-mbscpy-s?view=msvc-170)

**看清容量和长度。** 讲师先用 `char str[100] = "hello";` 说明，初始化后下标 `0` 至 `4` 是字母，下标 `5` 是字符串的结束符，其余元素也被初始化为零。若改用 `char str[] = "hello";`，数组恰有六个元素。两种写法的文本长度都为五。

**照着函数声明读一次调用。** 在 Visual Studio 中写 `#include <string.h>`，按住 `Ctrl` 单击函数名可查看声明里的目标、容量和源。整理者把课堂成功用例补上返回值检查，便于复习：

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    const char src[] = "hello";
    char dest[50];
    errno_t err = strcpy_s(dest, sizeof dest, src);
    if (err != 0) {
        return 1;
    }
    printf("src = %s\ndest = %s\n", src, dest);
    return 0;
}
```

此处 `dest` 是本地 `char` 数组，`sizeof dest` 给出完整的 50 字节。若把复制封装到接收 `char *dest` 的函数中，要另外传入调用者掌握的容量；函数内的 `sizeof dest` 只会得到指针大小。

**现场结果。** 约 04:20，普通 `strcpy` 在该项目的编译配置下触发 C4996，构建失败。改用带容量参数的 `strcpy_s` 后，约 06:18，源和目标都打印 `hello`。讲师保留 50 个元素的目标数组，把传给函数的容量改为 `4`，程序构建成功，运行时弹出 `Debug Assertion Failed`，其中写着 `Buffer is too small`。该次没有得到可用的复制结果。MSVC 会调用无效参数处理程序，只有在允许继续执行时才会返回 `ERANGE`。片尾讲师把数组长度和参数统一为宏 `SIZE`，查看了 `errno_t` 的定义，并开始用 `err` 接收返回值；这一步没有现场展示错误处理分支。[Microsoft Learn：容量不足时的行为](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/strcpy-s-wcscpy-s-mbscpy-s?view=msvc-170)
