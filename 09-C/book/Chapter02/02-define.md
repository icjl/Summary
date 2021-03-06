预处理
===

预处理指令以 `#` 开始 (前面可以为 space 或 tab)，通常独立一行，可用 `\` 换行，常用来定义宏常量。也可定义伪函数，通常以 `({... })` 组织多条语句，最后一条语句作为返回值 (无 return，注意分号)。

```c
#define test(x, y) ({ \
        int _z = x + y; \
        _z; })
int main(int argc, char* argv[]) {
    printf("%d\n", test(1, 2));
    return 0;
}

// 展开后
int main(int argc, char* argv[]) {
    printf("%d\n", ({ int _z = 1 + 2; _z; }));
    return 0;
}
```

### 可选性变量

`__VA_ARGS__` 标识符用来表示一组可选性自变量。

```c
#define println(format, ...) ({ \
        printf(format "\n", __VA_ARGS__); })

int main(int argc, char* argv[]) {
    println("%s, %d", "string", 1234);
    return 0;
}

// 展开后
int main(int argc, char* argv[]) {
    ({ printf("%s, %d" "\n", "string", 1234); });
    return 0;
}
```

### 字符串化运算符

单元运算符 `#` 将一个宏参数转换为字符串，且其会自动进行转义操作。。

```c
#define test(name) ({ \
        printf("%s\n", #name); })

int main(int argc, char* argv[]) {
    test(main);
    test("\"main");
    return EXIT_SUCCESS;
}

// 展开后
int main(int argc, char* argv[]) {
    ({ printf("%s\n", "main"); });
    ({ printf("%s\n", "\"\\\"main\""); });
    return 0;
}
```

### 粘贴记号运算符

二元运算符 `##` 将左和右操作数结合成一个记号。

```c
#define test(name, index) ({ \
        int i, len = sizeof(name ## index) / sizeof(int); \
        for (i = 0; i < len; i++) { \
            printf("%d\n", name ## index[i]); \
        }})

int main(int argc, char* argv[]) {
    int x1[] = { 1, 2, 3 };
    int x2[] = { 11, 22, 33, 44, 55 };
    test(x, 1);
    test(x, 2);
    return 0;
}

// 展开后
int main(int argc, char* argv[]) {
    int x1[] = { 1, 2, 3 };
    int x2[] = { 11, 22, 33, 44, 55 };
    ({ int i, len = sizeof(x1) / sizeof(int); for (i = 0; i < len; i++) { printf("%d\n", 
                                                                                 x1[i]); }});
    ({ int i, len = sizeof(x2) / sizeof(int); for (i = 0; i < len; i++) { printf("%d\n", 
                                                                                 x2[i]); }});
    return 0;
}
```

### 条件编译

可用 `#if... #elif... #else... #endif`、`#define`、`#undef` 进行条件编译。

```c
#define V1
#if defined(V1) || defined(V2)
    printf("Old\n");
#else
    printf("New\n");
#endif
#undef V1

// 展开后
int main(int argc, char* argv[]) {
    printf("Old\n");
    return 0;
}

// 也可用 #ifdef、#ifndef 代替 #if
#define V1
#ifdef V1
    printf("Old\n");
#else
    printf("New\n");
#endif
#undef A

// 展开后
int main(int argc, char* argv[]) {
    printf("Old\n");
    return 0;
}
```

### typeof

- 使用 GCC 扩展 typeof 可以获取参数类型。

```c
#define test(x) ({ \
        typeof(x) _x = (x); \
        _x += 1; \
        _x; \
        })
int main(int argc, char* argv[]) {
    float f = 0.5F;
    float f2 = test(f);
    printf("%f\n", f2);
    return EXIT_SUCCESS;
}
```

### 调试宏

- 使用 assert 宏进行函数参数和执行条件判断。

```c
#include <assert.h>
void test(int x) {
    assert(x > 0);
    printf("%d\n", x);
}

int main(int argc, char* argv[]) {
    test(-1);
    return EXIT_SUCCESS;
}

// 展开后
$ gcc -E main.c

void test(int x) {
    ((x > 0) ? (void) (0) : __assert_fail ("x > 0", "main.c", 16, __PRETTY_FUNCTION__));
    printf("%d\n", x);
}

// 如果 assert 条件表达式不为 true，则出错并终止进程

$ ./test
test: main.c:16: test: Assertion `x > 0' failed.
Aborted

// 在编译 Release 版本时，记得加上 -DNDEBUG 参数
$ gcc -E -DNDEBUG main.c
void test(int x) {
    ((void) (0));
    printf("%d\n", x);
}
```

- 其他常用的特殊常量。

1. `#error "message"` : 定义一个编译器错误信息。
2. `__DATE__ `: 编译日期字符串。
3. `__TIME__` : 编译时间字符串。
4. `__FILE__` : 当前源码文件名。
5. `__LINE__` : 当前源码行号。
6. `__func__` : 当前函数名称。
