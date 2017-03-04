title: 编译apue.3e并解决编译错误
date: 2016-01-28 22:38:58
tags: [Linux,APUE]
---
## 下载源代码
我们在学习APUE这本书的时候，总会见到书上的源代码总会引用`apue.h`这个头文件，这个并不是Linux系统自带的。所以，我们要自己编译。
我们在[http://www.apuebook.com/src.3e.tar.gz](http://www.apuebook.com/src.3e.tar.gz)下载源代码后，解压。
```shell
tar -zxvf src.3e.tar.gz
```
然后编译
```shell
sudo make
```
但是在编译过程中，会出现错误，错误部分的log是这样的。
```
gcc -ansi -I../include -Wall -DLINUX -D_GNU_SOURCE  badexit2.c -o badexit2  -L../lib -lapue -pthread -lrt -lbsd
/usr/bin/ld: cannot find -lbsd
collect2: ld returned 1 exit status
Makefile:31: recipe for target 'badexit2' failed
make[1]: *** [badexit2] Error 1
make[1]: Leaving directory '/home/pi/apue.3e/threads'
Makefile:6: recipe for target 'all' failed

make: *** [all] Error 1
```

## 修复
出现这个错误是因为，我们缺少一个库。
```shell
sudo apt-get inatall libbsd-dev
```

安装完成后，重新进行编译.

```
cp ./include/apue.h ./lib/error.c /usr/include
```
然后，在`/usr/include/apue.h`的`ifdef`和`endif`中间添加`#include "error.c"`,就可以在编写的程序中，愉快的使用`apue.h`了。