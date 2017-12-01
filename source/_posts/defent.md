---
title: 常见网络攻击及处理办法
date: 2017-11-29 11:35:30
tags:
---

# 跨站请求伪造(CSRF)

## 原理

危害是攻击者可以盗用你的身份，以你的名义发送恶意请求。比如可以盗取你的账号，以你的身份发送邮件，购买商品等

## 例子

在某个论坛管理页面，管理员可以在list.php页面执行删除帖子操作，根据URL判断删除帖子的id，像这样的一个URL

```
http://localhost/list.php?action=delete&id=12
```

当恶意用户想管理员发送包含CSFR的邮件，骗取管理员访问 http://test.com/csrf.php 在这个恶意网页中只要包含这样的html语句就可以利用让管理员在不知
情的情况下删除帖子了
```
<img alt="" src="http://localhost/list.php?action=delete&id=12"/>
```

## 使用post修改信息就安全了么？

```
<?php
$action=$_POST['action'];
$id=$_POST['id'];
delete($action,$id);
?>
```
同样可以攻击
```
<!DOCTYPE html>
<html>
　　<body>
　　　　<iframe display="none">
　　　　　　<form method="post" action="http://localhost/list.php">
　　　　　　　　<input type="hidden" name="action" value="delete">
　　　　　　　 <input type="hidden" name="id" value="12">
<input id="csfr" type="submit"/>
　　　　　　</form>
　　　　</iframe>

<script type="text/javascript">
　　　　 document.getElementById('csfr').submit();
　　　　</script>
　　</body>
</html>
```

## 如何防范

1. 使用post，不使用get修改信息
2. 验证码，所有表单的提交需要验证码，但是貌似用起来很麻烦，所以一些关键的操作可以
3. 在表单中预先植入一些加密信息，验证请求是此表单发送

# 跨站脚本攻击(XSS)

## 原理

跨站脚本攻击(Cross Site Script为了区别于CSS简称为XSS)指的是恶意攻击者往Web页面里插入恶意html代码，当用户浏览该页之时，嵌入其中Web里面的htm
l代码会被执行，从而达到恶意用户的特殊目的

## 例子

我们有个页面用于允许用户发表留言，然后在页面底部显示留言列表
```
<!DOCTYPE html>
<html>
<head>
<?php include('/components/headerinclude.php');?></head>
<style type="text/css">
.comment-title{
    font-size:14px;
margin: 6px 0px 2px 4px;
}

.comment-body{
    font-size: 14px;
color:#ccc;
      font-style: italic;
      border-bottom: dashed 1px #ccc;
margin: 4px;
}
</style>
<script type="text/javascript" src="/js/cookies.js"></script>
<body>
<form method="post" action="list.php">
<div style="margin:20px;">
<div style="font-size:16px;font-weight:bold;">Your Comment</div>
<div style="padding:6px;">
Nick Name:
<br/>
<input name="name" type="text" style="width:300px;"/>
</div>
<div style="padding:6px;">
Comment:
<br/>
<textarea name="comment" style="height:100px; width:300px;"></textarea>
</div>
<div style="padding-left:230px;">
<input type="submit" value="POST" style="padding:4px 0px; width:80px;"/>
</div>
<div style="border-bottom:solid 1px #fff;margin-top:10px;">
<div style="font-size:16px;font-weight:bold;">Comments</div>
</div>
<?php 
require('/components/comments.php'); 
if(!empty($_POST['name'])){
    addElement($_POST['name'],$_POST['comment']);
}
renderComments();
?>
</div>
</form>
</body>
</html>
```
addElement()方法用于添加新的留言，而renderComments()方法用于展留言列表，网页看起来是这样的
![](http://img.blog.csdn.net/20171008105555518?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMTY3MTI2NzE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

因为我们完全信任了用户输入，但有些别有用心的用户会像这样的输入
![](http://img.blog.csdn.net/20171008104916455?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luYXRfMTY3MTI2NzE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
这样无论是谁访问这个页面的时候控制台都会输出“Hey you are a fool fish!”，如果这只是个恶意的小玩笑，有些人做的事情就不可爱了，有些用户会利用
这个漏洞窃取用户信息、诱骗人打开恶意网站或者下载恶意程序等

## 如何防范

上面演示的是一个非常简单的XSS攻击，还有很多隐蔽的方式，但是其核心都是利用了脚本注入，因此我们解决办法其实很简单，不信赖用户输入，对特殊字符如”<”,”>”转义，就可以从根本上防止这一问题，当然很多解决方案都对XSS做了特定限制，如上面这中做法在ASP.NET中不幸不同，微软validateRequest对表单提交自动做了XSS验证。但防不胜防，总有些聪明的恶意用户会到我们的网站搞破坏，对自己站点不放心可以看看这个[XSS跨站测试代码大全](http://www.cnblogs.com/dsky/archive/2012/04/06/2434768.html)试试站点是否安全。

