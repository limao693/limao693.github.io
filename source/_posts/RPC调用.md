---
title: RPC调用
urlname: etrzfl
date: 2019-06-03 11:41:05 +0800
tags: [rpc,HTTP]
categories: []
---

### 知识点

#### Target

an RPC Server's target:
       topic and server is required; exchange is optional
an RPC endpoint's target:
       namespace and version is optional
an RPC client sending a message:
       topic is required, all other attributes optional
a Notification Server's target:
       topic is required, exchange is optional; all other attributes ignored
a Notifier's target:
       topic is required, exchange is optional; all other attributes ignored
