# 指针与内存管理

## 指针基础

```c
#include <stdio.h>

int main() {
    int x = 42;
    int *p = &x;        // p 指向 x 的地址

    printf("值: %d\n", *p);      // 解引用
    printf("地址: %p\n", (void*)p);

    *p = 100;           // 通过指针修改值
    printf("修改后: %d\n", x);   // 输出 100
    return 0;
}
```

## 动态内存分配

```c
#include <stdlib.h>
#include <string.h>

char* create_string(const char *src) {
    char *buf = malloc(strlen(src) + 1);  // +1 for '\0'
    if (!buf) return NULL;                // 总是检查 NULL
    strcpy(buf, src);
    return buf;
}

int main() {
    char *s = create_string("Hello, C!");
    printf("%s\n", s);
    free(s);    // 记得释放！
    s = NULL;   // 悬空指针置 NULL
    return 0;
}
```

!!! danger "常见陷阱"
    - 忘记 `free()` → 内存泄漏  
    - `free()` 后继续使用 → 未定义行为  
    - 越界访问数组 → 缓冲区溢出

## 内存布局

| 区域 | 内容 |
|------|------|
| Stack | 局部变量、函数调用帧 |
| Heap | `malloc/free` 动态内存 |
| BSS | 未初始化全局变量 |
| Data | 已初始化全局/静态变量 |
| Text | 代码段（只读） |
