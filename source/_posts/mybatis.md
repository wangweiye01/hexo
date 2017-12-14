---
title: mybatis Integer类型，0的问题
date: 2017-12-14 10:17:34
tags:
---

问题：在mybatis中，0被认为是空字符串
解决：普通判断1. != null 2. != ''。当类型为Integer类型时只进行方式1的判断
