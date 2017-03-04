title: 使用Xshell进行上传和下载
date: 2015-10-12 21:13:53
tags: [Xshell,Linux]
---

在日常使用Xshell进行远程登录的时候，我们想通过Xshell在Windows和Linux之间便捷的进行文件传输，这时候我们可以使用 `rz` 或 `sz` 命令。


在树莓派上安装`rz`和`sz`命令。

```
sudo apt-get install lrzsz
```


下载某个文件或文件夹
```
sz filename
```

`rz`直接把文件拖到Xshell上就可以了.

`sz`和`rz`的帮助文档
```
rz version 0.12.21rc
Usage: rz [options] [filename.if.xmodem]
Receive files with ZMODEM/YMODEM/XMODEM protocol
    (X) = option applies to XMODEM only
    (Y) = option applies to YMODEM only
    (Z) = option applies to ZMODEM only
  -+, --append                append to existing files
  -a, --ascii                 ASCII transfer (change CR/LF to LF)
  -b, --binary                binary transfer
  -B, --bufsize N             buffer N bytes (N==auto: buffer whole file)
  -c, --with-crc              Use 16 bit CRC (X)
  -C, --allow-remote-commands allow execution of remote commands (Z)
  -D, --null                  write all received data to /dev/null
      --delay-startup N       sleep N seconds before doing anything
  -e, --escape                Escape control characters (Z)
  -E, --rename                rename any files already existing
      --errors N              generate CRC error every N bytes (debugging)
  -h, --help                  Help, print this usage message
  -m, --min-bps N             stop transmission if BPS below N
  -M, --min-bps-time N          for at least N seconds (default: 120)
  -O, --disable-timeouts      disable timeout code, wait forever for data
      --o-sync                open output file(s) in synchronous write mode
  -p, --protect               protect existing files
  -q, --quiet                 quiet, no progress reports
  -r, --resume                try to resume interrupted file transfer (Z)
  -R, --restricted            restricted, more secure mode
  -s, --stop-at {HH:MM|+N}    stop transmission at HH:MM or in N seconds
  -S, --timesync              request remote time (twice: set local time)
      --syslog[=off]          turn syslog on or off, if possible
  -t, --timeout N             set timeout to N tenths of a second
      --tcp-server            open socket, wait for connection (Z)
      --tcp-client ADDR:PORT  open socket, connect to ... (Z)
  -u, --keep-uppercase        keep upper case filenames
  -U, --unrestrict            disable restricted mode (if allowed to)
  -v, --verbose               be verbose, provide debugging information
  -w, --windowsize N          Window is N bytes (Z)
  -X  --xmodem                use XMODEM protocol
  -y, --overwrite             Yes, clobber existing file if any
      --ymodem                use YMODEM protocol
  -Z, --zmodem                use ZMODEM protocol

short options use the same arguments as the long ones
```