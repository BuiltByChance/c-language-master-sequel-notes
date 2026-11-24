# P67 用 malloc 创建游戏角色，防止内存泄漏

课堂的 `Character` 有三个成员：`char *name`、`int level` 和 `int hp`。等级、生命值直接存放在结构体里。`name` 只存地址，名字的字符还需要另一块空间。`main` 中的 `create_character("Hero", 1, 100)` 因此涉及**两次分配**：一个角色对象，加上存放 `Hero` 和终止符 `\0` 的 5 字节。

`create_character` 要求传入有效、以 `\0` 结尾的名字。它先执行 `malloc(sizeof(Character))`，再为 `new_character->name` 申请 `strlen(name) + 1` 个字节。两者都成功，才用 `strcpy_s` 复制名字并写入等级和生命值。这样创建的角色拥有自己的名字副本，调用者无需延长原字符串的生存期。[微软的 malloc 说明](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/malloc?view=msvc-170)

沿每个返回点清点已经成功分配的资源，即可决定谁来释放：

| 返回前走到哪一步 | 已取得的资源 | 处理方式 |
| --- | --- | --- |
| 第一次 `malloc` 失败 | 无 | 打印错误并返回 `NULL`。 |
| 第二次 `malloc` 失败 | 只有角色结构体 | 先 `free(new_character)`，再返回 `NULL`。 |
| 成功返回角色 | 结构体和名字缓冲区 | 调用者使用完后调用 `free_character(hero)`。 |

`free_character` 先查 `character != NULL`，再顺序执行 `free(character->name);` 和 `free(character);`。顺序很重要，取得名字缓冲区的地址还需要访问有效的结构体。整理者补充：如果复制函数返回非零，应在创建函数中释放已取得的两块内存并返回失败；课堂未检查该返回值。[微软的 free 说明](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/free?view=msvc-170) · [微软的 strcpy_s 说明](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/strcpy-s-wcscpy-s-mbscpy-s?view=msvc-170)

讲师完成了创建函数、释放函数和 `main` 中的调用，失败时由 `main` 返回 `EXIT_FAILURE`。开头提到的 `calloc` 没有用到。画面未展示构建、运行、分配失败或泄漏检测结果，能核实的是源码中各条路径的清理顺序。
