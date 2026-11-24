# P9 Union 联合

要让同一个打印函数接收整数、浮点数或字符串，可以让每条记录保存两样东西：一份取三者之一的**数据**，以及一个说明该用哪种方式读取它的**标签**。数据用 `union`，标签用 `enum`，二者放进同一个 `struct`。

```c
typedef union {
    int int_value;
    float float_value;
    char *string_value;
} Data;

typedef enum { INT, FLOAT, STRING } DataType;

typedef struct {
    DataType type;
    Data data;
} TypedData;
```

`Data` 的成员共享存储空间，因此一个对象适合在同一时刻保存其中一种值。`string_value` 保存的是指向字符的指针。讲师说联合的大小等于最大成员，C 的保证是**足以容纳最大成员**，末尾还可以有填充。要得知当前编译环境的大小，用 `sizeof(Data)`。[WG14 C11 草案 §6.7.2.1 第 16、17 段](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=133)

## 写入和读取必须约定同一个成员

课堂创建了三条独立记录。内层花括号里的 `.成员名 = 值` 指定初始化哪个联合成员，外层的标签记录这一选择。

| 初始化记录 | `print_data` 检查 `typed_data->type` 后的操作 |
| --- | --- |
| `TypedData data1 = { INT, { .int_value = 10 } };` | 读 `typed_data->data.int_value`，用 `%d` 打印 |
| `TypedData data2 = { FLOAT, { .float_value = 3.14 } };` | 读 `typed_data->data.float_value`，用 `%f` 打印 |
| `TypedData data3 = { STRING, { .string_value = "Hello, Union!" } };` | 读 `typed_data->data.string_value`，用 `%s` 打印 |

函数参数是 `TypedData *typed_data`，所以从 `main` 调用时传 `&data1`、`&data2`、`&data3`。`switch` 只检查标签，程序负责让标签与实际写入的成员一致。每个分支完成打印后用 `break` 退出。

## 一次构建成功、运行失败的对照

约 09:20，首版构建成功，运行时却在整数 `10` 后多打出浮点数 `0.000000`，随后进程异常退出。回看片中源码，整数和浮点两个 `case` 都漏了 `break`，第一次调用就从整数分支落入了后面两个分支。用另一成员读取联合会按该成员的类型重新解释对象表示，这次的 `0.000000` 不能作为数值转换规则；此时也没有可交给 `%s` 的有效字符串指针。[WG14 C11 草案 §6.5.2.3 脚注 95](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=101)

补上 `break` 再运行，约 09:46 控制台每条记录只输出一行，进程退出码为 `0`：

```text
Integer: 10
Float: 3.140000
String: Hello, Union!
```
