# P66 企业案例：用 realloc 扩充部门预算列表

课堂模拟公司从三个部门扩充为五个。原表用 `malloc` 申请了三个 `double`，金额依次为 `10000、15000、12000`。扩充时要保留原有金额，并给新增的第 4、5 个部门各写一笔预算。

**扩容时要保留原指针，先检查新结果。** `<stdlib.h>` 的 `realloc(budgets, newSize * sizeof *budgets)` 接收新表的**总字节数**，此处 `newSize` 为 5。在这个非零大小的调用中，有两种结果。[Microsoft Learn 的 realloc 说明](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/realloc?view=msvc-170)

- 返回 `NULL` 时，旧块和原来三项仍然有效。课堂程序选择结束，于是先 `free(budgets)` 再返回 `EXIT_FAILURE`。如果业务选择继续用三项旧表，可以保留它。用临时变量 `newBudgets` 接返回值，才不会在失败时遗失原指针。
- 返回非空时，立即用 `budgets = newBudgets;` 接管结果。分配器可能已经搬移数据，之后只通过这个返回地址访问整块。原三项得以保留，新增空间还没有初始化，先写 `budgets[3] = 20000;`、`budgets[4] = 18000;`，再以 `newSize` 打印。最终只 `free(budgets)` 一次；这时它保存的是成功扩容后的地址。

课堂先后排除了两种“看起来像预算没写进去”的显示问题。第一轮金额显示成字面文字 `$.2f`，因为格式里漏了 `%`，补成 `$%.2f` 后金额正常。部门序号用 `size_t i` 计算，课堂把 `%d` 改为 `%zd`；自己输出 `i + 1` 应配无符号的 `%zu`。扩容后的第一次打印仍把旧长度 `size` 传给 `printBudgets`，因此第二遍也只显示三项。

最后一次运行改为 `printBudgets(budgets, newSize)`，控制台先列原来的三个部门，再列五个。前三个金额未变，第 4、5 项分别为 `$20000.00`、`$18000.00`，进程退出码为 0。课堂没有触发申请失败分支，也没有打印地址，所以这次是否发生了搬移无法从输出判断。
