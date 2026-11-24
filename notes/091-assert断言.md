# P91 assert 断言

`<assert.h>` 提供宏 `assert(表达式)`，用来验证程序执行到某处时理应成立的条件。断言启用时，表达式非零就继续往下走；表达式为零则报告失败。诊断中的**表达式、文件名、行号**能帮助找到哪条假设被违反。遇到用户输入不合要求、文件打不开等正常可能发生的情况，要用显式分支处理，才能在关闭断言的版本中仍然作出响应。[Microsoft Learn 的 assert 文档](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/assert-macro-assert-wassert?view=msvc-170)

## 一段代码为什么有两种结果

```c
#include <assert.h>
#include <stdio.h>

int main(void)
{
    int a = 5;
    assert(a == 5);
    a = 3;
    assert(a == 5);
    puts("Hello");
}
```

讲师预期第二条断言失败，但首次运行的 Release 配置实际打印了 `Hello`。随后在 Visual Studio 顶部配置框切到 Debug 再运行，第一条断言通过；第二条失败，控制台标出 `a == 5`、`run_app.c` 和行号。示例后来添加了注释，同一条失败断言出现在第 24 行。诊断列出的表达式是“这里应当等于 5”，当时 `a` 的实际值已经是 3。

控制标准断言的是**包含 `<assert.h>` 时有没有定义 `NDEBUG`**。定义了它，`assert` 不再求值。讲师约 04:46 在源文件开头加入 `#define NDEBUG`，04:54 保持 Debug 再跑也打印了 `Hello`。单个源文件中的定义只作用于相应翻译单元，需要统一设置时可在项目的预处理器定义中配置。判断条件不要包含必须执行的输入、赋值或资源获取；应先完成操作，再单独断言结果。

讲师还口头举了前置条件和后置条件的用途。例如，设计要求某个函数不改变一个状态值，进入前与返回前可以分别断言该值。若想检查可变参数数量，必须有独立的已知项数才能比较，`assert` 本身无法推断省略号后实际传了几个参数。
