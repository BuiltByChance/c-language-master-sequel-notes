# P98 企业案例：应用逻辑模块 application_logic

`application_logic.h` 声明 `bool app_init(void)`、`bool app_execute(void)` 和 `void app_cleanup(void)`。前两项把是否成功交给上层决定，清理放在业务完成之后。讲师把它称作应用入口，本节写的是这三个供入口调用的函数，尚未展示由 `main` 把它们串起来运行。

| 阶段 | 函数内部的实际顺序 |
| --- | --- |
| 初始化 | `logger_init(NULL)` 选择标准输出，失败则直接向 `stderr` 写消息并返回 `false`。成功则把最低日志级别设为 `LOG_DEBUG`，记录 INFO 初始化消息，再返回 `true`。 |
| 执行业务 | 记 DEBUG 开始消息，调用 `safe_malloc(sizeof(int))`，检查分配结果。如果调用正常返回空指针，记错误并返回 `false`。成功时记 DEBUG，将整数设为 42，再以 `%d` 记录 INFO，释放内存，记 DEBUG 结束消息并返回 `true`。 |
| 清理 | **先**记录 INFO 清理消息，**再**调用 `logger_close()`。顺序颠倒会使末条消息被未初始化检查跳过。 |

虽然实现文件包含了 `pointer_safety.h`，这里实际用的是 `data == NULL` 和普通解引用，没有测试上一节的 `SAFE_DEREF` 宏。另需留意，上游 `safe_malloc` 在启用断言时遇到失败会先中止，业务层的 `false` 分支只有在分配函数确实返回后才有机会执行。

## 让分配和释放时的类型一致

课堂把 `int *data` 的地址强制转换成 `void **` 交给 `safe_free`，让释放函数修改这个指针变量。C 支持对象指针和 `void *` 间的转换，但这项规则不会让 `int **` 成为可互换的 `void **`。下面是整理者修订的成功路径：由一个真正的 `void *` 持有分配块，临时使用 `int *` 读写它，释放后不再使用该别名。[C11 草案 §6.3.2.3](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=73)

```c
void *block = safe_malloc(sizeof(int));
if (block == NULL) {
    log_error("Failed to allocate memory for data.");
    return false;
}
int *data = block;
*data = 42;
log_message(LOG_INFO, "Data value set to %d.", *data);
safe_free(&block);
```

片尾停在应用逻辑源码。今后联调时，初始化成功后可用 `Data value set to 42.` 和随后的结束、清理消息核对成功路径。本节没有构建或运行，没有生成日志文件或触发故障分支。
