# P8 Enumeration 枚举

星期、职位等级、错误码都有一个共同点：程序需要从一组相关选项中取一个值。C 的 `enum` 把这些选项写在同一份定义里，代码中便可用有意义的名字选择或比较它们。讲师用一周七天举例。

```c
typedef enum {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY,
    FRIDAY, SATURDAY, SUNDAY
} Weekday;

Weekday day = FRIDAY;
```

`Weekday` 是类型别名，`day` 是变量，`FRIDAY` 是枚举常量。未显式赋值时，第一项从 `0` 开始，后面的项依次比前一项大一。本例的 `FRIDAY`、`SATURDAY`、`SUNDAY` 因而分别是 `4`、`5`、`6`。[WG14 C11 草案 §6.7.2.2](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=136)

**输出编号。** 约 02:25，讲师说枚举不能“这样输出”。画面中被标红的是 `printf("%d\n", Weekday);`：这个位置需要一个值，写的却是类型名。随后他在 Visual Studio 中分别打印变量 `day`、常量 `FRIDAY`、常量 `SATURDAY`，三次实际结果为 `4`、`4`、`5`。

**输出名称。** `%d` 打印编号，不会从 `4` 自动找回文字 `FRIDAY`。讲师另写 `const char *get_weekday_name(Weekday day)`，用 `switch (day)` 把 `MONDAY` 映射到字符串 `"MONDAY"`。只写了一个分支，`default` 仅有 `break`，随后移除了该函数；课中没有调用它或验证字符串输出。整理者补充：若要把它用于显示星期，给全部七个枚举值各写一个返回词条，再让默认分支返回 `"UNKNOWN"`，调用者便可用 `%s` 打印结果。
