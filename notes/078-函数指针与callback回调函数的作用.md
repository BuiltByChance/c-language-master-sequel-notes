# P78 函数指针与 callback 回调函数的作用

如果三种处理都要按顺序访问数组每个元素，可把循环保留在一个函数中，把“拿到当前元素后做什么”交给调用者。调用者把处理函数的指针传进来，遍历函数在循环内调用它。这个被传入并由接收方调用的处理函数就是回调。

```c
typedef void (*Callback)(int);

void traverse(int *array, int size, Callback callback) {
    for (int i = 0; i < size; ++i) {
        callback(array[i]);
    }
}
```

`Callback` 是类型名，表示目标函数接收一个 `int`、没有返回值。`callback` 才是本次调用持有的指针变量。如果去掉 `typedef`，同样形状的声明只声明一个具名指针变量；口头的“只能用一次”应理解为“一次只保存一个目标”，该指针可以多次调用或重新赋值给其他签名兼容的函数。[C11 工作草案 §6.5.2.2：函数调用](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=99)

开头比较两种声明时，编辑器里短暂把返回 `int` 的 `add(int)` 赋给要求返回 `void` 的指针。这两个签名不兼容，示意代码未执行。正式示例使用的三个处理函数都符合 `void (int)`。

课堂数组是 `int numbers[] = {1, 2, 3, 4, 5};`。`main` 能用 `sizeof numbers / sizeof numbers[0]` 求出 5，再把数组首地址、长度和处理函数传入。到了 `traverse` 中，形参 `array` 只保存首地址，所以循环边界依赖调用者给出的 `size`。

| 第三个实参 | 本次遍历做什么 | 实际看到的结果 |
| --- | --- | --- |
| `printElement` | 逐项打印 `element`。 | 1、2、3、4、5。 |
| `doubleElement` | 打印原值和 `element * 2`。 | 翻倍值依次是 2、4、6、8、10。 |
| `checkIfEven` | 检查每项的奇偶性并打印。 | 1 奇、2 偶、3 奇、4 偶、5 奇。 |

例如调用 `traverse(numbers, size, doubleElement);` 时，传的是函数名。循环到第三项才执行等价于 `doubleElement(3)` 的调用。参数按值传递，打印 6 不会把原数组的 3 改掉；后一次奇偶判断仍看到原来的 3。

约 12:09，控制台按表中顺序显示三组共 15 行，进程退出码为 0。这个回调在当前线程的循环中立即执行。最后讲到的复制数组、排序和元素加一只是应用设想，课堂没有实现这些操作。
