title: git学习笔记
date: 2014-01-28 20:40:18
tags:
- 开始
- 我
- 日记
categories: git
---

由于公司历史的原因,项目代码的版本管理一直在用svn,
	但是由于git的流行,和github的使用广泛, 故记录git的学习笔记,仅供参考

<!-- more -->

初始化一个Git仓库，使用git init命令。

添加文件到Git仓库，分两步：

第一步，使用命令git add <file>，注意，可反复多次使用，添加多个文件；

添加全部文件

```bash
git add .  
```


第二步，使用命令git commit，完成。

```bash
git commit -m "init"
```

仓库当前的状态:

```bash
 git status
 ```


查看修改的内容:

```bash
 git diff readme.txt 
```



要关联一个远程库，使用命令

```bash
git remote add origin git@server-name:path/repo-name.git
```

推送到远程仓库,创建master分支
把本地库的内容推送到远程，用git push命令，实际上是把当前分支master推送到远程。

由于远程库是空的，我们第一次推送master分支时，加上了-u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令。

```bash
git push -u origin master
```

从现在起，只要本地作了提交，就可以通过命令：

```bash
 git push origin master
 ```



参考:
	[git教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001374027586935cf69c53637d8458c9aec27dd546a6cd6000)

