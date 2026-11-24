# P114 动态数组 Dynamic Array

一个能继续追加元素的整数数组，需要分别记住“已经放了几个”和“现在能放几个”。课堂把这两项和数据指针放在一个结构体里。

```c
typedef struct {
    int *data;
    size_t size;
    size_t capacity;
} Array;
```

`data` 指向连续存放的第一个整数，`size` 是已写入的元素数，`capacity` 是可容纳的元素数。二者都按元素计数，始终应有 `size <= capacity`。有效元素的下标满足 `0 <= i && i < size`，空数组没有可读元素。这里的 `int *` 每前进一步跨过一个整数；P112 的行指针 `int (*)[N]` 则跨过整行。

| 操作 | 如何维护数组状态 |
| --- | --- |
| 创建 | `create_array(4)` 先分配一个 `Array`，再为 4 个整数分配空间。第一步失败直接返回 `NULL`，第二步失败先释放外层对象再返回 `NULL`。成功时设置 `size=0`、`capacity=4`。 |
| 追加，还有余位 | 用旧 `size` 作下标写入值，再把 `size` 加一。课中的 `arr->data[arr->size++] = value;` 把两步合在一个表达式里。 |
| 追加，容量已满 | 向 `realloc` 请求新**总**容量所需的字节数。用临时指针接收结果；申请正大小失败则返回 0，旧数据、元素数和容量仍可用。成功才替换 `data`、更新容量，然后写新元素并返回 1。 |
| 打印和释放 | `print_array` 循环到 `size` 为止，再用 `%zu` 打印个数和容量。`free_array` 先释放 `data` 所指的数据块，再释放外层 `Array`。 |

**第 5 项最能检验扩容是否正确。** 已存 1 至 4 时，`size=4`、`capacity=4`。添加 5 之前，程序先把容量增至 8，再写入下标 4，最后才有 `size=5`。继续写入 6 和 7 不再分配。22:21 控制台依次打印 `1 2 3 4 5 6 7`、元素个数 7、容量 8。相对初始容量增加了四个槽，其中一个仍空闲。

整理者补充：若把这个接口交给其他调用者使用，应明确初始容量的范围。可以要求初始容量至少为 1，或在零容量时首次追加专门分配 1 项。令 `limit = SIZE_MAX / sizeof *arr->data`，创建时须保证初始容量不超过 `limit`；采用翻倍策略时，超过 `limit/2` 就不能直接乘 2。每次检查 `append` 的返回值，失败后仍可打印或释放原数组。课堂的 `main` 只检查了创建结果，未触发分配或扩容失败。扩容若成功，应从新的 `data` 重新取得元素地址。[Microsoft Learn，realloc](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/realloc?view=msvc-170)
