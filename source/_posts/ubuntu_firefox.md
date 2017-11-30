---
title: ubuntu更新火狐
date: 2017-11-28 15:49:48
tags:
---
# 删除老版本

```
sudo apt-get remove firefox
```

# 下载最新软件包

```
http://www.firefox.com.cn/download/
```
解压到本地

# 移动文件到/opt文件夹下

```
sudo mv firefox /opt
```

# 创建桌面图标

在/usr/share/applications下创建firefox.desktop文件
```
sudo touch firefox.desktop
```
编辑文件
```
[Desktop Entry]
Name=Firefox
Name[zh_CN]=火狐浏览器
Comment=this is firefox
Exec=/opt/firefox/firefox
Icon=/opt/firefox/icons/fox.png
Terminal=false
Type=Application
Categories=Application;Network;
```
