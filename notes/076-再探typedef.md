# P76 再探 typedef

`typedef` 给类型起另一个名字。区分别名和变量，可以看声明是否有 `typedef`：`typedef unsigned long ulong;` 引入类型名 `ulong`，随后 `ulong number = 123456789;` 才创建名为 `number` 的对象。日后在同一作用域内写 `ulong`，它仍遵循 `unsigned long` 的取值范围和运算规则。[微软 C 文档：typedef 声明](https://learn.microsoft.com/zh-cn/cpp/c-language/typedef-declarations?view=msvc-170)

课堂把上述例子改为：

```c
typedef uint32_t u32;
u32 number = 1;
u32 n = 1;
```

`u32` 是讲师选择的名字，固定 32 位来自底层的 `uint32_t`。在提供这一精确宽度类型的实现中，可包含 `<stdint.h>` 取得它；课上已有的 `<inttypes.h>` 也会包含相应声明。`unsigned long` 的宽度随实现而异，不能仅凭名字 `ulong` 推断它固定为 32 位。[C11 工作草案 §7.20.1.1：精确宽度整数类型](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=269)

同一规则也适用于结构体。讲师在文件作用域短暂写出 `typedef struct { int x; int y; } Point;`。末尾的 `Point` 给这个含两个整型字段的结构体命名，以后就能用 `Point` 声明坐标变量。课堂没有创建坐标实例。

两个整型别名的演示写在 `main` 内，只在相应块作用域中可见。若其他源文件也要使用 `u32`，可将别名及其所需头文件放在公共头文件里，让各使用者包含它。讲师打开了项目的 Header Files 分组并口头说明这个做法，本节未创建新头文件，也未构建或运行程序。
