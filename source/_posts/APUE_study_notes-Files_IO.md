title: APUE学习笔记——文件IO
date: 2016-02-25 08:36:08
tags: [Linux,APUE]
---
# 文件描述符

所有打开的文件都通过文件描述符(一个非负整数)引用。

维基百科上对于文件描述符的解释是这样的：
>文件描述符在形式上是一个非负整数。实际上，它是一个索引值，指向内核为每一个进程所维护的该进程打开文件的记录表。当程序打开一个现有文件或者创建一个新文件时，内核向进程返回一个文件描述符。在程序设计中，一些涉及底层的程序编写往往会围绕着文件描述符展开。但是文件描述符这一概念往往只适用于UNIX、Linux这样的操作系统。

当读、写一个文件时，使用`open`或`creat`返回的文件描述符标识该文件，将其作为参数传给`read`或`write`。

在POSIX.1的应用程序中，文件描述符0与进程的标准输入关联(STDIN_FILENO)、文件描述符1与进程的标准输出关联(STDOUT_FILENO)、文件描述符2与进程的标准错误关联(STDERR_FILENO)，且已经确定化。
文件描述符的变化范围是`0~OPEN_MAX-1`。

# 函数`open`和`openat`

```c
#include <fcntl.h>
int open(const char *path, int oflag, ... /* mode_t mode */);
int openat(int fd, const char *path, int oflag, ... /* mode_t mode */);
/*
 *path ：文件的名称，可以包含（绝对和相对）路径
 *flags：文件打开模式
 *mode：用来规定对该文件的所有者，文件的用户组及系 统中其他用户的访问权限
 */
/* 两函数的返回值： 若成功，返回文件描述符； 若出错，返回-1 */
```

|打开方式|描述|
|------------|:---------------------------------------------------------------|
|`O_RDONLY`| 只读打开.|
|`O_WRONLY`| 只写打开.|
|`O_RDWR`| 读、写打开.|
|`O_EXEC`| 只执行打开.|
|`O_SEARCH`| 只搜索打开 (应用于目录).|
|`O_APPEND`| 每次写时都追加到文件的尾端.|
|`O_CLOEXEC`| 设置`FD_CLOEXEC`文件描述符. |
|`O_CREAT`| 若此文件不存在则创建它.|
|`O_DIRECTORY`|如果`path`引用的不是目录，则出错.|
|`O_EXCL`|如果同时指定了`O_CREAT`，而文件已存在，则出错.|
|`O_NOCTTY`| 如果`path`引用的是终端设备，则不将该设备分配作为此进程的控制终端.|
|`O_NOFOLLOW`|如果`path`引用的是一个符号连接，则出错.|
|`O_NONBLOCK`| 如果`path`引用的是一个`FIFO`、一个块特殊文件或一个自负特殊文件，则此选项为文件的本次打开操作和后续的I/O操作设置非阻塞方式.|
|`O_SYNC`|使每次`write`等待物理I/O完成，包括由该`write`操作引起的文件属性更新所需的I/O.|
|`O_TRUNC`|如果此文件存在，而且为只写或读-写成功打开，则将其长度截断为0.|
|`O_TTY_INIT`|如果打开一个还未打开的终端设备，设置非标准`termios`参数值，使其符合`Single UNIX Specification`.|
|`O_DSYNC`|使每次`write`要等待物理I/O完成，但是如果该写操作并不影响读取刚写入的数据，则不需等待文件属性被更新.|
|`O_RSYNC`|使每一个一文件描述符作为参数进行的`read`操作等待，直至所有对文件同一部份挂起的写操作都完成.|

### `fd`参数吧`open`和`openat`函数区分开，共有3种可能性。
1. `path`参数指定的是绝对路径名，在这种情况下，`fd`参数被忽略，`openat`函数就相当于`open`函数。
2. `path`参数指定的是相对路径名，`fd`参数指出了相对路径名在文件系统中的开始地址。`fd`参数是通过打开相对路径名所在的目录来获取。
3. `path`参数指定了相对路径名，`fd`参数具有特殊值`AT_FDCWD`。在这种情况下，路径名在当前工作目录中获取，`openat`函数在操作上与`open`函数类似。

# 函数`creat`

### 调用函数`creat`创建一个新的文件
```c
#include <fcntl.h>

int creat(const char *path, mode_t mode);
/* 返回值： 若成功，返回为只写打开的文件描述符；如出错，返回-1 */
```
等效于：
```c
open(path, O_WRONLY|O_CREAT|O_TRUNC, mode);
```

# 函数`close`

### 调用`close`函数关闭一个打开文件
```c
#include <unistd.h>
int close(int fd);
/* 返回值：若成功，返回0；如出错，返回-1 */
```

# 函数`lseek`

### 当前文件偏移量(current file offset)：它通常是一个非负整数，用以度量从文件开始处计算的字节数。

### 调用`lseek`显式的为一个打开文件设置偏移量。
```c
#include <unistd.h>

off_t lseek(int fd, off_t offset, int whence);
/* 返回值：若辰宫，返回新的文件偏移量；若出错，返回-1 */
```

对参数`offset`的解释与参数`whence`的值有关。
- 若`whence`是`SEEK_SET`，则经该文件的偏移量设置为距文件开始处`offset`个字节。
- 若`whence`是`SEEK_CUR`，则经该文件的偏移量设置为其当前量增加`offset`，`offset`可为正或负。
- 若`whence`是`SEEK_END`，则经该文件的偏移量设置为文件长度加`offset`，`offset`可正可负。

