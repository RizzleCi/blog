---
title: 不常用的知识整理
date: 2016-07-21 08:47:05
tags: [css,javascript,html]
---
很多整理自以前的博客，算是记录一下自己记不住的东西了~
## user-modify
这个东西要加浏览器前缀才能用：-webkit-user-modify -moz-user-modify
值有  read-only（只读） read-write（可读写） read-write-plaintext-only（可读写文本）。据说老版本的火狐还支持write-only，既然现在都不支持这个了还是忽略它吧。
张鑫旭的[这一篇文章](http://www.zhangxinxu.com/wordpress/2016/01/contenteditable-plaintext-only/)详细介绍了该属性和另一个可以让html可编辑的属性：contenteditable。
<!--more-->
## Sublime Text 3包管理
Ctrl + ~打开命令行 输入
import urllib.request,os; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); open(os.path.join(ipp, pf), 'wb').write(urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ','%20')).read())
重启
shift + ctrl + p 输入 install 可找到安装package，输入remove可以找到卸载package的

### 一些包
ConvertToUTF8 GBK转UTF8编码
babel Snippets react语法高亮

## ubuntu升级内核后VB虚拟机不能启动 
运行sudo /etc/init.d/vboxdrv setup 重新安装

## 兼容hack

| hack      |	写法	  | IE6(S) |IE6(Q) |IE7(S) |IE7(Q)| IE8(S) |IE8(Q) |IE9(S) |IE9(Q) |IE10(S)|IE10(Q)|
| ----------|:-----------:|:------:|:-----:|:-----:|:----:|:------:|:-----:|:-----:|:-----:|:-----:| -----:|
| *         |	*color	  |   Y    |   Y   |   Y   |  Y   |   N    |   Y   |   N   |   Y   |   N   |   Y   |
| +         |	+color	  |   Y    |   Y   |   Y   |  Y   |   N    |   Y   |   N   |   Y   |   N   |   Y   |
| -         |	-color	  |   Y    |   Y   |   N   |  N   |   N    |   N   |   N   |   N   |   N   |   N   |
| _         |	_color	  |   Y    |   Y   |   N   |  Y   |   N    |   Y   |   N   |   Y   |   N   |   N   |
| #         |	#color	  |   Y    |   Y   |   Y   |  Y   |   N    |   Y   |   N   |   Y   |   N   |   Y   |
|\0	        |color:red\0  |	  N    |   N   |   N   |  N   |   Y    |   N   |   Y   |   N   |   Y   |   N   |
|\9\0       |color:red\9\0|	  N    |   N   |   N   |  N   |   N    |   N   |   Y   |   N   |   Y   |   N   |
*html *前缀只对IE6生效
*+html *+前缀只对IE7生效
@media screen\9{...}只对IE6/7生效
@media \0screen {body { background: red; }}只对IE8有效
@media \0screen\,screen\9{body { background: blue; }}只对IE6/7/8有效
@media screen\0 {body { background: green; }} 只对IE8/9/10有效
@media screen and (min-width:0\0) {body { background: gray; }} 只对IE9/10有效
@media screen and (-ms-high-contrast: active), (-ms-high-contrast: none) {body { background: orange; }} 只对IE10有效

## 移动端最小触控区域
44*44pt