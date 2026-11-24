# P51 怎样查阅 math.h

C 程序使用数学库时，在源文件中写 `#include <math.h>`，让编译器看到相应函数的声明以及头文件提供的类型和宏。字幕中的“max”对应画面中的 `math.h`。讲师在 Visual Studio 的 `.c` 文件里加入这行，打开本机头文件查看，再对照网页查资料；这一节没有执行数学函数或验证数值。

查某个函数时，可从两个入口判断：

- **当前工具链提供什么。** 在 Visual Studio 中打开 `math.h`，需要时沿它包含的文件继续查看。课堂的微软版本把数学声明放在 `corecrt_math.h`，部分额外常量由 `_USE_MATH_DEFINES` 控制。要了解函数用途、参数和返回值，在微软 C 运行时文档的函数目录中按名称查找，再核对所需头文件。[Microsoft Learn 数学和浮点函数目录](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/floating-point-support?view=msvc-170)
- **哪些写法属于标准 C。** 查 WG14 文档中 `<math.h>` 的相应条目。微软实现可以提供扩展，而搜索结果中的 `<cmath>` 是 C++ 的头文件；遇到 `std::` 或重载写法时，要回到 C 接口核对函数名和参数类型。[WG14 C11 草案 §7.12 数学库](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=246)
