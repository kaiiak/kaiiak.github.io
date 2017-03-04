title: APUE学习笔记——文件和目录
date: 2016-02-29 21:48:30
tags: [Linux,APUE]
---
# 函数`stat`、`fstat`、`fstatat`、`lstat`
```c
#include <sys/stat.h>

int stat(const char *restrict pathname, struct stat *restrict buf);
int fstat(int fd, struct stat *buf);
int lstat(const char *restrict pathname, struct stat *restrict buf);
int lstat(int fd, const char *restrict pathname, struct stat *restrict buf, int flag);

/* 所有4个函数的返回值：若成功，返回0；若出错，返回-1 */

sturct stat { 
    mode_t          st_mode;        /* file type & mode (permissions) */
    ino_t           st_ino;         /* i-node number (serial number) */
    dev_t           st_dev;         /* device number (file system) */
    dev_t           st_rdev;        /* device number for special files */
    nlink_t         st_nlink;       /* number of links */
    uid_t           st_uid;         /* user ID of links */
    gid_t           st_gid;         /* group ID of owner */
    off_t           st_size;        /* size in bytes, for regular files  */
    struct timespec st_atime;       /* time of last access */
    struct timespec st_mtime;       /* time of last modification */
    struct timespec st_ctime;       /* time of last file status change */
    blksize_t       st_blksize;     /* best I/O block size */
    blkcnt_t        st_blocks;      /* number of disk blocks allocated */
}
```

- 函数`stat`:返回`pathname`命名文件有关的信息结构；
- 函数`fstat`:返回在描述符`fd`上打开文件的有关信息；
- 函数`lstat`:类似于函数`stat`，但当命名的文件是一个符号链接时，`lstat`返回该符号连接的有关信息；
- 函数`fstatat`:为一个相对于当前打开目录(由`fd`参数指向)的路径名返回文件统计信息。

文件信息包含`stat`结构的`st_mode`成员中。可以下表中的宏文件确定文件的类型。

|宏|文件类型|
|-----------|:----------|
|`S_ISREG()`|普通文件|
|`S_ISDIR()`|目录文件|
|`S_ISCHR()`|字符特殊文件|
|`S_ISBLK()`|块特殊文件|
|`S_ISFIFO()`|管道或FIFO|
|`S_ISLNK()`|符号链接|
|`S_ISSOCK()`|套接字|




# 文件类型
文件类型包括如下几类：

1. 普通文件(regular file):包含了某种形式的数据；
2. 目录文件(directory file):包含了其他文件的名字以及指向与这些文件有关信息的指针；
3. 块特殊文件(block special file):该类型文件提供对设备(如磁盘)带缓冲的访问，每次访问以固定长度为单位进行；
4. 字符特殊文件(character special file):该文件提供对设备不带缓冲的访问，每次访问长度不变。系统中的所有设备要么是字符特殊文件，要么是块特殊文件。
5. FIFO:该文件用于进程通信，也称为命名管道(named pipe)；
6. 套接字(socket)：该文件用于进程间的网络通信，也可以用在一台宿主机上进程间的非网络通信。
7. 符号链接(symbolic link):该文件指向另一个文件。

# 设置用户ID和设置组ID
- 实际用户ID和实际组ID标识我们究竟是谁。
- 有效用户ID、有效组ID以及附属组ID决定了我们的文件访问权限。
- 保存的设置用户ID和保存的设置组ID在执行一个程序时包含了有效用户ID和有效组ID的副本。

每个文件有一个所有者和组所有者，所有者由`stat`结构中的`st_uid`指定，组所有者则由`st_gid`指定。

# 文件访问权限
所有文件类型(目录、字符特别文件等)都有访问权限(access permisiion)。

|`st_mode`屏蔽|含义|
|-------------|:-------------------------|
|`S_IRUSR`|用户都|
|`S_IWUSR`|用户写|
|`S_IXUSR`|用户执行|
|`S_IRGRP`|组读|
|`S_IWGRP`|组写|
|`S_IXGRP`|组执行|
|`S_IROTH`|其他读|
|`S_IWOTH`|其他写|
|`S_IXOTH`|其他执行|

# 函数`access`和`faccessat`
```c
#include <unistd.h>
int access(const char *pathname, int mode);
int faccessat(int fd, const char *pathname, int mode, int flag);
/* 两个函数的返回值：若成功，返回0；若出错，返回-1 */
```

`access`和`faccessat`函数是按实际用户ID和实际组ID进行访问权限测试的。

