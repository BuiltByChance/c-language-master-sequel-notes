# P89 可变参数函数（variadic function）

要求 2、3、4、5 的平均值，课堂调用 `average(4, 2, 3, 4, 5)`。第一个 4 是调用者承诺的**数据项数**，后四项才参与求和。声明 `double average(int num_args, ...)` 允许同一个函数接收不同数量的数。省略号本身没有附带每项类型或长度；要么像本例一样约定计数和统一类型，要么像 `printf` 一样用格式字符串说明该怎样解释每个实参。

## 逐项读取并计算

以下按课中例子整理，在循环前处理非正计数。原代码用无符号的 `size_t i` 和有符号的 `int num_args` 比较，把非正数判断留在返回时；负计数可能在循环条件中被转成很大的无符号数。提早返回就不会错误地开始取参。返回 0.0 是这个接口对空输入的约定。

```c
#include <stdarg.h>
#include <stdio.h>

static double average(int num_args, ...)
{
    if (num_args <= 0)
        return 0.0;

    double sum = 0.0;
    va_list args;
    va_start(args, num_args);
    for (int i = 0; i < num_args; ++i)
        sum += va_arg(args, int);
    va_end(args);
    return sum / num_args;
}

int main(void)
{
    printf("Ave 2,3,4,5 = %f\n", average(4, 2, 3, 4, 5));
}
```

`va_list` 保存读取进度。`va_start(args, num_args)` 的第二项是省略号前最后一个有名字的形参；决定循环次数的是函数自己的 `for`。每次 `va_arg(args, int)` 取出一个 `int` 并推进，不必手算地址或假设固定移动四个字节。`va_end` 结束此次访问。若同一批值要遍历两次，可以在遍历前用 `va_copy` 复制状态，并对两份状态分别调用 `va_end`。[Microsoft Learn 的可变参数宏文档](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/va-arg-va-copy-va-end-va-start?view=msvc-170)

调用前核对**计数和类型**。本函数约定后续各项提升后都是 `int`。若要支持浮点值，应另定统一协议，按 `double` 传入和读取；可变部分的 `float` 会提升为 `double`。读取超过实际传入的个数或用不匹配的类型读取，都不能得到可靠结果。

约 06:09，课堂程序打印 `Ave 2,3,4,5 = 3.500000`，退出码为 0。讲师还打开本机头文件查看宏定义，口头提到 `va_copy`、日志和格式化输出；本节实际运行的是四个整数的平均值。
