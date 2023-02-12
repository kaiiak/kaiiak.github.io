title: Golang踩坑小结之SMTP
date: 2017-03-01 00:16:30
tags: [Golang,Smtp]
---
# smtp之golang标准库
函数`SendMail`就是标准库里面最简单的发送邮件的方法。
```golang
func SendMail(addr string, a Auth, from string, to []string, msg []byte) error
```
对不熟悉smtp协议的小伙伴来说，最难的地方就是构造`msg`了。
在指定`smtp`的[RFC531](https://tools.ietf.org/html/rfc5321)中，定义了`smtp`客户端和服务器的通讯方式和报文的格式。

## 缺陷
现在的邮件服务器除了`25`端口外，还采用了其他的默认采用TLS的端口（只能TLS连接）。
但是标准库还会发送`STARTTLS`命令，但是服务器不会回应，这样就产生了阻塞。
以下是直接通过tls端口发送邮件的代码。

```golang
// SendMailTLS not use STARTTLS commond
func SendMailTLS(addr string, auth smtp.Auth, from string, to []string, msg []byte) error {
	host, _, err := net.SplitHostPort(addr)
	if err != nil {
		return err
	}
	tlsconfig := &tls.Config{ServerName: host}
	if err = validateLine(from); err != nil {
		return err
	}
	for _, recp := range to {
		if err = validateLine(recp); err != nil {
			return err
		}
	}
	conn, err := tls.Dial("tcp", addr, tlsconfig)
	if err != nil {
		return err
	}
	defer conn.Close()
	c, err := smtp.NewClient(conn, host)
	if err != nil {
		return err
	}
	defer c.Close()
	if err = c.Hello("localhost"); err != nil {
		return err
	}
	if err = c.Auth(auth); err != nil {
		return err
	}
	if err = c.Mail(from); err != nil {
		return err
	}
	for _, addr := range to {
		if err = c.Rcpt(addr); err != nil {
			return err
		}
	}
	w, err := c.Data()
	if err != nil {
		return err
	}
	_, err = w.Write(msg)
	if err != nil {
		return err
	}
	err = w.Close()
	if err != nil {
		return err
	}
	return c.Quit()
}

// validateLine checks to see if a line has CR or LF as per RFC 5321
func validateLine(line string) error {
	if strings.ContainsAny(line, "\n\r") {
		return errors.New("a line must not contain CR or LF")
	}
	return nil
}
```

# MIME
> 多用途互联网邮件扩展（MIME，Multipurpose Internet Mail Extensions）是一个互联网标准，它扩展了电子邮件标准，使其能够支援：
>非ASCII字符文本；
>非文本格式附件（二进制、声音、图像等）；
>由多部分（multiple parts）组成的消息体；
>包含非ASCII字符的头信息（Header information）。这个标准被定义在RFC 2045、RFC 2046、RFC 2047、RFC 2048、RFC 2049等RFC中。
>MIME改善了由RFC 822转变而来的RFC 2822，这些旧标准规定电子邮件标准并不允许在邮件消息中使用7位ASCII字符集以外的字符。正因如此，一些非英语字符消息和二进制文件，图像，声音等非文字消息原本都不能在电子邮件中传输（MIME可以）。MIME规定了用于表示各种各样的数据类型的符号化方法。此外，在万维网中使用的HTTP协议中也使用了MIME的框架，标准被扩展为互联网媒体类型。

## 基本格式

```
MIME-Version: 1.0
Content-Type: text/plain
Content-transfer-encoding: base64
boundary = XXXXXXXXXXXXXXXXX
--XXXXXXXXXXXXXXXXX
base64 string
--XXXXXXXXXXXXXXXXX
base64 string

```

# 邮件格式

## 基本字段
```
From:来自测试发送<aNxFi37X@outlook.com>
To:aNxFi37X@outlook.com
Date:27 Mar 17 00:45 +0800
Cc:aNxFi37X@outlook.com
Subject: TEST
```
## bcc
密送的实现很好玩，在邮件内容里不填写信息，但是发送的时候发送给密送的那个人。
这里的设计很巧妙。

## <CRLF>
以上的每一项都以`<CRLF>`结束，即`\r\n`。

## 邮件内容
这里使用MIME，当然也可以使用`8BITMIME`发送多媒体内容。
具体的方式是创建一个MIME part，每个part里面设置自己的`Content-Disposition`、`Content-Transfer-Encoding`、`Content-Type`。
`Content-Transfer-Encoding`使用base64编码，不然直接发送`ascii`中含有`\r\n.\r\n`，邮件就直接结束了。
虽然编码率高了不少，但是简单，易操作，不依赖服务器支持`8BITMIME`。

## 邮件样例
```
From:来自测试发送<aNxFi37X@outlook.com>
To:aNxFi37X@outlook.com
Date:27 Mar 17 00:45 +0800
Cc:aNxFi37X@outlook.com
Subject:=?UTF-8?B?5rWL6K+V?=
Content-Type: multipart/mixed; boundary=5b83f06665140150554c5847a8a3b05a5a9d64f5ec6a42fec16a4460223c


MIME-Version: 1.0
--5b83f06665140150554c5847a8a3b05a5a9d64f5ec6a42fec16a4460223c
Content-Transfer-Encoding: base64
Content-Type: text/html

PHA+6L+Z5piv5LiA5bCB5rWL6K+V6YKu5Lu2PC9wPg==
--5b83f06665140150554c5847a8a3b05a5a9d64f5ec6a42fec16a4460223c
Content-Disposition: attachment; filename="=?UTF-8?B?MS50eHQ=?="
Content-Transfer-Encoding: base64
Content-Type: application/application/octet-stream

MS50eHQ=

--5b83f06665140150554c5847a8a3b05a5a9d64f5ec6a42fec16a4460223c
Content-Disposition: attachment; filename="=?UTF-8?B?Mi50eHQ=?="
Content-Transfer-Encoding: base64
Content-Type: application/application/octet-stream

Mi50eHQ=

```

# 发送过程
```
S: 220 xyz.com Simple Mail Transfer Service Ready
C: EHLO foo.com
S: 250 xyz.com is on the air
C: MAIL FROM:<@foo.com:JQP@bar.com>
S: 250 OK
C: RCPT TO:<Jones@XYZ.COM>
S: 250 OK
C: DATA
S: 354 Start mail input; end with <CRLF>.<CRLF>
C: Received: from bar.com by foo.com ; Thu, 21 May 1998
C:     05:33:29 -0700
C: Date: Thu, 21 May 1998 05:33:22 -0700
C: From: John Q. Public <JQP@bar.com>
C: Subject:  The Next Meeting of the Board
C: To: Jones@xyz.com
C:
C: Bill:
C: The next meeting of the board of directors will be
C: on Tuesday.
C:                         John.
C: .
S: 250 OK
C: QUIT
S: 221 foo.com Service closing transmission channel
```
## HELO or EHLO
确定服务器可用

## MAIL
声明发信邮箱

## RCPT
声明收件邮箱。密送的邮箱就是在这里声明，但是不写在邮件内容里面。

## DATA
发送邮件的内容。

## QUIT
结束发送，并关闭连接。

# 拓展命令

## STARTLTLS
试探服务器是否支持TLS。

## 8BITMIME
试探服务器是否支持8字长的编码。

## 其他
其他的命令没有使用，这里挖个坑。

<center>
一篇文章从一号拖到现在，真的好丢人啊。
多谢您花费时间阅读我的文章，有不好的地方欢迎指正。
</center>