|`mode`|说明|
|--------|:----------|
|`R_OK`|测试读权限|
|`W_OK`|测试写权限|
|`X_OK`|测试执行权限|

# 函数`umask`
`umask`函数为进程设置文件模式创建屏蔽字(mask for the process)，并返回之前的值。(这是少数几个没有出错返回函数中的一个。)

```c
#include <sys/stat.h>

mode_t umask(mode_t cmask);
/* 返回值：之前的文件模式穿件屏蔽字 */
```
参数`cmask`是由上上个表列出的9个常量(S_IRUSR、S_IWUSR等)中若干个按位“或”构成的。

|屏蔽位|含义|
|--------|:---------------|
|0400|用户读|
|0200|用户写|
|0100|用户执行|
|0040|组读|
|0020|组写|
|0010|组执行|
|0004|其他读|
|0002|其他写|
|0001|其他执行|

# 函数`chmod`、`fchmod`和`fchmodat`
```c
#include <sys/stat.h>

int chomd(const char *pathname, mode_t mode);
int fchome(int fd, mode_t mode);
int fchomdat(int fd, const char *pathname, mode_t mode, int flag);
/* 3个函数返回值：若成功，返回0；若出错，返回-1 */
```

`chmod`函数在指定的文件上进行操作，而`fchmod`函数则对已打开的文件进行操作。
`fchomdat`函数与`chmod`函数在下面两种情况下是相同的：一种是`pathname`参数为绝对路径，另一种是`fd`参数取值为`AT_FDCWD`而`pathname`参数为相对路径。否则，`fchmodat`计算相对于打开目录(由`fd`参数指向)的pathname。`flag`参数可以用于改变`fchmodat`的行为，当设置了`AT_SYMLIN_NOFOLLOW`标志时，`fchmodat`并不会跟随符号链接。

|`mode`|说明|
|-------------------|:-------------------------------------|
|`S_ISUID`|执行时设置用户ID|
|`S_ISGID`|执行时设置组ID|
|`S_ISVTX`|保存正文(粘着位)|
|`S_IRWXU`|用户(所有者)读、写和执行|
|`S_IRUSR`|用户(所有者)读|
|`S_IWUSR`|用户(所有者)写|
|`S_IXUSR`|用户(所有者)执行|
|`S_IRWXG`|组读、写和执行|
|`S_IRGRP`|组读|
|`S_IWGRP`|组写|
|`S_IXGRP`|组执行|
|`S_IRWXO`|其他读、写和执行|
|`S_IROTH`|其他读|
|`S_IWOTH`|其他写|
|`S_IXOTH`|其他执行|

# 粘着位(sticky bit)
`S_ISVTX`位被称为`粘着位`：如果一个可执行的程序文件的这一位被设置了，那么当程序第一次被执行，在其终止时，程序正文的一个副本仍被保存在交换区。
后来的UNIX版本称为它为`保存正文位(saved-text bit)`，因此也就有了常量`S_ISVTX`。

# 函数`chown`、`fchown`、`fchownat`和`lchown`
下面几个`chown`函数可用于更改文件的用户ID和组ID。如果两个参数`owner`或`group`中的任意一个是-1，则对应的ID不变。

```c
#include <unistd.h>

int chown(const char *pathname, uid_t owner, git_t group);
int fchown(int fd, uid_t owner, gid_t group);
int fchownat(int fd, const char *pathname, uid_t owner, gid_t group, int flag);
int lchown(const char *pathname, uid_t owner, gid_t group);
/*
 * 4个函数的返回值：若辰宫，返回0；若出错，返回-1
 */
```

# 文件长度
`stat`结构成员`st_size`表示以字节为单位的单位长度。只对普通文件、目录文件和符号连接有效。

# 文件中的空洞
空洞是由所设置的偏移量超过文件端尾，并写入了某些数据后造成的。

# 文件截断
```c
#include <unistd.h>

int truncate(const char *pathname, off_t length);
int ftruncate(int fd, off_t length);
/*
 * 两个函数的返回值：若成功，返回0；若失败，返回-1
 */
```

如果该文件以前的长度大于`length`，则超过`length`以外的数据就不能再访问。如果以前的长度小于`length`，文件长度将增加，在以前的文件尾端和新的文件尾端之间的数据将读作0。

# 文件系统

# 函数`link`、`linkat`、`unlink`、`unlinkat`和`remove`
```c
#include <unistd.h>

int link(const char *existingpath, const char *newpath);
int linkat(int efd, const char *existingpath, int nfd, const char *newpath, int flag);
/* 两个函数的返回值：若成功，返回0；若出错，返回-1 */
```

