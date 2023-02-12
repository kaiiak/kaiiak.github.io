---
title: NSQ环境的搭建和简单的使用
date: 2018-07-25 22:47:30
tags: [Nsq,Docker]
---

## NSQ安装说明

### 使用docker-compose运行
如果使用`docker-compose`的方式运行`nsq`,首先要保证`Consumer`要与nsq服务在一个docker网络环境中。
如果不能保证在一个网络环境中，则需要从源代码安装。
如果要存储`nsq`的消息，需要在启动`nsqd`的时候加上`--data-path=/data`命令，并在`docker`配置文件中将`/data`使用`-v`或`--volume`命令挂载到本地。
使用`docker-compose up -d`后台运行。

```yaml
version: '3'

services:
  nsqlookupd:
    image: nsqio/nsq
    networks:
      - nsq-network
    hostname: nsqlookupd
    ports:
      - "4161:4161"
      - "4160:4160"
    command: /nsqlookupd
  nsqd:
    image: nsqio/nsq
    depends_on:
      - nsqlookupd
    hostname: nsqd
    volumes:
      - ./data:/data
    networks:
      - nsq-network
    ports:
      - "4151:4151"
      - "4150:4150"
    command: /nsqd --broadcast-address=nsqd --lookupd-tcp-address=nsqlookupd:4160 --data-path=/data
  nsqadmin:
    image: nsqio/nsq
    depends_on:
      - nsqlookupd
    hostname: nsqadmin
    ports:
      - "4171:4171"
    networks:
      - nsq-network
    command: /nsqadmin --lookupd-http-address=nsqlookupd:4161

networks:
  nsq-network:
    driver: bridge
```

### 运行的独立的docker
1. 从dokcer hub拉取最新的镜像`docker pull nsqio/nsq`
2. 运行`nsqlookupd`
```shell
docker run --name lookupd -p 4160:4160 -p 4161:4161 nsqio/nsq /nsqlookupd
```
3. 运行`nsqd`,需要手动挂载`volume`
```shell
docker run -v ./data:/data --name nsqd -p 4150:4150 -p 4151:4151 \
    nsqio/nsq /nsqd \
    --broadcast-address=127.0.0.1 \
    --lookupd-tcp-address=127.0.0.1:4160 \
    --data-path=/data
```
4. 运行`nsqadmin`
```shell
docker run --name nsqadmin -p 4171:4171 \
    nsqio/nsq /nsqadmin \
    --lookupd-http-address=127.0.0.1:4161
```

### 从源代码安装

```shell
git clone https://github.com/nsqio/nsq $GOPATH/src/github.com/nsqio/nsq
cd $GOPATH/src/github.com/nsqio/nsq
dep ensure
go install
```

#### 启动nsqlookupd

```shell
nsqlookupd
```
#### 启动nsq

```shell
nsqd –lookupd-tcp-address=127.0.0.1:4160 --broadcast-address=127.0.0.1 --data-path=/data 
```

#### 启动nsqadmin

```shell
nsqadmin --lookupd-http-address=127.0.0.1:4161
```
