title: 关于树莓派lirc的配置
date: 2015-10-31 21:10:19
tags: [Linux,lirc,树莓派]
---

## 记录原因
这是我正在DIY的一个[基于树莓派智能家居](https://github.com/kaiiak/NineSky/blob/master/README.md)的过程中，配置lirc的一个小问题的总结。
因为在网上搜索到的类似的博客给出的解决方式都不可行，所以打算记录下来，方便后来者。

## 事情的起因
安装lirc——一个开源的红外控制的库
```shell
sudo apt-get install lirc
```
配置到这一步时，网络上给出的下一步都是在
>加载内核模块
>```shell
sudo modprobe lirc_rpi gpio_in_pin=23gpio_out_pin=24
```
然而照着做以后，并不能成功。

## 解决方法
配置树莓派的lirc，需要在`/boot/config.txt`中添加，在[这里](https://github.com/raspberrypi/firmware/blob/master/boot/overlays/README)找到的。
```
dtoverlay=lirc-rpi,gpio_in_pin=23,gpio_out_pin=24
```
然后重启就可以了。

加载lir_rpi内核模块
```shell
sudo modprobe lirc_rpi
```
测试
```shell
sudo mode2 -d /dev/lirc0
```
现在就会看到一大串space和pulse交替产生。