这两个函数创建一个新目录项`newpath`，它引用现有文件`existingpath`。如果`newpath`已存在，则返回出错。
对于`linkat`函数，现有文件是通过`efd`和`existingpath`指定，新的路径名是通过`nfd`和`newpath`指定的。

```c
#include <unistd.h>

int unlink(const char *pathname);
int unlinkat(int fd, const char *pathname, int flag);
/* 两个函数的返回值：若成功，返回0；若出错，返回-1 */
```

这两个函数删除目录项，并将由`pathname`所引用的链接数减1。

```c
#include <stdio.h>

int remove(const char *pathname);
/* 返回值：若成功，返回0；若失败，返回-1 */
```

# 函数`rename`和`renameat`

```c
#include <stdio.h>

int rename(const char *oldname, const char *newname);
int renameat(int oldfd, const char *oldname, int newfd, const char *newname);
```

1. 如果`oldname`指的是一个文件而不是目录，那么为该文件或符号连接重命名。
2. 如果`oldname`指的是一个空目录，那么为该目录重命名。
3. 如果`oldname`或`newname`引用符号连接，则处理的是符号链接本身，而不是它引用的文件。
4. 不能对`.`和`..`重命名。
5. 作为一个特例，如果`oldname`和`newname`引用同意文件，则函数不做任何更改而成功返回。

# 符号链接
符号链接是对一个文件的间接指针。符号链接以及它指向何种对象并无任何文件系统限制，任何用户都可以创建指向目录的符号连接。

# 创建和读取符号连接
```c
#include <unistd.h>

int symlink(const char *actualpath, const char *sympath);
int symlinat(const char *acutalpath, int fd, const char *symptah);
/* 两个函数的返回值：若成功，返回0；若出错，返回-1 */
```

```c
#include <unistd.h>

ssize_t readlink(const char *actualpath, const char *symptah);
sszie_t readlinkat(int fd, const char* restrict pathname, char *restrict buf, size_t bufsize);
/* 两个函数的返回值：若成功，返回读取的字节数；若出错，返回-1 */
```

# 文件时间

|字段|说明|例子|ls(1)选项|
|------|:--------------|:-----|:-----|
|`st_atim`|文件数据的最后访问时间|`read`|`-u`|
|`st_mtim`|文件数据的最后修改时间|`write`|`默认`|
|`st_ctim`|i节点状态的最后更改时间|`chomd、chown`|`-c`|

# 函数`futimens`、`utimensat`和`utimes`
```c
#include <sys/stat.h>

int futimens(int fd, const struct timespec times[2]);
int utimensat(int fd, const char *path, const struct timespec times[2], int flag);
/* 两个函数返回值：若成功，返回0；若出错，返回-1 */
```

# 函数`mkdir`、`mkdirat`和`rmdir`
两个函数创建一个新目录。
```c
#include <sys/stat.h>

int mkdir(const char *pathname, mode_t mode);
int mkdir(int fd, const char *pathname, mode_t mode);
/* 两个函数返回值：若成功，返回0；若出错，返回-1 */
```

用`rmdir`函数可以创建一个空目录。
```c
#include <unistd.h>

int rmdir(const char *pathname);
/* 返回值：若成功，返回0；若失败，返回-1 */
```


# 读目录
```c
#include <dirent.h>

DIR *opendir(const char *pathname);
DIR *fdopendir(int fd);
/* 两个函数返回值：若成功，返回指针；若出错，返回NULL */

struct dirent *readdir(DIR *dp);
/* 返回值：若成功，返回指针；若在目录尾或出错，返回NULL */

void rewinddir(DIR *dp);
int closeddir(DIR *dp);
/* 返回值：若成功，返回0；若失败，返回-1 */

long telldir(DIR *dp);
/* 返回值：与dp关联的相关位置 */

void seekdir(DIR *dp, long loc);
```

# 函数`chdir`、`fchdir`和`getcwd`
```c
#include <unistd.h>

int chdir(const char *pathname);
int fchdir(int fd);

```

进程调用`chdir`和`fchdir`函数更改当前工作目录。

```c
#include <unistd.h>

char *getcwd(char *buf, size_t size);
```
逐层上移，直到遇到根，得到当前工作目录完整的绝对路径。
必须像此函数传递两个函数，一个是缓冲区地址`buf`(要有足够的长度容纳绝对路径再加上一个终止`null`字节)，另一个是缓冲区的长度`size`(以字节为单位)。

