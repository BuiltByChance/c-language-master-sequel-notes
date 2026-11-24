# P88 悬挂指针（dangling pointer）

**悬挂指针**曾经指向一个有效对象，但那个对象的生命周期已经结束。例如，堆内存已被释放，或指针原先指向的自动局部对象已随函数返回而失效。判断依据是对象的生命周期。不能靠地址看起来像以前的值，或仅凭一次非空检查，就证明对象仍然存在。[C11 草案 §6.2.4](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=56)

课堂同时提到三种容易混淆的状态。**空指针**是程序可以用 `ptr == NULL` 检查的明确状态。俗称的**野指针**常指没有初始化、没有建立有效指向的指针，不能靠检查它的值来补救遗漏的初始化。悬挂则有明确的先后关系：它起初有目标，后来目标失效。

## 在最后一次有效使用后释放

以下是整理者按课堂例子保留的有效操作。分配失败就提前返回，成功才存入和读取 10。若持有指针的代码还要继续运行，可以在释放后把这个指针置空。

```c
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    int *ptr = malloc(sizeof *ptr);
    if (ptr == NULL)
        return EXIT_FAILURE;

    *ptr = 10;
    printf("%d\n", *ptr);
    free(ptr);
    ptr = NULL;
    return EXIT_SUCCESS;
}
```

`free(ptr)` 结束动态对象的生命周期。随后给 `ptr` 赋 `NULL` 只更新这一个指针变量；其他函数若保存了同一对象的地址，那里不会自动置空。安排释放位置时，要找出最后一个合法使用者，所有借用者都用完再由持有者释放一次。[Microsoft Learn 的 free 文档](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/free?view=msvc-170)

**现场停在编辑器。** 讲师临时写了释放后的第二次读取。约 02:49，Visual Studio 显示两条 C6001 警告，分别提到 `*ptr` 和 `ptr`，状态栏为 0 个错误、2 个警告。随后他注释这次读取并加上 `ptr = NULL`，全程未运行，没有现场输出或崩溃结果。
