---
title: 基于Agones的游戏服务器架构
date: 2021-02-17 23:34:14
tags: [K8S,AGONES,GAMESERVER]
---
https://mp.weixin.qq.com/s/nmZXg-5bt3SNxtS_Tyy4Fw
我们游戏按照业务逻辑划分，服务器可分为三种类型，前端服务器（客户端直接进行连接的）、后端服务器（只负责处理各种游戏逻辑不提供连接）、任务服务器（各种 cron、job 任务），其中前端服务器按照功能划分为 http 短连接服务器和 socket 长连接服务器，后端服务器按照业务划分 例如 matching 匹配服务器。

```bash
kubectl create secret docker-registry aliyun.com -n agones-system --docker-server=registry.cn-hangzhou.aliyuncs.com --docker-username=yoock@outlook.com --docker-password=XmVs0xgcGZ6IWerV --docker-email=yoock@outlook.com
```

https://agones.dev/site/docs/installation/install-agones/yaml/

https://agones.dev/site/docs/installation/install-agones/helm/

XmVs0xgcGZ6IWerV


