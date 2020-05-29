---
title: 测试语雀-Travis-Hexo
urlname: dbxh9r
date: 2019-01-29 00:16:15 +0800
tags: []
categories: []
---

## 信息来源

本次语雀到 Travis 的配置，来源于：
[https://segmentfault.com/a/1190000017797561#articleHeader1](https://segmentfault.com/a/1190000017797561#articleHeader1)

仅分享搭建历程，及代码分析。

## TEST 步骤

第一次，init 测试。检测是否同步信息，有效期 5 分钟。过期删除
第二次测试，时间 5 分钟
第三次测试，时间 3 分钟
第四次测试，时间 3 分钟，仅测试 POSTMan 触发请求，查看语雀是否会同步至 github。失败
第五次测试，测试 yuque-hexo 组件 clean/sync 命令的先后顺序。
第六次测试，测试 yuque-hexo 组件，先 clean 后执行 sync 命令。
第七次测试，测试 POSTMan 触发请求。
第八次测试，测试 serverless 函数的正确性。
第九次测试，测试 serverless 函数的正确性。
第十次测试，测试 serverless 函数的正确性,参数进行硬编码。
第十一次测试，测试 serverless 函数的正确性.
第十二次测试，测试 serverless 函数的正确性.
第十二次测试，测试 serverless 函数的正确性,测试返回值。
第十三次测试，测试 Travis pull queest 的设置是否阻止的触发。
第十四次测试，测试 serverless 函数的正确性,移除触发 body 体的 message。
第十五次测试，更改上次 body 体的错误，错误传输<‘hexo’>，加了单引号，未识别。
成功了，哈哈。。。
2019.1.30
