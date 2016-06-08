
title: svn 使用笔记
date: 2015-12-27 20:41:22
tags:
- 开始
- 我
- 日记
categories: svn
---

### svn目录的作用

  svn的目录分为:  
    1. branchs  
    1. tags  
    1. trunk  
  其中trunk不做开发使用,保持为最新发布或者预发布的版本, 
  tags 作为版本标记,每次发版或者bug修复后标记版本 
  branches 分支作为开发使用,开发新功能或者修复bug ,创建 ***-dev分支 作为新功能开发
<!-- more -->

#### 场景1
	
在__***-dev__分支进行开发,合并到__trunk__主干,测试,正常发布,
测试出现问题,加入__***-dev__分支更改bug , 再次合并到__trunk__主干,测试发布
发布成功之后, 在trunk 上 标记 tags 

#### 场景2

在__***-dev__分支进行开发, 最新版本有bug , 在主干__trunk__创建分支,修改bug,
修改完成 ,合并到主干,  发布成功之后, 在trunk 标记tags, 假如,主干已经合并了新功能,
则在分支,发布, 标记tags.

### svn分支合并

#### svn 分支合并到主干


#### svn 主干合并到分支

![测试图](/uploads/gjh.jpg)



