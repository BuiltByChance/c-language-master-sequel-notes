# P2 初识结构体 struct

一条日期记录有日、月、年三个值。用 `struct` 可以给每个值取成员名、指定类型，再把它们作为一个整体交给程序处理。定义类型时写明每条记录有哪些成员；创建这个类型的变量时，才有一条具体记录。[Microsoft Learn：C 结构声明](https://learn.microsoft.com/zh-cn/cpp/c-language/structure-declarations?view=msvc-170)

## 读懂日期的声明和初始值

讲师在 Visual Studio 的 `main` 中写了下面这段代码。约 02:55，他把各行成员声明末尾误写的逗号改成了分号。

```c
struct Date {
    int day;
    int month;
    int year;
};

struct Date today = { 12, 2, 2024 };
```

`Date` 是结构体标签，例中没有为它定义别名，因此声明变量要写完整类型 `struct Date`。`today` 是变量名，`day`、`month`、`year` 是它所包含的三个成员。结束结构体定义的右花括号后也有一个分号。

读这种没有写成员名的初始化列表，要按成员的**声明顺序**从左向右配对。这里先写 `day`，所以 `12` 表示日，`2` 表示月，`2024` 表示年，合起来是 **2024 年 2 月 12 日**。换一个习惯的日期书写顺序，可能仍能通过类型检查，却会得到另一条日期记录。[WG14 草案 §6.7.9 第 17 段](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2433.pdf)

## 同一条记录可以容纳不同类型

讲师随后定义了 `struct Person`。依次声明的成员是 `char name[50]`、`int age`、`float height`。因此，`struct Person frank = { "frank", 77, 280 };` 会用字符串 `"frank"` 初始化姓名数组，年龄取 `77`，身高取 `280`。讲师口头举例说“两米八”，代码没有注明身高单位。

同一种结构体可以创建多个变量，每个变量有自己的一组成员值。约 07:15，讲师临时添加 `struct Person zhangsan = {};` 来说明可以再建一条人物记录，编辑器在空花括号处显示红线。片尾他已删除该行，保留 `today` 和 `frank`。本节展示到编辑源码为止，成员输出留待后讲，现场没有编译或运行结果。
