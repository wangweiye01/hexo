---
title: mac系统更新后git不可用
date: 2017-12-07 11:40:04
tags:
---
更新了系统后，使用git命令，提示错误如下：
```
xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun
```
## 原因

因为每次更新系统之后xcode就被卸载了，因此需要重新安装一次

## 解决方案

终端执行
```
xcode-select --install
```
