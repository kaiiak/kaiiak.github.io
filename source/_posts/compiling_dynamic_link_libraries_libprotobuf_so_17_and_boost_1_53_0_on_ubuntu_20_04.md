---
title: 在Ubuntu上编译libprotobuf.so.17和boost1.53
tags: [Ubuntu,C++,gcc,编译,protobuf,boost,docker]
---

最近在迁移服务器的时候，涉及到操作系统的替换和升级，导致一些C++依赖的项目无法在新系统中运行。由于某些原因，无法在新环境编译旧的服务，只能安装旧的依赖。
在Ubuntu20.04中，没有那么旧的依赖可以直接安装，只能自己编译。因为Gcc5.x中添加了对某些C++11的支持，更改了ABI，所以必须使用4.X编译依赖库，需要使用Ubuntu14.04。
下面是编译使用的`Dockerfile`。

## libprotobuf.so.17.0

```Dockerfile
FROM ubuntu:14.04

RUN apt-get update \
  && apt-get install -y git \
                        g++ \
                        make \
                        wget \
                        autoconf \
                        libtool \
                        automake

# 文档地址：https://github.com/protocolbuffers/protobuf/tree/v3.6.1.3/src
# 如果因为网络原因导致压缩包下载缓慢，可以下载到本地再使用COPY命令，复制Docker中
# COPY protobuf-3.6.1.3.tar.gz /home
RUN cd /home && wget https://github.com/protocolbuffers/protobuf/archive/refs/tags/v3.6.1.3.tar.gz \
  && tar xfz protobuf-3.6.1.3.tar.gz \
  && rm protobuf-3.6.1.3.tar.gz \
  && cd protobuf-3.6.1.3 \
  && ./autogen.sh \
  && ./configure --prefix=/usr/ \
  && make
# make install 不成功, 启动Container，需要进入内部make install
# RUN cd /home/protobuf-3.6.1.3 \
#   && make intall \
#   && cd /home \
#   && rm -rf protobuf-3.6.1.3

CMD ["bash"]
```
### boost 1.53.0
例如`boost_thread-mt.so.1.53.0`直接可以软连接到`boost_thread.so.1.53.0`上。

```Dockerfile
FROM ubuntu:14.04

RUN apt-get update \
  && apt-get install -y git \
                        g++ \
                        make \
                        wget

# 如果因为网络原因导致压缩包下载缓慢，可以下载到本地再使用COPY命令，复制Docker中
# COPY boost_1_53_0.tar.gz /home
RUN cd /home && wegt http://downloads.sourceforge.net/project/boost/boost/1.53.0/boost_1_53_0.tar.gz \
  && tar xfz boost_1_53_0.tar.gz \
  && rm boost_1_53_0.tar.gz \
  && cd boost_1_53_0 \
  && ./bootstrap.sh --prefix=/usr/local --with-libraries=program_options,regex,date_time,filesystem,system,thread \
  && ./b2 install \
  && cd /home \
  && rm -rf boost_1_53_0

CMD ["bash"]
```

## 复制文件
- 使用`docker run`启动容器以后，再使用`docker cp`命令复制容器中的文件
- 使用`docker save`命令压缩`image`，然后解压缩
