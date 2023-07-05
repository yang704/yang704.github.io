# Node.js编程基础概要

##### Node.js是什么？

- Node.js不是一种独立的编程语言。
- Node.js不是Javascript框架。
- Node.js是一个javascript运行环境。
> Node.js是一个出浏览器外可以让javascript运行的环境。

##### Node.js与javascript的关系

- Node.js在内置了JavaScript V8 Engine以后实际上只能执行 ECMAScript,就是语言中的语法部分。
- 浏览器为了能够让js操作浏览器窗口以及html文档，所以在JavaScript V8 Enginet添加了能够操作他们的API，就是我们常说的DOM和BOM，所以JavaScript在浏览器中是可以操作浏览器的窗口对象和DOM的文档对象的。
- Node.js中是没有DOM和BOM,所以在node.js中是不能执行和他们相关的代码，比如`window.alert()`或者`document.getElementById()`。DOM与BOM是浏览器环境中特有的。在Node.js中添加了很多系统级别的API，比如对操作系统的文件和文件夹进行操作。获取操作系统信息，比如系统内存总量是多少，系统的临时目录在哪，对系统进程进行操作。

![image](https://cdn.jsdelivr.net/gh/yang704/docsImg/Image/nodejs202307051012306.jpg)

##### Node.js能做什么？

- 后端Web服务器开发与网络爬虫开发。
- 脚手架命令行工具。
- 图形界面应用程序开发。
