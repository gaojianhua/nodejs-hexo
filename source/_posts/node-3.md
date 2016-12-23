title: node学习笔记(3)--hello
date: 2014-03-27 21:10:29
tags:
- 笔记
categories: node
---

node.js 有控制台交互模式和脚本模式

<!-- more -->
### 交互模式
打开控制台,输入node 进入交互模式 
```bash
D:\myWorkTemp\node-study>node
> console.log('hello word!')
hello word!
```

输入.exit 退出交互模式 ,或者使用两次ctrl+c 退出

### 脚本模式

创建文件内容如下:
console.log('Hello World!');

保存该文件，文件名为 helloworld.js， 并通过 node命令来执行：

```bash
>node helloworld.js 
Hello world !
```