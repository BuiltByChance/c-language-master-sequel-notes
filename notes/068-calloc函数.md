# P68 用 calloc 创建初值为零的任务列表

示例中每个员工有名字 `char *name`、任务 ID 数组 `int *taskList` 和槽位数 `int taskCount`。课堂约定任务 ID 为 0 表示“尚未分配”。`create_employee("Frank", 5)` 给员工预留五个槽位；刚创建时，他已获分配的任务仍是零个。

`<stdlib.h>` 的 `calloc(taskCount, sizeof(int))` 用第一个参数指定元素个数，第二个参数指定单个元素的字节数。它把分配到的存储按字节清零，因此这里五个 `int` 初始值都是 0。`malloc` 不提供这项初始化保证。若以后让用户指定任务数，应先确认数量有效，并在访问第一个槽位前确认至少有一项。[Microsoft Learn 的 calloc 说明](https://learn.microsoft.com/en-us/cpp/c-runtime-library/reference/calloc?view=msvc-170)

创建和销毁时按已取得的资源处理：

1. 用 `malloc` 申请员工结构体。若失败，直接返回 `NULL`。
2. 为名字申请 `strlen(name) + 1` 字节。若失败，释放结构体后返回；成功则用 `strcpy_s` 复制名字。讲师片尾口头只提到 `stdio.h` 和 `stdlib.h`，实际源码还要靠 `<string.h>` 声明这两个字符串函数。
3. 保存 `taskCount` 并用 `calloc` 申请整数列表。若失败，释放已经成功分配的名字、结构体后再返回 `NULL`。`main` 检查创建结果后才访问字段。
4. 正常结束时，`free_employee` 先释放 `name`、再释放 `taskList`，最后 `free(employee)`。两个成员缓冲区各自独立；结构体必须留到取完这两个地址以后再释放。

讲师写 `developer->taskList[0] = 101;`，第一次运行输出 Frank 的任务 ID 为 `101`。第二次仅把打印的下标改成 `1`，输出 `0`，对照出已写过和未写过的槽位。其余三个槽位没有逐个打印，分配失败分支也没有在课堂触发。
