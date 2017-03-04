title: 解决Yilia主题的subnav中链接的问题
date: 2015-08-01 00:11:23
tags: Hexo主题
---
## 问题说明
我很喜欢Yilia这个漂亮的主题，但是当我自己用的时候发现在subnav区生成的链接并不是我想要的。比如我的github地址是`github.com/kaiiak`，但是实际生成的是`kaiiak.github.io/github.com/kaiiak`。微博地址也是这样，邮箱地址也是。

## 心路历程
我一点一点试验嘛，今天一晚上翻来覆去改了有30多次。

```html
subnav:
  github: "https://github.com/kaiiak"
  weibo:  "http://weibo.com/itkaikai"
  #mail:   "itkaikai@gmail.com"
  ```
  这时可以了，我以为是因为链接对齐了呢，可是我把mail那里的`#`去掉后，发现邮箱那一栏生成的地址还是没变啊。我又陷入了沉思～

## 问题解决
解决的原因是我把git生成的diff看了一遍，发现微博那一栏在没加`http://`时，生成的html代码是这样的`/weibo.com/itkaikai`，加了`http://`后生成的html代码就变成了`http://weibo.com/itkaikai`了。看到这里，大家一定会明白了。
把subnav改成这样子就可以了。
```html
subnav:
  github: "https://github.com/kaiiak"
  weibo:  "http://weibo.com/itkaikai"
  mail:   "mailto:itkaikai@gmail.com"
```

## 后记
把时间花费在打磨工具上有点很吃亏啊，还是等有钱买个vps好了。
晚安～