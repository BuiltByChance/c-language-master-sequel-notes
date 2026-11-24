# P39 用 fprintf 把一条记录写成文本

`fprintf(stream, format, ...)` 把已经在内存中的值格式化成文字，写到 `stream`。`printf` 把结果写到标准输出，`fprintf` 让程序指定输出流；反过来，要从文件内的文字解析出值，用 `fscanf_s`。判断时只需问一次“数据是流入程序变量，还是由程序写出去”。[Microsoft Learn：fprintf](https://learn.microsoft.com/zh-cn/cpp/c-runtime-library/reference/fprintf-fprintf-l-fwprintf-fwprintf-l?view=msvc-170)

讲师用 `fopen_s(&stream, path, "w+")` 打开桌面的 `frank.txt`，检查打开成功后，把四个独立变量写成四行。`"w+"` 会创建文件或清空旧内容，并允许读写；本次程序只用了写入。讲师口头提到 `fprintf_s`，画面上实际调用的是 `fprintf`。

```c
fprintf(stream, "ID: %d\n", id);
fprintf(stream, "Name: %s\n", name);
fprintf(stream, "Temperature: %.1f\n", temperature);
fprintf(stream, "Grade: %c\n", grade);
```

`%d` 写整数，`%s` 写以 `\0` 结尾的姓名，`%.1f` 把温度按小数点后一位输出，`%c` 写一个字符。字段名、冒号、空格和换行都是格式字符串的一部分。初次运行后，等级位置显示为问号，修改格式里的空格也没解决。源代码试图用双引号的 `"A"` 初始化单个 `char`；双引号构成一个字符串，给等级变量赋值应写 `char grade = 'A';`。[Microsoft Learn：C 字符常量](https://learn.microsoft.com/zh-cn/cpp/c-language/c-character-constants?view=msvc-170)

约 06:24，讲师改成单引号，运行后用记事本重新打开文件，看到以下内容。姓名口述为 Frank，变量和文件都实际拼作 `Frnak`。这份文件是四行可读文本，课堂没有写入结构体的内存表示。

```text
ID: 1
Name: Frnak
Temperature: 36.5
Grade: A
```

控制台的“数据写入完成”由程序自行打印。整理者补充，若要由程序判断是否可以显示成功提示，写入时检查每次 `fprintf` 的返回值是否非负，写完再检查 `fclose(stream)` 是否返回 0。