# 函数`read`

### 调用`read`函数从打开文件中读数据。
```c
#include <unistd.h>

ssize_t read(int fd, void *buf, size_t nbytes);
/* 返回值：读到的字节数，若已到文件尾，返回0；若出错，返回-1 */
```

有多种情况可使实际读到的字节数少于要求读的字节数：
- 读到普通文件时，在读到要求字计数之前已到达了文件微端。
- 当从终端设备读时，通常一次最多读一行。
- 当从网络读时，网络中的缓冲机制可能造成返回值小于所要求的字节数。
- 当从管道或`FIFO`读时，如若管道包含的字节少于所需的数量，那么`read`将只返回实际可用的字节数。
- 当从某些面向记录的设备读时，一次最多返回一个记录。
- 当一信号造成中断，而已经读到部分数据量时。

# 函数`write`

### 调用`write`函数向打开文件写数据。
```c
#include <unistd.h>

ssize_t write(int fd, const void *buf, size_t nbytes);
/* 返回值：若成功，返回已写的字节数；若出错，返回-1 */
```

## 函数`pread`和`pwrite`

```c
#include <unistd.h>

ssize_t pread(int fd, void *buf, size_t nbytes. off_t offset);
/* 返回值：读到的字节数，若已到文件尾，返回0；若出错，返回-1 */

ssize_t pwrite(int fd, const void *buf, size_t nbytes, off_t offset);

```

调用`pread`相当于调用`lseek`后调用`read`，但是`pread`又与这种顺序调用有下列重要区别。
- 调用`pread`时，无法中断其定位和都读操作。
- 不更新当前文件偏移量。
调用`pwrite`相当于调用`lseek`后调用`write`，但也与它们有类似的区别。

>原子操作(atomic operation)；指的是由多步组成一个一个操作。如果该操作原子地执行，则要么执行所有步骤，要么一步也不执行，不可能只执行所有步骤的一个子集。

# 函数`dup`和`dup2`

### 复制一个现有的文件描述符
```c
#include <unistd.h>

int dup(int fd);
int dup2(int fd, int fd2);
/* 两函数的返回值：若成功，返回新的文件描述符；如出错，返回-1 */
```

# 函数`sync`、`fsync`和`fdatasync`

>内核通常先将数据复制到缓冲区中，然后排入队列，晚些时候写入磁盘。这种方法被称为`延迟写(delayed write)`.

```c
#include <unitstd.h>

int fsync(int fd);
int fdatasync(int fd);
/* 返回值：若成功，返回0；若出错，返回-1 */

void sync(void);
```

`sync`只是将所有修改过的块缓冲区排入写队列，然后就返回，它并不等待实际写磁盘操作结束。
`fsync`函数只对由文件描述符`fd`指定的一个文件起作用，并且等待写磁盘操作结束才返回。
`fdatasync`函数类似于`fsync`，但它只影响文件的数据部分。

# 函数`fcntl`

### 改变已经打开文件的属性。
```c
#include <fcntl.h>

int fcntl(int fd, int cmd, ... /* int arg */);
/* 返回值：若成功，则依赖cmd，若出错，返回-1 */
```

#### `fcntl`函数有以下5种功能。
1. 复制一个已有的描述符 (`cmd=F_DUPFD`或`F_DUPFD_CLOEXEC`)。
2. 获取/设置文件描述符标志 (`cmd=F_GETFD`或`F_SETFD`)。
3. 获取/设置文件状态标志 (`cmd=F_GETFL`或`F_SETFL`)。
4. 获取/设置异步I/O所有权 (`cmd=F_GETOWN`或`F_SETOWN`)。
5. 获取/设置记录锁 (`cmd=F_GETLK`或`F_SETLKW`)。

|cmd|功能
|
|------|:----------------------------------------------------|
|`F_DUPFD`|复制文件描述符`fd`。|
|`F_DUPFD_CLOEXEC`|复制文件描述符，设置与新描述符关联的`FD_CLOEXEC`文件描述符标志的值，返回新文件描述符。|
|`F_GETFD`|对应于`fd`的文件描述符标志作为函数值返回。|
|`F_SETFD`|对于`fd`设置文件描述符标志。|
|`F_GETFL`|对应于`fd`的文件状态标志作为函数值返回。|
|`F_SETFL`|将文件状态标志设置为第3个参数的值。可以更改的几个标志是：`O_APPEND`、`O_NONBLOCK`、`O_SYNC`、`O_DSYNC`、`O_RSYNC`、`O_FSYNC`、和`O_ASYNC`。|
|`F_GETOWN`|获取当前接收`SIGIO`和`SIGURG`信号进程ID或进程组ID。|
|`F_SETOWN`|设置接收`SIGIO`和`SIGURG`信号的进程ID或进程组ID。|

# 函数`ioctl`

```c
#include <unistd.h> /* System V */

#include <sys/ioctl.h>  /* BSD and Linux */

int ioctl(int fd, int request, ...);
/* 返回值：若出错，返回-1；若成功，返回其他值 */
```

## `/dev/fd`

较新的系统都提供名为`/dev/fd`的目录，其目录项是名为0、1、2等的文件。打开文件`/dev/fdn`等效于复制描述符n(假设描述符n是打开的).