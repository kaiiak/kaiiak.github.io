title: APUE学习笔记——进程控制
date: 2016-04-27 23:08:52
tags: [APUE,Linux]
---
# 进程标识
每个进程都有一个非负整型表示的唯一进程ID。
- ID为0的进程通常是调度进程，常常被称为`交换进程(swapper)`。该进程是内核的一部分，它并不执行任何磁盘上的程序，因此被称为系统进程。
- 进程ID为1的通常是`init`进程，在自举过程结束时由内核调用。`init`进程决不会终止。它是一个普通的用户进程，但它以超级用户特权运行。
- 进程ID2是`页守护进程(page daemon)`，此进程负责支持虚拟存储器系统的分页操作。

```c
#include <unistd.h>

pid_t getpid(void);     /* 返回值：调用进程的进程ID */
pid_t getppid(void);    /* 返回值：调用进程的父进程ID */
uid_t getuid(void);     /* 返回值：调用进程的实际用户ID */
uid_t geteuid(void);    /* 返回值：调用进程的有效用户ID */
gid_t getgid(void);     /* 返回值：调用进程的实际组ID */
gid_t getegid(void);    /* 返回值：调用进程的有效组ID */ 
```

# 函数`fork`
```c
#include <unistd.h>

pid_t fork(void);
/* 返回值：子进程返回0，父进程返回子进程ID；若出错，返回-1 */
```
由`fork`创建的新进程被称为`子进程(child process)`。
两次返回的区别是子进程的返回值是0，而父进程的返回值则是新建子进程的进程ID。
子进程和父进程继续执行`fork`调用之后的指令。子进程是父进程的副本。父进程和子进程共享正文段。
一般来说，`fork`之后是父进程先执行还是子进程先执行是不确定的。
在重定向父进程的标准输出时，子进程的标准输出也被重定向。
子进程和父进程共享一个文件偏移量。

# 函数`vfork`
父进程和子进程共享数据段，并且先保诚子进程先运行，只有当子进程调用`exec`或`exit`后父进程才可能被调用运行。
*调用vfork()后，子程序要么_exit(),要么调用exec()，否则都是未定义行为(UB)*

# 函数`exit`
5种正常终止方式：
1. 在`main`函数内执行`return`语句。
2. 调用`exit`函数。
3. 调用`_exit`或`_Exit`函数。
4. 进程的最后一个线程在其启动例程中执行`return`语句。
5. 进程的最后一个线程调用`pthread_exit`函数。

3种异常终止：
1. 调用`abort`。
2. 当进程接收到某些信号时。
3. 最后一个线程对“取消”(cancellation)请求作出响应。

对于父进程已经终止的所有进程，它们的父进程都改为`init`进程，我们称为这些进程由`init`进程收养。

在UNIX术语中，一个已经终止、但是其父进程尚未对其进行善后处理(获取终止子进程的有关信息、释放它仍占用的资源)的进程被称为`僵死进程(zombie)`。

# 函数`wait`和`waitpid`
当一个进程正常或异常终止时，内核就向其父进程发送`SIGCHLD`信号。

- 如果其所有子进程都还在运行，则阻塞。
- 如果一个子进程终止，正等待父进程获取其终止状态，则取得该子进程的终止状态立即返回。
- 如果没有任何子进程，则立即出错返回。
```c
#include <sys/wait.h>

pid_t wait(int *statloc);
pid_t waitpid(pid_t pid, int *statloc, int options);
/* 两个函数返回值：若成功，返回进程ID；若出错，返回0或-1 */
```

- 在一个子进程终止前，`wait`使其调用者阻塞，而`waitpid`有一选项，可使调用者不阻塞。
- `waitpid`并不等待在其调用之后的第一个终止子进程，可以控制它所等待的进程。

这两个函数的参数`statloc`是一个整型指针。如果`statloc`不是一个空指针，则终止进程的终止状态就存放在它所指向的单元内。

|宏|说明|
|:--------:|:-------------------------------------:|
|WIFEXITED(status)|若为正常正常终止子进程返回的状态，则为真。对于这种情况可执行WIFEXITED(status)，获取子进程传送给exit或_exit参数的低8位|
|WIFSIGNALED(status)|若为异常终止子进程返回的状态，则为真(接到一个不捕捉的信号)。对于这种情况，可执行WIFSIGNALED(status)，则为真(接到一个不捕捉的信号)。对于这种情况，可执行()WTERMSIG(status),获取使子进程终止的信号编号。另外，有些实现(非Single UNIX Specification)定义宏WCOREDUMP(status),若已产生终止进程core文件，则它返回真|
|WIFSTOPPED(status)|若为当前暂停子进程的返回的状态，则为真。对于这种情况，可执行WSTOPSIG(status),获取使子进程暂停的信号编号|
|WIFCONTINUED(status)|若在作业控制暂停后已经继续的子进程返回了状态，则为真|

