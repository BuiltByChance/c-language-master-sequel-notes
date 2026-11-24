# P65 用 malloc 分配动态数组并释放

`main` 调用 `demo()`，对比两组各含五个 `int` 的数据。第一组写成 `int static_arr[5] = {1, 2, 3, 4, 5};`。它是固定长度的局部数组，名字中的 `static` 没有声明关键字的作用。第二组通过 `<stdlib.h>` 的 `malloc` 申请，局部指针 `dynamic_arr` 用来访问这块空间。

1. `malloc` 需要的参数是**字节数**。课堂用 `(int *)malloc(5 * sizeof(int))`。在 C 中，返回的 `void *` 可直接赋给对象指针；写成 `int *dynamic_arr = malloc(5 * sizeof *dynamic_arr);` 也可以，且元素大小随指针类型而定。
2. 在第一次写入前检查 `dynamic_arr == NULL`。课堂的失败分支用 `perror("动态数组分配失败")` 说明错误，再调用 `exit(EXIT_FAILURE)` 结束程序。这次分配成功，没有触发该分支。
3. `malloc` 不初始化元素。讲师在 `0` 至 `4` 范围内，先写入 `dynamic_arr[i] = (i + 1) * 10;`，再用 `%d` 打印。构建成功，同时有 `C4267` 警告：索引 `i` 是 `size_t`，计算值写入 `int` 可能缩窄。本例的五个结果都可用 `int` 表示。整理者补充：在固定五项的这个例子里，循环索引可直接用 `int`。
4. `demo` 使用完数据后，对原分配地址调用一次 `free(dynamic_arr)`。固定局部数组在离开函数时自动结束生存期；如果只让局部指针离开作用域，动态块并不会因此释放。释放后的块不得继续访问。[微软的 malloc 说明](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/malloc?view=msvc-170) · [微软的 free 说明](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/free?view=msvc-170)

课堂控制台依次显示 `1 2 3 4 5` 和 `10 20 30 40 50`，进程退出码为 0。讲师口头说两种方法的输出“一样”，实际相同的是都能用下标遍历五项，两行的元素值不同。本次元素个数写死为 5，没有从输入取得长度，也没有扩容。
