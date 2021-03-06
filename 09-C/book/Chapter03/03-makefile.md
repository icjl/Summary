Makefile
===

假设一个程序，源代码如下：

- main.c 文件。
```c
#include "mytool1.h"
#include "mytool2.h"

int main() {
    mytool1_print("hello");
    mytool2_print("hello");
    return 0;
}
```

- mytool1.h 文件。

```c
#ifndef _MYTOOL_1_H
#define _MYTOOL_1_H

void mytool1_print(char *print_str);

#endif
```

- mytool1.c 文件。

```c
#include "mytool1.h"

void mytool1_print(char *print_str) {
    printf("This is mytool1 print %s\n", print_str);
}
```

- mytool2.h 文件。

```c
#ifndef _MYTOOL_2_H
#define _MYTOOL_2_H

void mytool2_print(char *print_str);

#endif
```

- mytool2.c 文件。

```c
#include "mytool2.h"

void mytool2_print(char *print_str) {
    printf("This is mytool2 print %s\n", print_str);
}
```

- 编译方式一：

```
gcc -c main.c
gcc -c mytool1.c
gcc -c mytool2.c
gcc -o main main.o mytool1.o mytool2.o
```

该编译方式缺点在于，如果某个文件如 mytool1.c 有改动，所有文件需要重新编译。

- 编译方式二：

make 方式。执行make之前，要先编写 Makefile 文件。

```
main：main.o mytool1.o mytool2.o
gcc -o main main.o mytool1.o mytool2.o

main.o：main.c mytool1.h mytool2.h
gcc -c main.c

mytool1.o：mytool1.c mytool1.h
gcc -c mytool1.c

mytool2.o：mytool2.c mytool2.h
gcc -c mytool2.c
```

这样执行 make 即可，如果某个文件有改动，编译器都只会去编译与该文件相关的文件。

### 编写 Makefile

在 Makefile 中以 # 开始的行都是注释行，Makefile 中最重要的是描述文件的依赖关系。一般格式为：

```
target：components
TAB rule
```

第一行表示的是依赖关系，第二行是规则。如 `main：main.o mytool1.o mytool2.o` 表示目标 main 的依赖对象是 main.o mytool1.o mytool2.o，当依赖的对象修改后，要去执行规则行命令，即 gcc -o main main.o mytool1.o mytool2.o。注意规则行中的 TAB 表示 TAB 键。

- Makefile 有三个非常有用的变量。

```
$@  目标文件
$^  所有的依赖文件
$<  第一个依赖文件
```

如果使用上面三个变量，那么可以简化 Makefile：

```
# 简化后的 Makefile
main：main.o mytool1.o mytool2.o
gcc -o $@ $^
main.o：main.c mytool1.h mytool2.h
gcc -c $<
mytool1.o：mytool1.c mytool1.h
gcc -c $<
mytool2.o：mytool2.c mytool2.h
gcc -c $<
```

- Makefile 的一个缺省规则。

```
.c.o：
gcc -c $<
```

这个规则表示所有的 .o 文件都是依赖与相应的 .c 文件的。例如 mytool.o 依赖于 mytool.c 这样 Makefile 还可简化为：

```
# 再一次简化后的 Makefile
main：main.o mytool1.o mytool2.o
gcc -o $@ $^
.c.o：
gcc -c $<
```