`waitpid`函数中pid参数的作用解释如下：
- pid == -1 等待任一子进程。此种情况下，`wait`和`waitpid`等效。
- pid > 0   等待进程ID与pid相等的子进程。
- pid == 0  等待组ID等于调用进程组ID的任一子进程。
- pid < -1  等待组ID等于pid绝对值的任一子进程。

|options常量|说明|
|:--------:|:----------------------------:|
|WCONTINUED|若实现支持罪业控制，那么由pid制定的任一子进程在停止后已经继续，但其状态尚未报告，则返回其状态|
|WNOHANG|若由pid指定的子进程并不是立即可用的，则waitpid不阻塞，此时其返回值为0|
|WUNTRACED|若某实现支持作业控制，而由pid指定的任一子进程已处于停止状态，并且其状态由停止以来还没报告过，则返回其状态。WIFSTOPPED宏确定返回值是否对应于一个停止的子进程。|

# 函数`waittid`
```c
#include <sys/wait.h>

int waittid(idtype_t idtype, id_t id, siginfo_t *infop, int options);
/* 返回值：若成功，返回0；若出错，返回-1 */
```

|idtype常量|说明|
|:--------:|:----------------------------:|
|P_PID|等待一特定进程：id包含要等待子进程的进程ID|
|P_PGID|等待一特定进程组中的任一子进程：id包含要等待子进程的进程组ID|
|P_ALL|等待任一子进程：忽略id|

|options常量|说明|
|:--------:|:----------------------------:|
|WCONTINUED|等待一进程，它以前曾被停止，此后又已继续，但其状态尚未报告|
|WEXITED|等待已退出的进程|
|WNOHANG|如无可用的子进程推出状态，立即返回而非阻塞|
|WNOWAIT|不破坏子进程退出状态。该子进程退出状态可由后续的wait、waitid或waitpid调用取得|
|WSTOPPED|等待一进程，它已经停止，但其状态尚未报告|

# 函数`wait3`和`wait4`
```c
#include <sys/types.h>
#include <sys/wait.h>
#include <sys/time.h>
#include <sys/resource.h>

pid_t wait3(int *statloc, int options, struct rusage *rusage);
pid_t wait4(pid_t pid, int *statloc, int options, struct rusage *rusage);
/* 两个函数返回值：若成功，返回进程ID；若出错，返回-1 */
```

# 函数`exec`
调用`exec`并不创建新进程，所以前后的进程ID并没有改变。`exec`只是用磁盘上的一个新程序替换了当前进程的正文段、数据段、堆栈和栈段。

```c
#include <unistd.h>

int execl(const char *pathname, const char *arg0, ... /* (char *)0 */);
int execv(const char *pathname, char *const argv[]);
int execle(const char *pathname, const char *arg0, ...
    /* (char *)0, char *const envp[] */);
int execve(const char *pathname, char *const argv[], char *const envp[]);
int execlp(const char *filename, const char *arg0, ... /* (char *)0 */);
int execvp(const char *filename, char *const argv[]);
int fexecve(int fd, char *const argv[], char *const envp[]);
/* 7个函数返回值：若出错，返回-1；若成功，不返回 */
```

当`filename`作为参数时：
- 如果`filename`中包含`/`，则将其设为路径名；
- 否则就按`PATH`环境变量，在她所指定的各目录搜寻可执行文件。

execl,execle,execlp(结尾带l)要在可变参数结尾添加`(char *)0`。
execlp,execvp(结尾带p)表示第一个参数path不用输入完整路径，只有给出命令名即可，它会在环境变量PATH当中查找命令。
execv,execvp(不带l)表示命令所需的参数以char *arg[]形式给出且arg最后一个元素必须是NULL。

# 更改用户ID和更改组ID
可以用`setuid`函数设置实际用户ID和有效用户ID；可以用`setgid`函数设置实际组ID和有效组ID。
```c
#include <unistd.h>

int setuid(uid_t uid);
int setgit(gid_t gid);
/* 两个函数返回值：若成功，返回0；若出错，返回-1 */
```
1. 若进程拥有超级用户权限，则setuid函数将实际用户ID、有效用户ID以及保存的设置用户ID设置为uid；
2. 若进程没有超级用户权限，但是uid等于实际用户ID或保存的设置用户ID，则`setuid`只将有效用户ID设置为uid。不更改实际用户ID和保存的设置用户ID；
3. 若以上两个条件都不满足，则`errno`设置为`EPERM`，并返回-1.

1. 只有超级用户进程可以更改实际用户ID；
2. 仅当对程序文件设置了设置用户ID位时，`exec`函数才设置有效用户ID；
3. 保存的设置用户ID是由`ecec`复制有效用户ID而得到的。


