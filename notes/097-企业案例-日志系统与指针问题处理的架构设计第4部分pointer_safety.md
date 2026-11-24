# P97 企业案例：pointer_safety 与空指针检查

这一模块解决的是可选指针的读取：调用者已知参数是 `NULL` 或指向仍然有效的对象，空时希望记日志并使用默认值。课程标题还提到野指针和悬挂指针，但本节实现只比较 `ptr == NULL`。未经初始化的指针不能交给它检验，某个动态对象是否已经释放也无法从一次非空判断得知。要避免使用失效对象，应管理好持有者、借用者与释放时机。[C11 草案 §6.2.4](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=56)

- `pointer_safety.h` 声明 `bool is_nullptr(const void *ptr)`。`pointer_safety.c` 中，空值分支调用 `log_error("Detected a NULL pointer.")` 并返回 `true`，其他情况返回 `false`。
- 头文件的 `SAFE_DEREF(ptr, defaultValue)` 意图是先调用 `is_nullptr`。为空时通过逗号表达式先记录尝试解引用的错误，再取默认值；非空时读取 `*ptr`。因此在日志初始化且级别允许的前提下，空值路径会尝试记录两条错误。若目标只需一条记录，让判空函数只判断或移除宏中的重复记录即可。
- 函数式宏在使用它的文件里展开，调用者必须能看到 `log_error` 的声明。宏的非空分支会使 `ptr` 实参求值两次，因此应传一个已经保存好的、类型明确的指针变量。真正跨物理行定义宏时，反斜杠要紧接换行；使用同一物理行时应删除反斜杠。约 04:31，编辑器展示为一行自动折行，但宏名之后仍有反斜杠，这份最终写法没有在本节编译验证。[GCC 预处理器关于宏重复求值的说明](https://gcc.gnu.org/onlinedocs/cpp/Duplication-of-Side-Effects.html)

**整理者示例。** 对只读取整数的代码，一个内联函数就能使指针只求值一次。它复用课堂已带日志的判空函数，默认值 0 是否合适由调用者决定；若业务上要区分“没有值”和“值恰好是 0”，应单独传递是否成功的状态。

```c
static inline int read_int_or(const int *ptr, int fallback)
{
    return is_nullptr(ptr) ? fallback : *ptr;
}
```

令 `int score = 8;`，按该示例推算，`read_int_or(&score, 0)` 取到 8，`read_int_or(NULL, 0)` 取到 0。讲师在编辑器中写完了判空函数并回看头文件，未构建、调用宏或查看日志；本节没有野指针或悬挂指针的检测结果。
