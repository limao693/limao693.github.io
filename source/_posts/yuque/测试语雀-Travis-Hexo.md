
---
title: 测试语雀-Travis-Hexo
date: 2019-01-29 00:16:15 +0800
tags: []
categories: 
---
## 信息来源
本次语雀到Travis的配置，来源于：<br />[https://segmentfault.com/a/1190000017797561#articleHeader1](https://segmentfault.com/a/1190000017797561#articleHeader1)

仅分享搭建历程，及代码分析。
## TEST步骤
第一次，init测试。检测是否同步信息，有效期5分钟。过期删除<br />第二次测试，时间5分钟<br />第三次测试，时间3分钟<br />第四次测试，时间3分钟，仅测试 POSTMan触发请求，查看语雀是否会同步至github。失败<br />第五次测试，测试yuque-hexo组件clean/sync命令的先后顺序。<br />第六次测试，测试yuque-hexo组件，先clean后执行sync命令。<br />第七次测试，测试POSTMan触发请求。<br />第八次测试，测试serverless函数的正确性。<br />第九次测试，测试serverless函数的正确性。<br />第十次测试，测试serverless函数的正确性,参数进行硬编码。<br />第十一次测试，测试serverless函数的正确性.<br />第十二次测试，测试serverless函数的正确性.<br />第十二次测试，测试serverless函数的正确性,测试返回值。<br />第十三次测试，测试Travis pull queest的设置是否阻止的触发。<br />第十四次测试，测试serverless函数的正确性,移除触发body体的message。<br />第十五次测试，更改上次body体的错误，错误传输<‘hexo’>，加了单引号，未识别。<br />成功了，哈哈。。。<br />2019.1.30

