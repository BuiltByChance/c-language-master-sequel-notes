# P110 小仓库：用结构体数组添加商品和更新库存

库存程序要区分三个数量。`MAX_ITEMS = 100` 是数组预留的**商品记录容量**，`item_count` 是已经写入的**记录条数**，每个 `Item` 的 `quantity` 才是这一种商品的**库存件数**。记录还包含 `int32_t id`、`char name[50]` 和 `float price`。查找、打印都只遍历 `[0, item_count)`。

| 操作 | 为什么这么传参 | 成功判断 |
| --- | --- | --- |
| `add_item(inventory, &item_count, id, name, quantity, price)` | 既写数组又要修改调用者的条数，因此传计数地址。条数已到 100 时返回 0；否则写入 `inventory[*item_count]` 后执行 `(*item_count)++`。 | 返回 1，新记录位于原来计数对应的下标。 |
| `update_quantity(inventory, item_count, id, new_quantity)` | 按 ID 遍历已有记录，只改匹配记录的件数，传入计数的值即可。 | 更新第一条匹配记录后返回 1，找不到返回 0。课堂没有拒绝重复 ID 的逻辑。 |
| `print_inventory(inventory, item_count)` | 逐条打印 ID、名称、件数和两位小数的价格。讲师口头要求只读；换成 `Item` 别名后，片尾实际形参是 `Item inventory[]`。 | 整理时可在声明和定义中都写成 `const Item inventory[]`，让编译器约束这个函数对记录的写入。 |

名称复制用微软的 `strncpy_s`，第二参数为目标数组总容量 50，第四参数为 `_TRUNCATE`。它能保留最多 49 个数据字节并补终止符。若名称被截短，返回 `STRUNCATE`；课堂没有检查这个返回值，名称必须完整的场景应在计数加一前处理截断。[Microsoft Learn 的 strncpy_s](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/strncpy-s-strncpy-s-l-wcsncpy-s-wcsncpy-s-l-mbsncpy-s-mbsncpy-s-l?view=msvc-170)

约 15:47，控制台两次打印同一批三条记录，进程退出码为 0。

| ID | 名称 | 初始件数 | 更新后件数 | 两次显示的单价 |
| ---: | --- | ---: | ---: | ---: |
| 1 | Laptop | 10 | 10 | 999.99 |
| 2 | Mouse | 50 | 45 | 19.99 |
| 3 | Keyboard | 30 | 30 | 59.99 |

调用为 `update_quantity(inventory, item_count, 2, 45)`。讲师一度说成改价格，随后更正，画面证实鼠标价格始终是 19.99。按 ID 删除商品、改名和改价格被留作练习；满仓与查无 ID 的分支也未现场运行。