## 函数`setreuid`和`setregid`
功能是交换实际用户ID和有效用户ID
```c
#include <unistd.h>

int setreuid(uid_t ruid, uid_t euid);
int setregid(gid_t rgid, gid_t egid);
/* 两个函数的返回值：若成功，返回0；若出错，返回-1 */
```
如若其中任一参数的值为-1，则表示相应的ID应当保持不变。

## 函数`seteuid`和`setegid`
类似于`setuid`和`setgid`，但只更改有效用户ID和有效组ＩＤ。
```c
#include <unistd.h>

int seteuid(uid_t uid);
int setegid(uid_t gid);
/* 两个函数的返回值：若成功，返回0；若出错，返回-1 */
```

# 函数`system`
```c
#include <stdlib.h>

int system(const char *cmdstring);
```
如果`cmdstring`是一个空指针，则仅当命令程序可用时，`system`返回非0值，这一特征可以确定在一个给定的操作系统上是否支持`system`函数。

1. `fork`失败或者`waitpid`返回处`EINTR`之外的出错，则`system`返回-1，并且设置`errno`以指示错误类型。
2. 如果`exec`失败，则返回值如同shell执行了`exit(127)`一样。
3. 否则所有3个函数(`fork`、`exec`和`waitpid`)都成功，那么`system`的返回值是shell的终止状态，其格式已在`waitpid`中说明。

# 进程会计(process accounting)
```c
typedef u_short comp_t; /* 3-bit base 8 exponent; 13-bit fraction */
struct acct
{
    char ac_flag;   /* flag (see Figure 8.26) */
    char ac_stat;   /* termination status (signal & core flag only) */
                    /* (Solaris only) */
    uid_t ac_uid;   /* real user ID */
    gid_t ac_gid;   /* real group ID */
    dev_t ac_tty;   /* controlling terminal */
    time_t ac_btime; /* starting calendar time */
    comp_t ac_utime; /* user CPU time */
    comp_t ac_stime; /* system CPU time */
    comp_t ac_etime; /* elapsed time */
    comp_t ac_mem;  /* average memory usage */
    comp_t ac_io;   /* bytes transferred (by read and write) */
                    /* "blocks" on BSD systems */
    comp_t ac_rw;   /* blocks read or written */
                    /* (not present on BSD systems) */
    char ac_comm[8]; /* command name: [8] for Solaris, */
                     /* [10] for Mac OS X, [16] for FreeBSD, and */
                     /* [17] for Linux */
};
```

1. 我们不能获取永远不终止的进程的会计记录；
2. 在会计文件记录的顺序对应于进程终止的顺序，而不是它们启动的顺序；
3. 会计记录对应于进程而不是程序。`exec`并不创建一个新的会计记录，但相应记录中的命令名称改变了，`AFORK`标志被清除。

# 用户标识
得到运行该程序的用户的登录名，当有多个用户名对应着一个用户ID时，通常返回用户登录时用的用户名。
```c
#include <unistd.h>

char *getlogin(void);
/* 返回值：若成功，返回指向登陆名字符串的指针；若出错，返回NULL */
```

# 进程调度
进程可以通过调整`nice`值选择以最低优先级运行，只有特权进程允许提高调度权限。
`nice`值的大小在0~(2*NZERO)-1之间，有些实现支持0~2*NZERO。*nice值越小，优先级越高*。
```c
#include <unistd.h>

int nice(int incr);
/*返回值：若成功，返回新的nice值NZERO；若出错，返回-18*/
```

`getpriority`函数可以像`nice`函数那样用户获取进程的`nice`值，但是`getpriority`还可以获取一组相关进程的`nice`值。
```c
#include <sys/resource.h>

int getpriority(int which, id_t who);
/* 返回值：若成功，返回-NZERO~NZERO-1之间的nice值；若出错，返回-1 */
```

`setpriority`函数可以用于为进程、进程组和属于特定用户ID的所有进程设置优先级。
```c
#include <sys/resource.h>

int setpriority(int which, id_t who, int value);
/* 返回值：若成功，返回0；若出错，返回-1 */
```

# 进程时间
`times`函数获得它自己以及已终止子进程的墙上时钟时间、用户CPU时间和系统CPU时间。
```c
#include <sys/times.h>

clock_t times(struct tms *buf);
/* 返回值：若成功，返回墙上时钟时间(以时钟滴答数为单位)；若出错，返回-1 */
```

```c
struct tms {
    clock_t tms_utime;  /* 用户CPU时间 */
    clock_t tms_stime;  /* 系统CPU时间 */
    clock_t tms_cutime; /* 子进程的用户CPU时间 */
    clock_t tms_cstiem; /* 子进程的系统CPU时间 */
}
```















