title: tree命令源码分析
date: 2016-03-27 17:30:30
tags: [Linux,tree]
---
# 为什么选择tree这个命令？
学完APUE的前几章，学完了IO和文件与目录，我们可以做点东西出来了。而借鉴别人的程序就是做出东西的选择之一。
# 如何下载源代码
[http://mama.indstate.edu/users/ice/tree/src/tree-1.7.0.tgz](http://mama.indstate.edu/users/ice/tree/src/tree-1.7.0.tgz])这个链接可以下载到最新的tree源代码，也可以编译成功。
# 怎么分析源代码
这是个很大的问题，暂时我还没有太多的经验 :(
# 我们的目的
我们的目的暂时定为搞懂如何遍历所有的目录，并输出。
# 各部分的作用
```
tree-1.7.0
├── color.c
├── hash.c
├── html.c
├── INSTALL
├── json.c
├── LICENSE
├── Makefile
├── strverscmp.c
├── tree.c
├── tree.h
├── unix.c
└── xml.c
```
核心的就是这几个文件。其中`unix.c`和`tree.c`是遍历功能的主要函数的所在地。
## Makefile
```mk
prefix = /usr

CC=gcc

VERSION=1.7.0
TREE_DEST=tree
BINDIR=${prefix}/bin
MAN=tree.1
MANDIR=${prefix}/man/man1
OBJS=tree.o unix.o html.o xml.o json.o hash.o color.o

# Uncomment options below for your particular OS:

# Linux defaults:
CFLAGS=-ggdb -Wall -DLINUX -D_LARGEFILE64_SOURCE -D_FILE_OFFSET_BITS=64
#CFLAGS=-O4 -Wall  -DLINUX -D_LARGEFILE64_SOURCE -D_FILE_OFFSET_BITS=64
#LDFLAGS=-s

#------------------------------------------------------------

all:    tree

tree:   $(OBJS)
    $(CC) $(LDFLAGS) -o $(TREE_DEST) $(OBJS)

$(OBJS): %.o:   %.c tree.h
    $(CC) $(CFLAGS) -c -o $@ $<

clean:
    if [ -x $(TREE_DEST) ]; then rm $(TREE_DEST); fi
    if [ -f tree.o ]; then rm *.o; fi
    rm -f *~

install: tree
    install -d $(BINDIR)
    install -d $(MANDIR)
    if [ -e $(TREE_DEST) ]; then \
        install $(TREE_DEST) $(BINDIR)/$(TREE_DEST); \
    fi
    install doc/$(MAN) $(MANDIR)/$(MAN)

distclean:
    if [ -f tree.o ]; then rm *.o; fi
    rm -f *~
    

dist:   distclean
    tar zcf ../tree-$(VERSION).tgz -C .. `cat .tarball`

```

## 核心程序-遍历目录

```c
//存储文件和文件夹信息的结构体
struct _info {
  char *name;   /
  char *lnk;
  bool isdir;
  bool issok;
  bool isfifo;
  bool isexe;
  bool orphan;
  mode_t mode, lnkmode;
  uid_t uid;
  gid_t gid;
  off_t size;
  time_t atime, ctime, mtime;
  dev_t dev;
  ino_t inode;
  #ifdef __EMX__
  long attr;
  #endif
  char *err;
  struct _info **child;
};
```

```c
off_t unix_listdir(char *d, int *dt, int *ft, u_long lev, dev_t dev)
{
  char *path;
  bool nlf = FALSE, colored = FALSE;
  long pathsize = 0;
  struct _info **dir, **sav;
  struct stat sb;
  int n, c;
  
  if ((Level >= 0) && (lev > Level)) {
    fputc('\n',outfile);
    return 0;
  }
  
  if (xdev && lev == 0) {
    stat(d,&sb);
    dev = sb.st_dev;
  }
  
  sav = dir = read_dir(d,&n);
  if (!dir && n) {
    fprintf(outfile," [error opening dir]\n");
    return 0;
  }
  if (!n) {
    fputc('\n', outfile);
    free_dir(sav);
    return 0;
  }
  if (flimit > 0 && n > flimit) {
    fprintf(outfile," [%d entries exceeds filelimit, not opening dir]\n",n);
    free_dir(sav);
    return 0;
  }

  if (cmpfunc) qsort(dir,n,sizeof(struct _info *), cmpfunc);
  if (lev >= maxdirs-1) {
    dirs = xrealloc(dirs,sizeof(int) * (maxdirs += 1024));
    memset(dirs+(maxdirs-1024), 0, sizeof(int) * 1024);
  }
  dirs[lev] = 1;
  if (!*(dir+1)) dirs[lev] = 2;
  fprintf(outfile,"\n");
  
  path = malloc(pathsize=4096);
  
  while(*dir) {
    if (!noindent) indent(lev);

    fillinfo(path,*dir);
    if (path[0] == ' ') {
      path[0] = '[';
      fprintf(outfile, "%s]  ",path);
    }
    
    if (colorize) {
      if ((*dir)->lnk && linktargetcolor) colored = color((*dir)->lnkmode,(*dir)->name,(*dir)->orphan,FALSE);
      else colored = color((*dir)->mode,(*dir)->name,(*dir)->orphan,FALSE);
    }
    
    if (fflag) {
      if (sizeof(char) * (strlen(d)+strlen((*dir)->name)+2) > pathsize)
    path=xrealloc(path,pathsize=(sizeof(char) * (strlen(d)+strlen((*dir)->name)+1024)));
      if (!strcmp(d,"/")) sprintf(path,"%s%s",d,(*dir)->name);
      else sprintf(path,"%s/%s",d,(*dir)->name);
    } else {
      if (sizeof(char) * (strlen((*dir)->name)+1) > pathsize)
    path=xrealloc(path,pathsize=(sizeof(char) * (strlen((*dir)->name)+1024)));
      sprintf(path,"%s",(*dir)->name);
    }

    printit(path);

    if (colored) fprintf(outfile,"%s",endcode);
    if (Fflag && !(*dir)->lnk) {
      if ((c = Ftype((*dir)->mode))) fputc(c, outfile);
    }
    
    if ((*dir)->lnk) {
      fprintf(outfile," -> ");
      if (colorize) colored = color((*dir)->lnkmode,(*dir)->lnk,(*dir)->orphan,TRUE);
      printit((*dir)->lnk);
      if (colored) fprintf(outfile,"%s",endcode);
      if (Fflag) {
    if ((c = Ftype((*dir)->lnkmode))) fputc(c, outfile);
      }
    }
    
    if ((*dir)->isdir) {
      if ((*dir)->lnk) {
    if (lflag && !(xdev && dev != (*dir)->dev)) {
      if (findino((*dir)->inode,(*dir)->dev)) {
        fprintf(outfile,"  [recursive, not followed]");
      } else {
        saveino((*dir)->inode, (*dir)->dev);
        if (*(*dir)->lnk == '/')
          listdir((*dir)->lnk,dt,ft,lev+1,dev);
        else {
          if (strlen(d)+strlen((*dir)->lnk)+2 > pathsize) path=xrealloc(path,pathsize=(strlen(d)+strlen((*dir)->name)+1024));
          if (fflag && !strcmp(d,"/")) sprintf(path,"%s%s",d,(*dir)->lnk);
          else sprintf(path,"%s/%s",d,(*dir)->lnk);
          listdir(path,dt,ft,lev+1,dev);
        }
        nlf = TRUE;
      }
    }
      } else if (!(xdev && dev != (*dir)->dev)) {
    if (strlen(d)+strlen((*dir)->name)+2 > pathsize) path=xrealloc(path,pathsize=(strlen(d)+strlen((*dir)->name)+1024));
    if (fflag && !strcmp(d,"/")) sprintf(path,"%s%s",d,(*dir)->name);
    else sprintf(path,"%s/%s",d,(*dir)->name);
    saveino((*dir)->inode, (*dir)->dev);
    listdir(path,dt,ft,lev+1,dev);
    nlf = TRUE;
      }
      *dt += 1;
    } else *ft += 1;
    if (*(dir+1) && !*(dir+2)) dirs[lev] = 2;
    if (nlf) nlf = FALSE;
    else fprintf(outfile,"\n");
    dir++;
  }
  dirs[lev] = 0;
  free(path);
  free_dir(sav);
  return 0;
}

off_t unix_rlistdir(char *d, int *dt, int *ft, u_long lev, dev_t dev)
{
  struct _info **dir;
  off_t size = 0;
  char *err;
  
  dir = getfulltree(d, lev, dev, &size, &err);

  memset(dirs, 0, sizeof(int) * maxdirs);

  r_listdir(dir, d, dt, ft, lev);

  return size;
}

void r_listdir(struct _info **dir, char *d, int *dt, int *ft, u_long lev)
{
  char *path;
  long pathsize = 0;
  struct _info **sav = dir;
  bool nlf = FALSE, colored = FALSE;
  int c;
  
  if (dir == NULL) return;

  dirs[lev] = 1;
  if (!*(dir+1)) dirs[lev] = 2;
  fprintf(outfile,"\n");

  path = malloc(pathsize=4096);

  while(*dir) {
    if (!noindent) indent(lev);
    
    fillinfo(path,*dir);
    if (path[0] == ' ') {
      path[0] = '[';
      fprintf(outfile, "%s]  ",path);
    }
    
    if (colorize) {
      if ((*dir)->lnk && linktargetcolor) colored = color((*dir)->lnkmode,(*dir)->name,(*dir)->orphan,FALSE);
      else colored = color((*dir)->mode,(*dir)->name,(*dir)->orphan,FALSE);
    }
    
    if (fflag) {
      if (sizeof(char) * (strlen(d)+strlen((*dir)->name)+2) > pathsize)
    path=xrealloc(path,pathsize=(sizeof(char) * (strlen(d)+strlen((*dir)->name)+1024)));
      if (!strcmp(d,"/")) sprintf(path,"%s%s",d,(*dir)->name);
      else sprintf(path,"%s/%s",d,(*dir)->name);
    } else {
      if (sizeof(char) * (strlen((*dir)->name)+1) > pathsize)
    path=xrealloc(path,pathsize=(sizeof(char) * (strlen((*dir)->name)+1024)));
      sprintf(path,"%s",(*dir)->name);
    }
    
    printit(path);
    
    if (colored) fprintf(outfile,"%s",endcode);
    if (Fflag && !(*dir)->lnk) {
      if ((c = Ftype((*dir)->mode))) fputc(c, outfile);
    }
    
    if ((*dir)->lnk) {
      fprintf(outfile," -> ");
      if (colorize) colored = color((*dir)->lnkmode,(*dir)->lnk,(*dir)->orphan,TRUE);
      printit((*dir)->lnk);
      if (colored) fprintf(outfile,"%s",endcode);
      if (Fflag) {
    if ((c = Ftype((*dir)->lnkmode))) fputc(c, outfile);
      }
    }
    
    if ((*dir)->err) {
      fprintf(outfile," [%s]", (*dir)->err);
      free((*dir)->err);
      (*dir)->err = NULL;
    }
    if ((*dir)->child) {
      if (fflag) {
    if (strlen(d)+strlen((*dir)->name)+2 > pathsize) path=xrealloc(path,pathsize=(strlen(d)+strlen((*dir)->name)+1024));
    if (!strcmp(d,"/")) sprintf(path,"%s%s",d,(*dir)->name);
    else sprintf(path,"%s/%s",d,(*dir)->name);
      }
      r_listdir((*dir)->child, fflag? path : NULL, dt, ft, lev+1);
      nlf = TRUE;
      *dt += 1;
    } else {
      if ((*dir)->isdir) *dt += 1;
      else *ft += 1;
    }

    if (*(dir+1) && !*(dir+2)) dirs[lev] = 2;
    if (nlf) nlf = FALSE;
    else fprintf(outfile,"\n");
    dir++;
  }
  dirs[lev] = 0;
  free(path);
  free_dir(sav);
}

```
