title: APUE学习笔记——格式化I/O
date: 2016-03-29 21:53:18
tags: [Linux,APUE]
---
# 流和FILE对象
对于`ASCII`字符集，一个字符用于一个字节表示。对于国际字符集，一个字符可用于多个字节表示。标准I/O文件流可用于单字节或多字节(“宽”)字符集。流的定向(stream's orientation)决定了所读、所写的字符是单字节还是多字节。
```c
#include <stdio.h>
#include <wchar.h>

int fwide(FILE *fp, int mode);
/*
 * 返回值：若流是宽定向的，返回正值；
 * 若流是字节定向的，返回负值；若流是未定向的，返回0
 * 如果mode参数值为负，fwide将试图使指定的流是字节定向的。
 * 如果mode参数值为正，fwide将试图使指定的流是宽定向的。
 * 如果mode参数值为0，fwide将不试图设置流的定向，但返回标识该流定向的值。
 */
```

# 标准输入、标准输出和标准错误
可以通过预定义文件指针`stdin`、`stdout`和`stderr`加以引用。

# 缓冲
- 全缓冲。在这种情况下，在填满标准I/O缓冲区才进行实际I/O操作。
        冲洗(flush)说明标准I/O缓冲区的写操作。也可以直接调用`fflush`函数冲洗一个流。在终端驱动程序中，`flush(刷清)`表示丢弃已存储在缓冲区中的数据。
- 行缓冲。在这种情况下，当在输入和输出中遇到换行符时，标准I/O库执行I/O操作。
- 不带缓冲。标准I/O库不对字符进行缓冲存储。标准错误流`stderr`通常是不带缓冲的。

```c
#include <stdio.h>

void setbuf(FILE *restrict fp, char *restrict buf);
void setvbuf(FILE *restrict fp, char *restrict buf, int mode, size_t size);
/*
 * 返回值：若成功，返回0；若出错，返回非0
 */
```
这两个函数可用于更改缓冲类型。

- `_IOFBF` 全缓冲 
- `_IOLBF` 行缓冲
- `_IONBF` 不带缓冲

```c
#include <stdio.h>

int fflush(FILE *fp);
/* 返回值：若成功，返回0；若出错，返回EOF */
```

# 打开流
```c
#include <stdio.h>

FILE *fopen(const char *restrict ptahname, const char *restrict type);
FILE *freopen(const char *restrict pathname, const char *restrict type, FILE *restrict fp);
FILE *fdopen(int fd, const char *type);
/* 3个函数的返回值：若成功，返回文件指针；若出错，返回NULL */
```

1. `fopen`函数打开路径名为`pathname`的一个指定的文件。
2. `freopen`函数在一个指定的流上打开一个指定的文件，如若该流已经打开，则先关闭该流。
3. `fdopen`函数取一个已有的文件描述符，并使一个标准的I/O流与该描述符相结合。

调用`fclose`关闭一个打开的流。
```c
#include <stdio.h>

int fclose(FILE *fp);
/* 返回值：若成功，返回0；若返回，返回EOF */
```
**在该文件被关闭之前，冲洗缓冲区中的输出数据。**

# 读和写流

1. 输入函数
```c
#include <stdio.h>

int getc(FILE *fp);
int fgetc(FILE *fp);
int getchar(void);
/* 3个函数的返回值：若成功，返回下一个字符；若已到达文件尾端或出错，返回EOF */
```
函数`getchar`等同于`getc(stdin)`。前两个函数的区别是，`getc`可被实现为宏，而`fgetc`不能实现为宏。

```c
#include <stdio.h>

int ferror(FILE *fp);
int feof(FILE *fp);
/* 两个函数返回值：若条件为真，返回非0(真)；否则，返回0(假) */
void clearerr(FILE *fp);
```
为了区别是到达文件结尾还是出错。
例程
```c
#include <stdio.h>
int main(void)
{
    FILE*stream;
    /*openafileforwriting*/
    stream=
    fopen("DUMMY.FIL","w");
    /*forceanerrorconditionbyattemptingtoread*/
    (void)
    getc(stream);
    if(ferror(stream))/*testforanerroronthestream*/
    {
    /*displayanerrormessage*/
     
    printf("ErrorreadingfromDUMMY.FIL\n");
    /*resettheerrorandEOFindicators*/
     
    clearerr(stream);
    }
     
    fclose(stream);
    return0;
}
```

从流中读取数据后，可以调用`ungetc`将字符再压送回流中。
```c
#include <stdio.h>

int ungetc(int c, FILE *fp);
/* 返回值：若成功，返回c；若出错，返回EOF */
```

2. 输出函数
```c
#include <stdio.h>

int putc(int c, FILE *fp);
int fputc(int c, FILE *fp);
int putchar(int c);
/* 3个函数返回值：若成功，返回c;若出错，返回EOF */
```

# 每次一行I/O
下面两个函数提供每次输入一行的功能。
```c
#include <stdio.h>

char *fgets(char *restrict buf, int n, FILE *restrict fp);
char *gets(char *buf);
/* 两个函数返回值：若成功，返回buf；若已到达文件尾端或出错，返回NULL */
```

`fgets`函数一直读到下一个换行符为止，但是不超过n-1个字符，读入的字符被送入缓冲区。

`fputs`和`puts`提供每次输入一行的功能。
```c
#include <stdio.h>

int fputs(const char *restrict str, FILE *restrict fp);
int puts(const char *str);
/* 两个函数返回值：若成功，返回非负值；若出错，返回EOF */
```
函数`fputs`将一个以null字节终止的字符串写到指定的流，尾端的终止符null不写出。

# 二进制I/O

```c
#include <stdio.h>

size_t fread(void *restrict ptr, size_t size, size_t nboj, FILE *restrict fp);
size_t fwrite(const void *restrict ptr, size_t size, size_t nobj, FILE *restrict fp);
/* 两个函数的返回值：读或写的对象数 */
```
**指定size为每个数组元素的长度，nobj为欲写元素个数**
```c
/* 写数组 */
float data[10];

if (fwrite(&data[2], sizeof(float), 4, fp) != 4)
    err_sys("fwrite error");

/* 写一个结构体 */
struct {
    short   count;
    long    total;
    char    name[NAMESIZE];
} item;

if (fwrite(&item, sizeof(item), 1, fp) != 1)
    err_sys("fwrite error");
```
**使用二进制I/O的基本问题，它只能用于读在同一个系统上已写的数据**

# 定位流(Positioning a Stream)
```c
#include <stdio.h>

long ftell(FILE *fp);
/* 返回值：若成功，返回当前文件位置指示；若出错，返回-1L */

int fseek(FILE *fp, long offset, int whence);
/* 返回值：若成功，返回0；若出错，返回-1 */

void rewind(FILE *fp);
```

`whence`的值与`lseek`函数的相同：`SEEK_SET`表示从文件的起始位置开始，`SEEK_CUR`表示从当前文件位置开始，`SEEK_END`表示从文件的尾端开始。`offset`，文件偏移量。

```c
#include <stdio.h>

off_t ftello(FILE *fp);
/* 返回值：若成功，返回当前文件位置；若出错，返回(off_t)-1 */

int fseeko(FILE *fp, off_t offset, int whence);
/* 返回值：若成功，返回0；若出错，返回-1 */
```

```c
#include <stdio.h>

int fgetpos(FILE *restrict fp, fpos_t *restrict pos);
int fsetpos(FILE *fp, const fpos_t *pos);
/* 两个函数返回值：若成功，返回0；若出错，返回非0 */
```

# 格式化I/O
1. 格式化输出
```c
#include <stdio.h>

int printf(const char *restrict format, ...);
int fprintf(FILE *restrict fp, const char *restrict format, ...);
int dprintf(int fd, const char *restrict format, ...);
/* 3个函数返回值：若成功，返回输入字符数；若出错，返回负值 */

int sprintf(char *restrict buf, const char *restrict format, ...);
/* 返回值：若成功，返回输入数组的字符数；若编码出错，返回负值 */

int snprintf(char *restrict buf, size_t n, const char *restrict format, ...);
/* 返回值：若缓冲区足够大，返回将要存入数组的字符数；若编码出错，返回负值 */
```
**printf将格式化数据写到标准输出，fprintf写至指定的流，dprintf写至制定的文件描述符，sprintf将格式化的字符送入数组buf。**

下面5种`printf`族的变体，将可变参数表(...)替换成了arg。
```c
#include <stdarg.h>
#include <stdio.h>

int vprintf(const char *restrict format, va_list arg);
int vfprintf(FILE *restrict fp, const char *restrict format, va_list arg);
int vdprintf(int fd, const char *restrict format, va_list arg);
/* 3个函数返回值：若成功，返回输入字符数；若出错，返回负值 */

int vsprintf(char *restrict buf, const char *restrict format, va_list arg);
/* 返回值：若成功，返回输入数组的字符数；若编码出错，返回负值 */

int vsnprintf(char *restrict buf, size_t n, const char *restrict format, va_list arg);
/* 返回值：若缓冲区足够大，返回将要存入数组的字符数；若编码出错，返回负值 */
```

2. 格式化输入
```c
#include <stdio.h>

int scanf(const char *restrict format, ...);
int fscanf(FILE *restrict fp, const char *restrict format, ...);
int sscanf(const *restrict buf, const char *restrict format, ...);
/* 3个函数返回值：赋值的输入项数；若输入出错或在任一转换前已到达文件尾端，返回EOF */
```

# 临时文件
```c
#include <stdio.h>

char *tmpnam(char *ptr);
/* 返回值：指向唯一路径名的指针 */

FILE *tmpfile(void);
/* 返回值：若成功，返回文件指针；若出错，返回NULL */
```

`tmpnam`函数产生一个与现有文件名不同的一个有效路径名字符串。如果`ptr`是NULL，则产生的路径名存放在一个静态区中，指向该静态区的指针作为函数值返回。后续调用`tmpnam`会从写该静态区。如过`ptr`不是NULL，则认为它应该指向长度至少是`L_tmpnam`个字符的数组。所产生的路径名存放在该数组中，`ptr`也作为函数值返回。
`tmpfile`创建一个临时二进制文件(wb+)，在关闭该文件或程序结束时将自动删除这种文件。

```c
#include <stdlib.h>

char *mkdtemp(char *template);
/* 返回值：若成功，返回指向目录名的指针；若出错，返回NULL */

int mkstemp(char *template);
/* 返回值：若成功，返回文件描述符；若出错，返回-1 */
```
`mkdtemp`函数创建了一个目录，该目录有唯一的名字；`mkstemp`函数创建了一个文件，该文件有一个唯一的名字。**名字是通过template字符串进行选择的。这个字符串后6位设置为XXXXXX的路径名。**

# 内存流
```c
#include <stdio.h>

FILE *fmemopen(void *restrict buf, size_t size, const char *restrict type);
/* 返回值：若成功，返回流指针；若出错，返回NULL */
```

`fmemopen`函数允许调用者提供缓冲区用于内存流：`buf`参数指向缓冲区的开始位置，`size`参数指定了缓冲区大小的字节数。

```c
#include <stdio.h>
FILE *open_memstream(char **bufp, size_t *sizep);

#include <wchar.h>
FILE *open_wmemstream(wchar_t **bufp, size_t *sizep);
/* 两个函数的返回值：若成功，返回流指针；若出错，返回NULL */
```
`open_memstream`函数创建的流是面向字节的，`open_wmemstream`函数创建的流是面向宽字节的。

- 创建的流只能打开；
- 不能指定自己的缓冲区，但可以分别通过`bufp`和`sizep`参数访问缓冲区地址和大小；
- 关闭后需要自行释放缓冲区；
- 对流添加字节会增加缓冲区大小。
