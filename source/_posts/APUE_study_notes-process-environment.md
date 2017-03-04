title: APUE学习笔记——进程环境
date: 2016-04-07 23:35:06
tags: [Linux,APUE]
---
# 进程终止
1. 退出函数
```c
#include <stdlib.h>

void exit(int status);
void _Exit(int status);

#include <unistd.h>

void _exit(int status);
```

2. 函数`atexit`
```c
#include <stdlib.h>

int atexit(void (*func)(void));
```
其中，`atexit`的参数是一个函数地址，当调用此函数时无需向它传递任何参数，也不期望它返回一个值。`exit`调用这些函数的顺序与它们等级时候的顺序相反。同一函数如若登记多次也会被调用多次。

# 环境表
环境表也是一个字符指针数组，其中每个指针包含一个以`null`结束的C字符串的地址。全局变量`environ`则包含了该指针的数组的地址：
```c
extern char **environ;
```
我们称`environ`为`环境指针(environment pointer)`，指针数组为环境表，其中各指针指向的字符串为环境字符串。
通常用`getenv`和`putenv`函数来访问特定的环境变量，而不是`environ`变量。

# C程序的存储空间布局
- 正文段。
- 初始化数据段。
- 未初始化数据段。
- 栈。
- 堆。

# 存储空间分配
1. `malloc`，分配指定字节的存储区。此存储区中的初始值不确定。
2. `calloc`,为指定数量指定长度的对象分配存储空间。该空间中的每一位(bit)都初始化为0。
3. `realloc`,增加或减少以前分配区的长度。

```c
#include <stdlib.h>

void *malloc(size_t size);
void *calloc(size_t nobj, size_t size);
void *realloc(size_t nobj, size_t newsize);
/*
 * 3个函数返回值：若成功，返回非空指针；若出错，返回NULL
 */
void free(void *ptr);
```

# 环境变量
```c
#include <stdlib.h>

char **getenv(const char *name);
/* 返回值：指向与name关联的value的指针；若未找到，返回NULL */
```

`getenv`,可以用其取环境变量值

```c
#include <stdlib.h>

int putenv(char *str);
/* 函数返回值：若成功，返回0；若出错，返回非0 */
int setenv(const char *name, const char *value, int rewrite);
int unsetenv(const char *name);
/* 两个函数返回值：若成功，返回0；若出错，返回-1 */
```

- `putenv`取形式为`name=value`的字符串，将其放到环境中。
- `setenv`将`name`设置为`value`。如果在环境中`name`已存在,那么(a)若`rewrite`非0，则首先删除其现有的定义;(b)若`rewrite`为0，则不删除其现有定义(`name`不设置为新`value`,而且也不出错)。
- `unsetenv`删除`name`的定义。即使不存在这种定义与不出错。

# 函数`setjmp`和`longjmp`
```c
#include <setjmp.h>

int setjmp(jmp_buf env);
/* 返回值：若直接调用，返回0；若从longjmp返回，则为非0 */
void longjmp(jmp_buf env, int val);
```

在希望返回到的位置调用`setjmp`。当调用`longjmp`是，第一个参数是调用`setjmp`时的`env`;第二个参数是具有非0值的`val`,它将称为从`setjmp`处返回的值。
在课本的[例程7.13](https://github.com/kaiiak/APUE/blob/master/chapter7/7.13.c)中，全局变量、静态变量和易失变量(volatile variables)不受优化的影响。如果要编写一个使用非局部跳转的可移植程序，则必须使用`volatile`属性。但是一个系统移植到另一个系统，其他任何事情都可能改变。

# 函数`getrlimit`和`setrlimit`
```c
#include <sys/resource.h>

int getrlimit(int resource, struct rlimit *rlptr);
int setrlimit(int resource, const struct rlimit *rlptr);
/* 两个函数返回值：若成功，返回0；若出错，返回非0 */
```
这两个函数的每一次调用都指定一个资源以及一个指向下列结构的指针。
```c
struct rlimit {
    rlim_t rlim_cur;    /* soft limit: current limit */
    rlim_t rlim_max;    /* hard limit: maximum value for rlim_cur */
};
```

1. 任何一个进程都可以将一个限制值更改为小于或等于其硬限制值。
2. 任何一个进程都可以降低其硬限制值，但它必须大于或等于其软限制值。
3. 只有超级用户进程才可以提高硬限制值。





