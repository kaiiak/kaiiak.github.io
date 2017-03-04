title: 使用Python控制树莓派的GPIO(1)
date: 2015-10-18 17:19:07
tags: [Pyhon,树莓派,PRI.GPIO]
---
### 使用的树莓派2B

通过PRI.GPIO来实现Python控制树莓派的GPIO。

## 安装

在[官方文档](http://pythonhosted.org/RPIO/)中给出了三种按章方式。分别是

### 用 `easy_install`或者`pip`来安装：
```shell
sudo apt-get install python-setuptools
sudo easy_install -U RPIO
```

### 从Github上克隆然后安装
```shell
git clone https://github.com/metachris/RPIO.git
cd RPIO
sudo python setup.py install
```

### 从Github或者别处下载安装
```shell
curl -L https://github.com/metachris/RPIO/archive/master.tar.gz | tar -xz
cd RPIO-master
sudo python setup.py install
```

## 使用

![树莓派引脚图](http://ww1.sinaimg.cn/mw690/ae94c92cgw1ex5fscddh0j20fi0bigqz.jpg)
![树莓派2B引脚图](http://ww3.sinaimg.cn/mw690/ae94c92cgw1ex6qv7sjpoj20jq0c6gps.jpg)


这是用wiringPi生成的引脚图，用wiringPi控制GPIO会在以后写。

树莓派的GPIO大致可以分为INPUT和OUTPUT两种状态。

```Python
import RPIO

# 设置为输入位无上拉
RPIO.setup(7, RPIO.IN)

# 设置为输出位有上拉. 可以设置为
# PUD_UP(上拉), PUD_DOWN(下拉) or PUD_OFF (default)
RPIO.setup(7, RPIO.IN, pull_up_down=RPIO.PUD_UP)

# 读取GPIO7的输入状态
input_value = RPIO.input(7)

# 设置GPIO为输出位
RPIO.setup(8, RPIO.OUT)

# 设置GPIO8为高电位
RPIO.output(8, True)

# 设置为输出位并给予一个初始值
RPIO.setup(8, RPIO.OUT, initial=RPIO.LOW)

# 改变为BOARD编号模式
RPIO.setmode(RPIO.BOARD)

# 在通道17上设置软件上拉
RPIO.set_pullupdn(17, RPIO.PUD_UP)  # new in RPIO

# 获得通道8的设置(IN、OUT、ALTo)
RPIO.gpio_function(8)

# 复位所有由该程序设置过的通道，
# 并清除 GPIO 中断接口
RPIO.cleanup()
```

照着上一篇文章写的那样做，并不能成功。因为RPIO的最后一个release版本是2013年的，并不支持我的树莓派2B。

如果想在树莓派2B上运行，需要做这些工作
```shell
sudo apt-get install rpi.gpio
sudo pip freeze
```
如果有一行`RPi.GPIO==0.511`就说明安装成功了。

然后我们在新建`led.py`：
```Python
import RPi.GPIO as GPIO
import time

LED = 4 

GPIO.setmode(GPIO.BCM)
GPIO.setup(LED, GPIO.OUT)

print("结束程序，请按CTRL+C")
try:
	while 1:
		GPIO.output(LED, False)
		#延时1s
		time.sleep(1)
		GPIO.output(LED, True)
		time.sleep(1)
except KeyboardInterrupt: # 如果程序被CTRL+C结束
	GPIO.cleanup()
```

前几天搞不懂GPIO.setmode函数中的参数GPIO.BCM和GPIO.BOARD指的是什么。
后来知道BOARD指的是主板引脚编号，而BCM指的是BCM芯片的引脚编号，在本文的配图中有。
